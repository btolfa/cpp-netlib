
# Copyright 2010 Dean Michael Berris.
# Distributed under the Boost Software License, Version 1.0.
# (See accompanying file LICENSE_1_0.txt or copy at
# http://www.boost.org/LICENSE_1_0.txt)

cmake_minimum_required(VERSION 3.11 FATAL_ERROR)
project(cpp-netlib-http-tests)

set(TESTS response_incremental_parser_test request_incremental_parser_test
  request_linearize_test)
foreach ( test ${TESTS} )
  add_executable(cpp-netlib-http-${test} ${test}.cpp)
  target_link_libraries(cpp-netlib-http-${test} PUBLIC
      cppnetlib-BuildSettings cppnetlib-uri gtest_main 
      $<$<PLATFORM_ID:Windows>:wsock32>
      $<$<PLATFORM_ID:Linux>:rt>
  )

  set_target_properties(cpp-netlib-http-${test}
    PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CPP-NETLIB_BINARY_DIR}/tests)
  add_test(NAME cpp-netlib-http-${test} COMMAND cpp-netlib-http-${test})
endforeach()

set(TESTS
  client_constructor_test
  client_get_test
  client_get_different_port_test
  # client_get_timeout_test
  client_get_ready_test
  client_get_streaming_test)
foreach ( test ${TESTS} )
  add_executable(cpp-netlib-http-${test} ${test}.cpp)
  target_link_libraries(cpp-netlib-http-${test} PUBLIC
      cppnetlib-BuildSettings cppnetlib-uri cppnetlib-client-connections gtest_main 
      $<$<PLATFORM_ID:Windows>:wsock32>
      $<$<PLATFORM_ID:Linux>:rt>
  )

  set_target_properties(cpp-netlib-http-${test}
    PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CPP-NETLIB_BINARY_DIR}/tests)
  add_test(NAME cpp-netlib-http-${test} COMMAND cpp-netlib-http-${test})
endforeach()

set(SERVER_API_TESTS server_constructor_test
  server_header_parser_test)
foreach (test ${SERVER_API_TESTS})
  add_executable(cpp-netlib-http-${test} ${test}.cpp)
  target_link_libraries(cpp-netlib-http-${test} PUBLIC
      cppnetlib-BuildSettings cppnetlib-server-parsers gtest_main 
      $<$<PLATFORM_ID:Windows>:wsock32>
      $<$<PLATFORM_ID:Linux>:rt>
  )
  set_target_properties(cpp-netlib-http-${test}
    PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CPP-NETLIB_BINARY_DIR}/tests)
  add_test(NAME cpp-netlib-http-${test} COMMAND cpp-netlib-http-${test})
endforeach()

# Add the server start/stop concurrency test
add_executable(cpp-netlib-http-server_async_run_stop_concurrency server_async_run_stop_concurrency.cpp)
target_link_libraries(cpp-netlib-http-server_async_run_stop_concurrency PUBLIC
  cppnetlib-BuildSettings cppnetlib-server-parsers gtest_main 
  $<$<PLATFORM_ID:Windows>:wsock32>
  $<$<PLATFORM_ID:Linux>:rt>
)
set_target_properties(cpp-netlib-http-server_async_run_stop_concurrency
  PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CPP-NETLIB_BINARY_DIR}/tests)
add_test(NAME cpp-netlib-http-server_async_run_stop_concurrency COMMAND cpp-netlib-http-server_async_run_stop_concurrency)