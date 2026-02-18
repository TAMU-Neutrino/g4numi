# dataProducts/CMakeLists.txt

# Build a shared library from sources in this directory.
file(GLOB dataprod_sources CONFIGURE_DEPENDS
  "${CMAKE_CURRENT_SOURCE_DIR}/*.cc"
  "${CMAKE_CURRENT_SOURCE_DIR}/*.cxx"
)
file(GLOB dataprod_headers CONFIGURE_DEPENDS
  "${CMAKE_CURRENT_SOURCE_DIR}/*.h"
  "${CMAKE_CURRENT_SOURCE_DIR}/*.hh"
  "${CMAKE_CURRENT_SOURCE_DIR}/*.hpp"
)

add_library(dataProducts SHARED
  ${dataprod_sources}
  ${dataprod_headers}
)

target_link_libraries(dataProducts
  PUBLIC  g4numi::deps
  PRIVATE g4numi::build_options g4numi::warnings
)

target_include_directories(dataProducts
  PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

# ---- ROOT dictionary (replaces build_dictionary / ArtDictionary) ----
# If you have a LinkDef here, we generate a dictionary and add it to the library.
set(_linkdef "")
foreach(_cand IN ITEMS LinkDef.h Linkdef.h linkdef.h LinkDef.hh Linkdef.hh)
  if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/${_cand}")
    set(_linkdef "${CMAKE_CURRENT_SOURCE_DIR}/${_cand}")
    break()
  endif()
endforeach()

if(_linkdef)
  # Heuristic: dictionary headers = headers in this directory (minus the LinkDef itself).
  set(_dict_headers ${dataprod_headers})
  list(REMOVE_ITEM _dict_headers "${_linkdef}")

  if(_dict_headers)
    ROOT_GENERATE_DICTIONARY(G__dataProducts
      ${_dict_headers}
      LINKDEF ${_linkdef}
    )
    target_sources(dataProducts PRIVATE
      "${CMAKE_CURRENT_BINARY_DIR}/G__dataProducts.cxx"
    )

    # These files are commonly produced; install if they exist.
    install(FILES
      "${CMAKE_CURRENT_BINARY_DIR}/libdataProducts.rootmap"
      "${CMAKE_CURRENT_BINARY_DIR}/libdataProducts_rdict.pcm"
      DESTINATION ${CMAKE_INSTALL_LIBDIR}
      OPTIONAL
    )
  else()
    message(WARNING "dataProducts: LinkDef found (${_linkdef}) but no headers found for dictionary generation.")
  endif()
else()
  message(STATUS "dataProducts: no LinkDef found; skipping ROOT dictionary generation (verify this is OK for your I/O).")
endif()

# ---- Install ----
install(TARGETS dataProducts
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

# Install headers local to dataProducts into include/dataProducts/
if(dataprod_headers)
  install(FILES ${dataprod_headers}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/dataProducts
  )
endif()

