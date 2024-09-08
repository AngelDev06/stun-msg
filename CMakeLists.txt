cmake_minimum_required(VERSION 3.21)

project(stun-msg)

set(PROJECT_IS_TOP_LEVEL)
if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
  set(PROJECT_IS_TOP_LEVEL ON)
endif()

option(STUNMSG_BUILD_TESTS "Build stunmsg tests" ${PROJECT_IS_TOP_LEVEL})

foreach(SRC "crc32" "hmac_sha1" "sha1" "md5" "stun_msg")
  list(APPEND STUNMSG_IMPL "src/${SRC}.c")
  if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/src/${SRC}.h")
    list(APPEND STUNMSG_IMPL "src/${SRC}.h")
  endif()
endforeach()

add_library(stunmsg STATIC
  ${STUNMSG_IMPL}
  "include/stun/msg.h"
)

target_include_directories(stunmsg PUBLIC include)

if(STUNMSG_BUILD_TESTS)
  add_subdirectory(gtest)

  target_compile_definitions(gtest
  PUBLIC
    _SILENCE_TR1_NAMESPACE_DEPRECATION_WARNING
  )

  foreach(SRC
    "stunmsg_unittest" 
    "stunmsgpp_unittest" 
    "sha1_unittest" 
    "hmac_sha1_unittest" 
    "md5_unittest" 
    "crc32_unittest"
  )
    list(APPEND STUNMSG_TESTS_IMPL "tests/${SRC}.cc")
  endforeach()

  add_executable(stunmsg_tests
    ${STUNMSG_TESTS_IMPL}
    "include/stun++/message.h"
  )

  target_include_directories(stunmsg_tests PRIVATE gtest/include src)
  target_link_libraries(stunmsg_tests
  PRIVATE 
    stunmsg 
    gtest 
    gtest_main
    $<$<BOOL:${WIN32}>:Ws2_32>
  )
endif()
