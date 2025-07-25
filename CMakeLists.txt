cmake_minimum_required(VERSION 3.10)

# uncomment this to display internal cmake state for targets, useful for debugging
# set_property(GLOBAL PROPERTY GLOBAL_DEPENDS_DEBUG_MODE 1)

list(APPEND CMAKE_MODULE_PATH
  ${CMAKE_CURRENT_SOURCE_DIR}/cmake
  ${CMAKE_CURRENT_SOURCE_DIR}/victor-clad/cmake
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/audio/cmake
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/util/cmake
  ${CMAKE_CURRENT_SOURCE_DIR}/tools/build/cmake
)

project(victor)

# Build options
option(USE_DAS "Enable DAS reporting" OFF)

# Use ccache if it is installed
include(ccache)

include(import)
include(clean_clad_targets)
include(license)

# These are used by some android cmake toolchains
# They appear to be ignored by the android-sdk toolchain that works with gradle/Android Studio
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)
set(CMAKE_C_STANDARD 99)

#
# Special settings for xcode.
# Set DEPLOY_WITH_CMAKE OFF to prevent anki_build_copy_assets from generating giant dependency lists.
# Ninja manages these lists efficiently but xcode build takes forever.
#
if (CMAKE_GENERATOR MATCHES "Xcode")
  option(CMAKE_XCODE_GENERATE_SCHEME "Generate xcode schema files" ON)
  option(DEPLOY_WITH_CMAKE "Use cmake instead of python3 for deploying assets" OFF)
endif()

# LTTNG based userspace tracing
option(USE_ANKITRACE "Enable ANKITRACE tracing" OFF)
include(ankitrace)

# Set up custom property for file locations for custom targets
define_property(TARGET PROPERTY ANKI_OUT_PATH
                BRIEF_DOCS "The output path for this custom target"
                FULL_DOCS "The output path for this custom target")

set(SYMBOL_VISIBILITY_FLAGS "")

set(ANKI_PLATFORM_NAME "unknown")
if(MACOSX)
  set(ANKI_PLATFORM_NAME "mac")
elseif(VICOS)
  set(ANKI_PLATFORM_NAME "vicos")
  set(SYMBOL_VISIBILITY_FLAGS "-fvisibility=hidden")
else()

endif()

message(STATUS "ANKI_PLATFORM_NAME: ${ANKI_PLATFORM_NAME}")

add_compile_options(
    $<$<CONFIG:Debug>:-O0>
    $<$<CONFIG:Release>:-O2>
)

# RPATH options
if (VICOS)
  set(CMAKE_INSTALL_RPATH "/anki/lib")
  set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
endif()

#
# Factory build options
#
option(FACTORY_TEST "Enable factory test features" OFF)
option(FACTORY_TEST_DEV "Enable factory test development features" OFF)

if (FACTORY_TEST)
  add_definitions(-DFACTORY_TEST=1)
else()
  add_definitions(-DFACTORY_TEST=0)
endif()

if (FACTORY_TEST_DEV)
  add_definitions(-DFACTORY_TEST_DEV=1)
else()
  add_definitions(-DFACTORY_TEST_DEV=0)
endif()

add_definitions(-DANKI_BUILD_SHA="${ANKI_BUILD_SHA}")
add_definitions(-DANKI_BUILD_BRANCH="${ANKI_BUILD_BRANCH}")

#
# Pass ANKI defines from the command-line to all targets
#
get_cmake_property(opts CACHE_VARIABLES)
foreach(opt ${opts})
  if(opt STREQUAL "ANKI_BUILD_SHA")
  elseif(opt STREQUAL "ANKI_BUILD_BRANCH")
    # ignore, set previously

  # we whitelist which defines are allowed to be added to prevent hidden, unexpected defines from sneaking in
  elseif((opt MATCHES "ANKI_") OR (opt MATCHES "REMOTE_CONSOLE_ENABLED") OR (opt MATCHES "DO_DEV_POSE_CHECKS"))
    get_property(helpstring CACHE ${opt} PROPERTY HELPSTRING)

    if(${helpstring} MATCHES "cached value")
      # value was set from a previous run, but not this run, turn off
      # Note: remove_definitions #defines the value as OFF (which itself is undefined)
      unset(${opt} CACHE)
      add_definitions(-U${opt})
    else()
      # value was set this run, from the command-line
      add_definitions(-D${opt}=${${opt}})
      set_property(CACHE ${opt} PROPERTY HELPSTRING "cached value from a previous command-line")

    endif()
  endif()
endforeach()

set(LINK_FLAGS "${LINK_FLAGS} -Wl,--threads")

if (NOT CMAKE_ARCHIVE_OUTPUT_DIRECTORY)
  set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
endif()

if (NOT CMAKE_LIBRARY_OUTPUT_DIRECTORY)
  set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
endif()

if (NOT CMAKE_RUNTIME_OUTPUT_DIRECTORY)
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
endif()

set(ANKI_EXTERNAL_DIR "${CMAKE_CURRENT_SOURCE_DIR}/EXTERNALS")
set(ANKI_THIRD_PARTY_DIR "${CMAKE_CURRENT_SOURCE_DIR}/3rd")
set(OKAO_VISION_DIR "${CMAKE_CURRENT_SOURCE_DIR}/okaoVision")

set(ANKI_SRCLIST_DIR "${CMAKE_SOURCE_DIR}/generated/cmake")

set(CLAD_BASE_DIR "${CMAKE_SOURCE_DIR}/victor-clad/tools/message-buffers")
set(CLAD_VICTOR_EMITTER_DIR "${CMAKE_SOURCE_DIR}/victor-clad/victorEmitters")
add_subdirectory("victor-clad") # includes clad via message-buffers

# Current DEFAULT (android or mac platforms)
set(CLEAN_CLAD_TARGETS TRUE)

include(audio) # See ./cmake/audio.cmake for definitions and lib
include(wwise) # for Wwise targets
include(protobuf) # for protobuf target
include(breakpad) # for breakpad targets
include(avs-device-sdk)

include(speexdsp)
include(pffft)
include(mpg123)

import(util "lib/util")
import(DAS "lib/das-client")
#include(picovoice)

# BRC-TODO: Make this top-level include into an "interface"
# lib so that other target can depend on it, instead of manually injecting
# include dirs
include_directories("${CMAKE_CURRENT_SOURCE_DIR}/include")

import(coretech "coretech")
add_subdirectory("lib/anki-ble")
add_subdirectory("lib/crash-reporting-vicos")
add_subdirectory("lib/micData")
add_subdirectory("lib/signalEssence")
add_subdirectory("platform")
add_subdirectory("clad")
add_subdirectory("cloud")
add_subdirectory("cubeBleClient")
add_subdirectory("osState")
add_subdirectory("cannedAnimLib")
add_subdirectory("animProcess")
add_subdirectory("webServerProcess")
add_subdirectory("testCrash")
add_subdirectory("dasmgr")

target_include_directories(victor_anim PRIVATE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/anki/cozmo>
)

target_include_directories(victor_web_library PRIVATE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/anki/cozmo>
)

add_subdirectory("robot")

import(cozmo_engine "engine")

target_include_directories(cozmo_engine PRIVATE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/anki/cozmo>
)

if (VICOS)
  include(vicos-sdk-stl-config)
  include(gen-version)
endif()

# god i just REALLY don't wanna fix this
#if (VICOS)
#  add_subdirectory("auto-test")
#endif()

add_subdirectory("resources")

if (MACOSX)
  add_subdirectory("test")
  add_subdirectory("simulator") # webots

  enable_testing()
endif()

# VICOS/STAGING note: Platforms that generate clad should set this variable
# This avoids nuking CLAD output when no targets were built for a platform
if (CLEAN_CLAD_TARGETS)
  clean_clad_targets()
endif()

# Note: if these dependencies are not included in this top-level cmake
#       file then their targets are local and we can't query them
#       for licenses here
#
# Recommend moving for leaf CMakeLists.txt to this file

include(opencv)
include(okao)
include(flatbuffers)
include(sodium)
include(aubio)
include(signalessence)
include(pryon_lite)

write_license_html()
check_licenses()
