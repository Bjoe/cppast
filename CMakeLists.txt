# Copyright (C) 2017-2018 Jonathan Müller <jonathanmueller.dev@gmail.com>
# This file is subject to the license terms in the LICENSE file
# found in the top-level directory of this distribution.

cmake_minimum_required(VERSION 3.8)

option(HUNTER_ENABLED "whether or not to build with hunter packages" OFF)
include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/cpp-pm/hunter/archive/v0.23.240.tar.gz"
    SHA1 "ca19f3769e6c80cfdd19d8b12ba5102c27b074e0"
)

project(cppast VERSION 0.0)

# options
option(CPPAST_BUILD_TEST "whether or not to build the tests" OFF)
option(CPPAST_BUILD_EXAMPLE "whether or not to build the examples" OFF)
option(CPPAST_BUILD_TOOL "whether or not to build the tool" OFF)
option(CPPAST_BUILD_EXTERNAL "whether or not to build the external third party libs" OFF)

if(CPPAST_BUILD_EXTERNAL AND NOT HUNTERENABLED)
    include(external/external.cmake)
endif()
if(HUNTER_ENABLED AND NOT CPPAST_BUILD_EXTERNAL)
    hunter_add_package(type_safe)
    find_package(type_safe CONFIG REQUIRED)

    hunter_add_package(tiny-process-library)
    find_package(tiny-process-library CONFIG REQUIRED)

    hunter_add_package(LLVM)
    find_package(LLVM CONFIG REQUIRED)

    include_directories(${LLVM_INCLUDE_DIRS})
    add_definitions(${LLVM_DEFINITIONS})
    llvm_map_components_to_libnames(llvm_libs clang)
endif()

if(CPPAST_BUILD_TEST AND CPPAST_TEST_GCOV AND (CMAKE_CXX_COMPILER_ID STREQUAL "GNU"))
    list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")
    include(CodeCoverage)
    APPEND_COVERAGE_COMPILER_FLAGS()
endif()

add_subdirectory(src)

if(CPPAST_BUILD_TEST)
    set(CMAKE_EXPORT_COMPILE_COMMANDS ON) # for the self integration test
    enable_testing()
    add_subdirectory(test)
endif()
if(CPPAST_BUILD_EXAMPLE)
    add_subdirectory(example)
endif()
if(CPPAST_BUILD_TOOL)
    add_subdirectory(tool)
endif()
