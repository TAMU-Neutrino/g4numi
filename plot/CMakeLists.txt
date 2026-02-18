# plot/CMakeLists.txt

file(GLOB plot_sources CONFIGURE_DEPENDS
  "${CMAKE_CURRENT_SOURCE_DIR}/*.cxx"
)
file(GLOB plot_headers CONFIGURE_DEPENDS
  "${CMAKE_CURRENT_SOURCE_DIR}/*.h"
  "${CMAKE_CURRENT_SOURCE_DIR}/*.hh"
)

add_library(plot SHARED
  ${plot_sources}
  ${plot_headers}
)

target_link_libraries(plot
  PUBLIC  g4numi::deps
  PRIVATE g4numi::build_options g4numi::warnings
)

target_include_directories(plot
  PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
)

install(TARGETS plot
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

if(plot_headers)
  install(FILES ${plot_headers}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/plot
  )
endif()

# Install ROOT macros (*.C) to share/<project>/macros/root
file(GLOB CONFIGURE_DEPENDS root_macros "${CMAKE_CURRENT_SOURCE_DIR}/*.C")
if(root_macros)
  install(FILES ${root_macros}
    DESTINATION ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/macros/root
  )
endif()

