cmake_minimum_required(VERSION 3.20)

project(g4numi
  VERSION 0.1.0
  LANGUAGES CXX
)

set(CMAKE_CXX_EXTENSIONS OFF)

include(GNUInstallDirs)

# Set CMAKE_INSTALL_DATAROOTDIR to a reasonable default if not already set by the user or CMake presets.
if(NOT CMAKE_INSTALL_DATADIR OR CMAKE_INSTALL_DATADIR STREQUAL "")
  set(CMAKE_INSTALL_DATADIR "${CMAKE_INSTALL_DATAROOTDIR}" CACHE PATH "Data install dir" FORCE)
endif()

# Build type default
if(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING
      "Choose the type of build." FORCE)
endif()

# Options
option(G4NUMI_WITH_GEANT4_UIVIS "Build with Geant4 UI and Vis drivers" ON)
option(G4NUMI_ENABLE_ASSERTS    "Keep asserts enabled even in Release-like builds" ON)
option(G4NUMI_NO_UNDEFINED      "Link with --no-undefined on ELF platforms" ON)
option(G4NUMI_BUILD_UPS_FILES   "Install legacy UPS metadata (not recommended)" OFF)
option(G4NUMI_G4_CXX17_COMPAT "Enable std::binary_function compatibility for Geant4<11 with C++17" ON)

set(DK2NUDATA_DIR "" CACHE PATH "Path to dk2nudata (flux files etc). If set, baked into compile defs.")

# Compile config targets
add_library(g4numi_build_options INTERFACE)
add_library(g4numi_warnings      INTERFACE)

target_include_directories(g4numi_build_options INTERFACE
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

# Replicate ML_NDEBUG behavior for Release-like configs (including legacy OPT/PROF)
target_compile_definitions(g4numi_build_options INTERFACE
  CMAKEBUILD
  "$<$<OR:$<CONFIG:Release>,$<CONFIG:RelWithDebInfo>,$<CONFIG:MinSizeRel>,$<CONFIG:Opt>,$<CONFIG:Prof>>:ML_NDEBUG>"
)


if(DK2NUDATA_DIR)
  # Use a quoted string literal in code
  target_compile_definitions(g4numi_build_options INTERFACE "DK2NUDATA_DIR=\"${DK2NUDATA_DIR}\"")
endif()

# Warnings
target_compile_options(g4numi_warnings INTERFACE
  "$<$<COMPILE_LANG_AND_ID:CXX,AppleClang,Clang,GNU>:-Wall;-Wextra;-Wshadow;-Wnon-virtual-dtor;-Woverloaded-virtual;-Wpedantic;-Wno-pedantic;-Wno-unused-local-typedefs;-Wno-narrowing;-Wno-deprecated-declarations;-Wno-unused-variable;-Wno-missing-braces;-Wno-undefined-var-template>"
)

# Asserts
if(G4NUMI_ENABLE_ASSERTS)
  target_compile_options(g4numi_build_options INTERFACE
    "$<$<AND:$<COMPILE_LANG_AND_ID:CXX,AppleClang,Clang,GNU>,$<OR:$<CONFIG:Release>,$<CONFIG:RelWithDebInfo>,$<CONFIG:MinSizeRel>,$<CONFIG:Opt>,$<CONFIG:Prof>>>:-UNDEBUG>"
    "$<$<AND:$<COMPILE_LANG_AND_ID:CXX,MSVC>,$<OR:$<CONFIG:Release>,$<CONFIG:RelWithDebInfo>,$<CONFIG:MinSizeRel>,$<CONFIG:Opt>,$<CONFIG:Prof>>>:/UNDEBUG>"
  )
endif()

# Linker: NO_UNDEFINED equivalent for ELF
if(G4NUMI_NO_UNDEFINED AND UNIX AND NOT APPLE)
  target_link_options(g4numi_build_options INTERFACE -Wl,--no-undefined)
endif()

# ---- Dependencies ----

# Geant4
if(G4NUMI_WITH_GEANT4_UIVIS)
  find_package(Geant4 10.4 REQUIRED COMPONENTS ui_all vis_all gdml)
else()
  find_package(Geant4 10.4 REQUIRED COMPONENTS gdml)
endif()

target_compile_features(g4numi_build_options INTERFACE cxx_std_17)

# ---- Geant4 + C++17 compatibility ----
if(G4NUMI_G4_CXX17_COMPAT)
  # Geant4 10.4.x uses std::binary_function in some headers; libc++ removes it in C++17.
  if(DEFINED Geant4_VERSION AND Geant4_VERSION VERSION_LESS "11.0")
    if(CMAKE_CXX_COMPILER_ID MATCHES "AppleClang|Clang")
      target_compile_definitions(g4numi_build_options INTERFACE
        _LIBCPP_ENABLE_CXX17_REMOVED_UNARY_BINARY_FUNCTION
        _LIBCPP_ENABLE_CXX17_REMOVED_BINDERS
      )
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
      target_compile_definitions(g4numi_build_options INTERFACE
        _GLIBCXX_USE_DEPRECATED=1
      )
    endif()
  endif()
endif()

# XercesC (needed for GDML in many Geant4 builds)
find_package(XercesC REQUIRED)

# ROOT6+
set(_root_components
  Core RIO Net Imt Hist Graf Graf3d Gpad Tree Rint Postscript Matrix Physics MathCore Thread EG
  Cling
)
find_package(ROOT CONFIG REQUIRED COMPONENTS ${_root_components})

# dk2nu
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
find_package(DK2NU REQUIRED)

# ---- Consolidate deps into a single INTERFACE target ----
add_library(g4numi_deps INTERFACE)

# Geant4 package is inconsistent across versions re: imported targets.
# Use Geant4_LIBRARIES variable (reliable) + include dirs.
target_include_directories(g4numi_deps INTERFACE ${Geant4_INCLUDE_DIRS})
target_link_libraries(g4numi_deps INTERFACE ${Geant4_LIBRARIES})

# XercesC
if(TARGET XercesC::XercesC)
  target_link_libraries(g4numi_deps INTERFACE XercesC::XercesC)
else()
  target_include_directories(g4numi_deps INTERFACE ${XercesC_INCLUDE_DIRS})
  target_link_libraries(g4numi_deps INTERFACE ${XercesC_LIBRARIES})
endif()

# ROOT imported targets
set(_root_targets "")
foreach(c IN LISTS _root_components)
  if(TARGET ROOT::${c})
    list(APPEND _root_targets ROOT::${c})
  endif()
endforeach()
if(NOT _root_targets)
  message(FATAL_ERROR "ROOT was found but imported targets (ROOT::Core etc.) are missing. Are you using ROOT CONFIG packages?")
endif()
target_link_libraries(g4numi_deps INTERFACE ${_root_targets})

# Convenience alias namespace
add_library(g4numi::build_options ALIAS g4numi_build_options)
add_library(g4numi::warnings      ALIAS g4numi_warnings)
add_library(g4numi::deps          ALIAS g4numi_deps)

# ---- Project layout ----
add_subdirectory(dataProducts)
add_subdirectory(exec)
add_subdirectory(macros)
add_subdirectory(plot)
add_subdirectory(src)
add_subdirectory(studies)

if(G4NUMI_BUILD_UPS_FILES)
  add_subdirectory(ups)
endif()

# Install headers directly
if(EXISTS "${PROJECT_SOURCE_DIR}/include")
  install(DIRECTORY "${PROJECT_SOURCE_DIR}/include/"
          DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")
endif()

# Install top-level ROOT macros
if(root_macros)
  file(GLOB root_macros CONFIGURE_DEPENDS "${PROJECT_SOURCE_DIR}/*.C")
  install(FILES ${root_macros}
          DESTINATION "${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/macros/root")
endif()

# Optional packaging
include(CPack)
