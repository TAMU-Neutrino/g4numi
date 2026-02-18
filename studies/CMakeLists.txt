# studies/CMakeLists.txt

file(GLOB studies_sources CONFIGURE_DEPENDS
  "${CMAKE_CURRENT_SOURCE_DIR}/*.cc"
)
file(GLOB studies_headers CONFIGURE_DEPENDS
  "${CMAKE_CURRENT_SOURCE_DIR}/*.hh"
)

add_library(studies SHARED
  ${studies_sources}
  ${studies_headers}
)

target_link_libraries(studies
  PUBLIC  g4numi::deps
  PRIVATE g4numi::build_options g4numi::warnings
)

target_include_directories(studies
  PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

install(TARGETS studies
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

if(studies_headers)
  install(FILES ${studies_headers}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/studies
  )
endif()

