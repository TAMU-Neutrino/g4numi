# macros/CMakeLists.txt

# Install Geant4 macro files to share/<project>/macros
file(GLOB CONFIGURE_DEPENDS mac_files "${CMAKE_CURRENT_SOURCE_DIR}/*.mac")
if(mac_files)
  install(FILES ${mac_files}
    DESTINATION ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/macros
  )
endif()

# Install common scripts (if you actually have them)
file(GLOB CONFIGURE_DEPENDS scripts
  "${CMAKE_CURRENT_SOURCE_DIR}/*.sh"
  "${CMAKE_CURRENT_SOURCE_DIR}/*.py"
  "${CMAKE_CURRENT_SOURCE_DIR}/*.pl"
)
if(scripts)
  install(PROGRAMS ${scripts}
    DESTINATION ${CMAKE_INSTALL_BINDIR}
  )
endif()

