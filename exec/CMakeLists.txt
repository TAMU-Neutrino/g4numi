# exec/CMakeLists.txt

add_executable(g4numi g4numi.cc)
target_link_libraries(g4numi
  PRIVATE
    src
    g4numi::deps
    g4numi::build_options
    g4numi::warnings
)

add_executable(g4na49 g4na49.cc)
target_link_libraries(g4na49
  PRIVATE
    src
    g4numi::deps
    g4numi::build_options
    g4numi::warnings
)

install(TARGETS g4numi g4na49
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

