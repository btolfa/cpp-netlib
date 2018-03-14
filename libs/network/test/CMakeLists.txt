# Copyright (c) Dean Michael Berris 2010.
# Copyright 2015 Google, Inc.
# Distributed under the Boost Software License, Version 1.0.
# (See accompanying file LICENSE_1_0.txt or copy at
# http://www.boost.org/LICENSE_1_0.txt)

cmake_minimum_required(VERSION 3.11 FATAL_ERROR)
project(cpp-netlib-tests CXX)

add_subdirectory(uri)
add_subdirectory(http)

set(TESTS message_test message_transform_test utils_thread_pool
# utils_base64_test -- turn on when ready.
)

foreach (test ${TESTS})
    add_executable(cpp-netlib-${test} ${test}.cpp)
    target_link_libraries(cpp-netlib-${test} PUBLIC
      cppnetlib-BuildSettings cppnetlib-uri gtest_main 
      $<$<PLATFORM_ID:Windows>:wsock32>
      $<$<PLATFORM_ID:Linux>:rt>
      )

    set_target_properties(cpp-netlib-${test}
      PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CPP-NETLIB_BINARY_DIR}/tests)
    add_test(NAME cpp-netlib-${test} COMMAND cpp-netlib-${test})
endforeach()

file(COPY server DESTINATION ${CPP-NETLIB_BINARY_DIR}/libs/network/test/)