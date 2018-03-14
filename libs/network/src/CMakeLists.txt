# Copyright (c) Glyn Matthews 2011.
# Copyright 2011 Dean Michael Berris (dberris@google.com)
# Copyright 2011 Google, Inc.
# Distributed under the Boost Software License, Version 1.0.
#    (See accompanying file LICENSE_1_0.txt or copy at
#          http://www.boost.org/LICENSE_1_0.txt)

cmake_minimum_required(VERSION 3.11 FATAL_ERROR)
project(cppnetlib-src CXX)

add_library(cppnetlib-uri uri/uri.cpp uri/schemes.cpp)
target_compile_definitions(cppnetlib-uri PRIVATE BOOST_SPIRIT_THREADSAFE)
target_link_libraries(cppnetlib-uri PUBLIC cppnetlib-headeronly)
set_target_properties(cppnetlib-uri
    PROPERTIES VERSION ${CPPNETLIB_VERSION_STRING}
    SOVERSION ${CPPNETLIB_VERSION_MAJOR})

add_library(cppnetlib-server-parsers server_request_parsers_impl.cpp)
target_link_libraries(cppnetlib-server-parsers PUBLIC cppnetlib-headeronly)
if (TARGET OpenSSL::SSL)
    target_link_libraries(cppnetlib-server-parsers PUBLIC OpenSSL::SSL)
    if (CPP-NETLIB_STATIC_OPENSSL)
        target_link_libraries(cppnetlib-server-parsers PUBLIC ${CMAKE_DL_LIBS})
    endif()
endif ()
set_target_properties(cppnetlib-server-parsers
    PROPERTIES VERSION ${CPPNETLIB_VERSION_STRING}
    SOVERSION ${CPPNETLIB_VERSION_MAJOR})

add_library(cppnetlib-client-connections client.cpp)
target_link_libraries(cppnetlib-client-connections PUBLIC cppnetlib-headeronly Threads::Threads Boost::regex)
if (TARGET OpenSSL::SSL)
    target_link_libraries(cppnetlib-client-connections PUBLIC OpenSSL::SSL)
    if (CPP-NETLIB_STATIC_OPENSSL)
        target_link_libraries(cppnetlib-client-connections PUBLIC ${CMAKE_DL_LIBS})
    endif()
endif ()
if (MINGW)
    target_link_libraries(cppnetlib-client-connections PUBLIC ws2_32)
endif ()
set_target_properties(cppnetlib-client-connections
    PROPERTIES VERSION ${CPPNETLIB_VERSION_STRING}
    SOVERSION ${CPPNETLIB_VERSION_MAJOR})