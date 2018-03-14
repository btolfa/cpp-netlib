# Copyright (c) Dean Michael Berris 2010.
# Copyright 2016 Google, Inc.
# Distributed under the Boost Software License, Version 1.0.
# (See accompanying file LICENSE_1_0.txt or copy at
# http://www.boost.org/LICENSE_1_0.txt)

cmake_minimum_required(VERSION 3.11 FATAL_ERROR)
project(CPP-NETLIB CXX)

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
  set(CPP-NETLIB_STANDALONE ON)
else()
  set(CPP-NETLIB_STANDALONE OFF)
endif()

option( CPP-NETLIB_BUILD_SHARED_LIBS "Build cpp-netlib as shared libraries." OFF )
option( CPP-NETLIB_BUILD_TESTS "Build the cpp-netlib project tests." ${CPP-NETLIB_STANDALONE})
# option( CPP-NETLIB_BUILD_EXPERIMENTS "Build the cpp-netlib project experiments." ON)
option( CPP-NETLIB_BUILD_EXAMPLES "Build the cpp-netlib project examples." ${CPP-NETLIB_STANDALONE})
option( CPP-NETLIB_ENABLE_HTTPS "Build cpp-netlib with support for https if OpenSSL is found." ON)
option( CPP-NETLIB_STATIC_OPENSSL "Build cpp-netlib using static OpenSSL" OFF)
option( CPP-NETLIB_STATIC_BOOST "Build cpp-netlib using static Boost" OFF)

# This is the interface library for global settings, flags, options and definitions,
# which have not be propagated to dependent targets. It's a good place for warning level flags.
add_library(cppnetlib-BuildSettings INTERFACE)

# This is the interface library for global settings, flags, option and definitions which
# have to be propagated to dependent targets.
add_library(cppnetlib-headeronly INTERFACE)

if(CPP-NETLIB_BUILD_SHARED_LIBS OR BUILD_SHARED_LIBS)
  message (STATUS "Linking boost testing libs dynamically...")
  set(CPP-NETLIB_BUILD_SHARED_LIBS ON)
  set(BUILD_SHARED_LIBS ON)
else()
  set(CPP-NETLIB_BUILD_SHARED_LIBS OFF)
  set(BUILD_SHARED_LIBS OFF)
endif()

# Use Boost's static libraries
if (CPP-NETLIB_STATIC_BOOST)
  set(Boost_USE_STATIC_LIBS ON)
endif()

# We need this for all tests to use the dynamic version.
target_compile_definitions(cppnetlib-BuildSettings INTERFACE BOOST_TEST_DYN_LINK)

# Always use multi-threaded Boost libraries.
set(Boost_USE_MULTI_THREADED ON)

find_package(Boost 1.58.0 REQUIRED COMPONENTS system thread OPTIONAL_COMPONENTS regex program_options)

if (CPP-NETLIB_ENABLE_HTTPS)
 if (APPLE)
   # If we're on OS X check for Homebrew's copy of OpenSSL instead of Apple's
   if (NOT OpenSSL_DIR)
     find_program(HOMEBREW brew)
     if (HOMEBREW STREQUAL "HOMEBREW-NOTFOUND")
       message(WARNING "Homebrew not found: not using Homebrew's OpenSSL")
       if (NOT OPENSSL_ROOT_DIR)
         message(WARNING "Use -DOPENSSL_ROOT_DIR for non-Apple OpenSSL")
       endif()
     else()
       execute_process(COMMAND brew --prefix openssl
         OUTPUT_VARIABLE OPENSSL_ROOT_DIR
         OUTPUT_STRIP_TRAILING_WHITESPACE)
     endif()
   endif()
 endif()
 if (CPP-NETLIB_STATIC_OPENSSL)
    set(OPENSSL_USE_STATIC_LIBS TRUE)
 endif()

 find_package(OpenSSL REQUIRED)
 target_compile_definitions(cppnetlib-headeronly INTERFACE BOOST_NETWORK_ENABLE_HTTPS)
 target_link_libraries(cppnetlib-headeronly INTERFACE OpenSSL::SSL)
endif()

find_package(Threads)

set(CPPNETLIB_VERSION_MAJOR 0) # MUST bump this whenever we make ABI-incompatible changes
set(CPPNETLIB_VERSION_MINOR 12)
set(CPPNETLIB_VERSION_PATCH 0)
set(CPPNETLIB_VERSION_STRING ${CPPNETLIB_VERSION_MAJOR}.${CPPNETLIB_VERSION_MINOR}.${CPPNETLIB_VERSION_PATCH})

target_compile_definitions(cppnetlib-BuildSettings INTERFACE $<$<CONFIG:Debug>:BOOST_NETWORK_DEBUG>)

target_compile_features(cppnetlib-BuildSettings INTERFACE cxx_std_14)
target_compile_options(cppnetlib-BuildSettings INTERFACE $<$<AND:$<CONFIG:DEBUG>,$<CXX_COMPILER_ID:GNU>>:-Wall>)
target_compile_options(cppnetlib-BuildSettings INTERFACE $<$<AND:$<CONFIG:DEBUG>,$<CXX_COMPILER_ID:Clang>>:-Wall>)

target_compile_options(cppnetlib-headeronly INTERFACE $<$<AND:$<CONFIG:DEBUG>,$<CXX_COMPILER_ID:Clang>>:-ftemplate-depth=256>)
target_compile_options(cppnetlib-headeronly INTERFACE $<$<PLATFORM_ID:Darwin>:-stdlib=libc++>)
if (CMAKE_SYSTEM_NAME MATCHES "Darwin")
  set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -lc++")
endif()

target_compile_options(cppnetlib-BuildSettings INTERFACE $<$<CXX_COMPILER_ID:MSVC>:/bigobj>)

target_compile_definitions(cppnetlib-headeronly INTERFACE $<$<CXX_COMPILER_ID:MSVC>:_SCL_SECURE_NO_WARNINGS>)
target_compile_definitions(cppnetlib-headeronly INTERFACE $<$<PLATFORM_ID:Windows>:BOOST_USE_WINAPI_VERSION=0x0601 _WIN32_WINNT=0x0601>)

target_include_directories(cppnetlib-headeronly INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
target_link_libraries(cppnetlib-headeronly INTERFACE 
  Boost::system
  Boost::thread)

add_subdirectory(libs/network/src)
if (CPP-NETLIB_BUILD_TESTS)
  enable_testing()

  if (MSVC)
    set(gtest_force_shared_crt ON CACHE BOOL "Use shared (DLL) run-time lib even when Google Test is built as static lib.")
  endif()
  add_subdirectory(deps/googletest)
  target_compile_definitions(gtest PUBLIC $<$<CXX_COMPILER_ID:MSVC>:_SILENCE_TR1_NAMESPACE_DEPRECATION_WARNING>)
  target_compile_definitions(gtest_main PUBLIC $<$<CXX_COMPILER_ID:MSVC>:_SILENCE_TR1_NAMESPACE_DEPRECATION_WARNING>)
  target_compile_definitions(gmock PUBLIC $<$<CXX_COMPILER_ID:MSVC>:_SILENCE_TR1_NAMESPACE_DEPRECATION_WARNING>)
  target_compile_definitions(gmock_main PUBLIC $<$<CXX_COMPILER_ID:MSVC>:_SILENCE_TR1_NAMESPACE_DEPRECATION_WARNING>)

  add_subdirectory(libs/network/test)
endif ()

add_library(cppnetlib::cppnetlib ALIAS cppnetlib-headeronly)
add_library(cppnetlib::client-connections ALIAS cppnetlib-client-connections)
add_library(cppnetlib::server-parsers ALIAS cppnetlib-server-parsers)
add_library(cppnetlib::uri ALIAS cppnetlib-uri)

if (CPP-NETLIB_BUILD_EXAMPLES)

  add_library(cxxopts INTERFACE)
  target_include_directories(cxxopts INTERFACE deps/cxxopts/src)

  add_subdirectory(libs/network/example)
endif ()
