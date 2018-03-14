#            Copyright (c) Dean Michael Berris 2010.
# Distributed under the Boost Software License, Version 1.0.
#    (See accompanying file LICENSE_1_0.txt or copy at
#          http://www.boost.org/LICENSE_1_0.txt)

cmake_minimum_required(VERSION 3.11 FATAL_ERROR)
project(cpp-netlib-examples CXX)

add_executable(http_client http_client.cpp)
add_executable(simple_wget simple_wget.cpp)
add_executable(atom_reader atom/atom.cpp atom/main.cpp)
add_executable(rss_reader rss/rss.cpp rss/main.cpp)
add_executable(hello_world_server http/hello_world_server.cpp)
add_executable(hello_world_client http/hello_world_client.cpp)
add_executable(hello_world_async_server_with_work_queue http/hello_world_async_server_with_work_queue.cpp)
add_executable(trivial_google trivial_google.cpp)

if (UNIX)
  add_executable(fileserver http/fileserver.cpp)
  target_link_libraries(fileserver PRIVATE cppnetlib::server-parsers)
  set_target_properties(fileserver PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CPP-NETLIB_BINARY_DIR}/example)

  add_executable(async_server_file_upload http/async_server_file_upload.cpp)
  target_link_libraries(async_server_file_upload PRIVATE cppnetlib::server-parsers)
  set_target_properties(async_server_file_upload PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CPP-NETLIB_BINARY_DIR}/example)
endif (UNIX)

target_link_libraries(http_client PRIVATE cppnetlib::uri cppnetlib::client-connections cxxopts Boost::program_options)
target_link_libraries(simple_wget PRIVATE cppnetlib::uri cppnetlib::client-connections)
target_link_libraries(atom_reader PRIVATE cppnetlib::uri cppnetlib::client-connections)
target_link_libraries(rss_reader PRIVATE cppnetlib::uri cppnetlib::client-connections)
target_link_libraries(trivial_google PRIVATE cppnetlib::uri cppnetlib::client-connections)

target_link_libraries(hello_world_server PRIVATE cppnetlib::server-parsers)
target_link_libraries(hello_world_client PRIVATE cppnetlib::uri cppnetlib::client-connections)
target_link_libraries(hello_world_async_server_with_work_queue PRIVATE cppnetlib::uri cppnetlib::client-connections cppnetlib::server-parsers)

if (TARGET OpenSSL::SSL)
  add_executable(ssl_server http/ssl/ssl_server.cpp)
  target_link_libraries(ssl_server PRIVATE cppnetlib::uri cppnetlib::client-connections cppnetlib::server-parsers)
  set_target_properties(ssl_server PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CPP-NETLIB_BINARY_DIR}/example)
endif()


set_target_properties(http_client PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CPP-NETLIB_BINARY_DIR}/example)
set_target_properties(simple_wget PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CPP-NETLIB_BINARY_DIR}/example)
set_target_properties(atom_reader PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CPP-NETLIB_BINARY_DIR}/example)
set_target_properties(rss_reader PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CPP-NETLIB_BINARY_DIR}/example)
set_target_properties(trivial_google PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CPP-NETLIB_BINARY_DIR}/example)
set_target_properties(hello_world_server PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CPP-NETLIB_BINARY_DIR}/example)
set_target_properties(hello_world_client PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CPP-NETLIB_BINARY_DIR}/example)
set_target_properties(hello_world_async_server_with_work_queue PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CPP-NETLIB_BINARY_DIR}/example)