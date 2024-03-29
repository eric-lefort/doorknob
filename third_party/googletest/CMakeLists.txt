# Defines pre_project_set_up_hermetic_build() and set_up_hermetic_build().
include(cmake/hermetic_build.cmake OPTIONAL)

if (COMMAND pre_project_set_up_hermetic_build)
  pre_project_set_up_hermetic_build()
endif()

########################################################################
#
# Project-wide settings

# Name of the project.
#
# CMake files in this project can refer to the root source directory
# as ${gtest_SOURCE_DIR} and to the root binary directory as
# ${gtest_BINARY_DIR}.
# Language "C" is required for find_package(Threads).

# Project version:

if (CMAKE_VERSION VERSION_LESS 3.0)
  project(gtest CXX C)
  set(PROJECT_VERSION ${GOOGLETEST_VERSION})
else()
  cmake_policy(SET CMP0048 NEW)
  project(gtest VERSION ${GOOGLETEST_VERSION} LANGUAGES CXX C)
endif()
cmake_minimum_required(VERSION 2.6.4)

if (POLICY CMP0063) # Visibility
  cmake_policy(SET CMP0063 NEW)
endif (POLICY CMP0063)

if (COMMAND set_up_hermetic_build)
  set_up_hermetic_build()
endif()

# These commands only run if this is the main project
if(CMAKE_PROJECT_NAME STREQUAL "gtest" OR CMAKE_PROJECT_NAME STREQUAL "googletest-distribution")

  # BUILD_SHARED_LIBS is a standard CMake variable, but we declare it here to
  # make it prominent in the GUI.
  option(BUILD_SHARED_LIBS "Build shared libraries (DLLs)." OFF)

else()

  mark_as_advanced(
    gtest_force_shared_crt
    gtest_build_tests
    gtest_build_samples
    gtest_disable_pthreads
    gtest_hide_internal_symbols)

endif()


if (gtest_hide_internal_symbols)
  set(CMAKE_CXX_VISIBILITY_PRESET hidden)
  set(CMAKE_VISIBILITY_INLINES_HIDDEN 1)
endif()

# Define helper functions and macros used by Google Test.
include(cmake/internal_utils.cmake)

config_compiler_and_linker()  # Defined in internal_utils.cmake.

# Where Google Test's .h files can be found.
set(gtest_build_include_dirs
  "${gtest_SOURCE_DIR}/include"
  "${gtest_SOURCE_DIR}")
include_directories(${gtest_build_include_dirs})

########################################################################
#
# Defines the gtest & gtest_main libraries.  User tests should link
# with one of them.

# Google Test libraries.  We build them using more strict warnings than what
# are used for other targets, to ensure that gtest can be compiled by a user
# aggressive about warnings.
cxx_library(gtest "${cxx_strict}" src/gtest-all.cc)
cxx_library(gtest_main "${cxx_strict}" src/gtest_main.cc)
# If the CMake version supports it, attach header directory information
# to the targets for when we are part of a parent build (ie being pulled
# in via add_subdirectory() rather than being a standalone build).
if (DEFINED CMAKE_VERSION AND NOT "${CMAKE_VERSION}" VERSION_LESS "2.8.11")
  target_include_directories(gtest SYSTEM INTERFACE
    "$<BUILD_INTERFACE:${gtest_build_include_dirs}>"
    "$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}>")
  target_include_directories(gtest_main SYSTEM INTERFACE
    "$<BUILD_INTERFACE:${gtest_build_include_dirs}>"
    "$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}>")
endif()
target_link_libraries(gtest_main PUBLIC gtest)
