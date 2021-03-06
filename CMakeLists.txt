project(Artery)
cmake_minimum_required(VERSION 3.0)
set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)
set(CMAKE_CXX_STANDARD 11)

find_package(OmnetPP 5.0 REQUIRED)
include(cmake/ImportOppTarget.cmake)
include(cmake/GetNedFolders.cmake)
include(cmake/AddOppRun.cmake)

find_package(Vanetza PATHS extern/vanetza/build REQUIRED)
option(Vanetza_STATIC "Link with static Vanetza libraries" OFF)

find_path(Veins_DIR NAMES src/veins/package.ned PATHS extern/veins DOC "Veins root directory")
import_opp_target(veins ${Veins_DIR}/src/Makefile)

find_program(MAKE_COMMAND NAMES make gmake HINTS ${CMAKE_MAKE_COMMAND} DOC "Makefile processor")
set(MAKE_ARGUMENTS "" CACHE STRING "Additional arguments passed to Makefile processor")
string(REPLACE " " ";" MAKE_ARGUMENTS "${MAKE_ARGUMENTS}")

if("${Veins_DIR}" STREQUAL "${PROJECT_SOURCE_DIR}/extern/veins")
    set(_veins_mode "release")
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        set(_veins_mode "debug")
    endif()

    set(_make_command ${MAKE_COMMAND})
    if(CMAKE_GENERATOR MATCHES "Makefiles")
        set(_make_command \$\(MAKE\))
    endif()

    add_custom_target(build_veins
        COMMAND ${_make_command} ${MAKE_ARGUMENTS} veins MODE=${_veins_mode}
        WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
        COMMENT "Building Veins (external dependency)"
        VERBATIM)
    add_dependencies(veins build_veins)
endif()

add_subdirectory(src/artery)
option(WITH_STORYBOARD "Build Artery with storyboard feature" ON)
if(WITH_STORYBOARD)
    add_subdirectory(src/artery/storyboard)
endif()
add_subdirectory(scenarios)

# sumo-launchd should be run from root directory for correct file look-up
set(SUMO_LAUNCHD_ARGUMENTS "--daemon --kill" CACHE STRING "Additional arguments passed to sumo-launchd")
string(REPLACE " " ";" _sumo_launchd_args "${SUMO_LAUNCHD_ARGUMENTS}")
add_custom_target(launch_sumo
    COMMAND ${Veins_DIR}/sumo-launchd.py ${_sumo_launchd_args}
    WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
    COMMENT "Launching SUMO"
    VERBATIM)
