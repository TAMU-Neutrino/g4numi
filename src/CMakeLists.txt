# src/CMakeLists.txt

file(GLOB src_sources CONFIGURE_DEPENDS
  "${CMAKE_CURRENT_SOURCE_DIR}/*.cc"
)

add_library(src SHARED
  ${src_sources}
)

target_link_libraries(src
  PUBLIC  dataProducts
  PUBLIC  g4numi::deps
  PRIVATE g4numi::build_options g4numi::warnings
)

target_include_directories(src
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

target_link_libraries(src PUBLIC DK2NU::dk2nuTree)

# ---- Install ----
install(TARGETS src
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

