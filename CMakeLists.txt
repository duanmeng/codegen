#
# Copyright © 2019 Paweł Dziepak
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

cmake_minimum_required(VERSION 3.12)

project(codegen
  LANGUAGES CXX
)

include(CTest)

option(CODEGEN_SANITIZERS "Build with AddressSanitizer and UndefinedBehaviorSanitizer." ON)

list(APPEND CODEGEN_CXX_FLAGS -Wall -Wextra -Werror -Wno-unused-parameter)
if (CODEGEN_SANITIZERS)
  list(APPEND CODEGEN_CXX_DEBUG_FLAGS -fsanitize=address -fsanitize=undefined)
endif()

if(CMAKE_BUILD_TYPE STREQUAL Debug)
  list(APPEND CODEGEN_CXX_FLAGS ${CODEGEN_CXX_DEBUG_FLAGS})
else()
  list(APPEND CODEGEN_CXX_FLAGS ${CODEGEN_CXX_RELEASE_FLAGS})
endif()

MESSAGE(STATUS "SystemRoot ${CMAKE_OSX_SYSROOT}")
MESSAGE(STATUS "SystemRoot ${CMAKE_TOOLCHAIN_FILE}")

if (APPLE)
  MESSAGE(STATUS "IS OS X")
  MESSAGE(STATUS "host system is ${CMAKE_HOST_SYSTEM_NAME}")
  MESSAGE(STATUS "host system is ${OS_DARWIN}")
  MESSAGE(STATUS "host system is ${OS_LINUX}")
  set(CMAKE_C_COMPILER /usr/local/opt/llvm/bin/clang)
  set(CMAKE_CXX_COMPILER /usr/local/opt/llvm/bin/clang++)
  #set(CMAKE_C_COMPILER /usr/local/Cellar/llvm@8/8.0.1_4/bin/clang)
  #set(CMAKE_CXX_COMPILER /usr/local/Cellar/llvm@8/8.0.1_4/bin/clang++)
  set(CMAKE_CXX_STANDARD 17)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)
else ()
  MESSAGE(STATUS "IS Not OS X")
  set(CMAKE_C_COMPILER gcc)
  set(CMAKE_CXX_COMPILER g++)
endif ()

# we do not need filesystem anly since the filesystem in part of standards in new version sdk
#set(CODEGEN_CXX_FILESYSTEM stdc++fs)
#if(APPLE) or use OS_DARWIN
#set(CODEGEN_CXX_FILESYSTEM c++fs)
#endif()

# llvm@8 is not int the default llvm cmkae Find Modules (FindLLVM@8.cmake) add LLVM_DIR
set(LLVM_DIR /usr/local/opt/llvm@8/lib/cmake/llvm/)
find_package(LLVM 8 REQUIRED)
#set(LLVM_DIR /usr/local/opt/llvm/lib/cmake/llvm/)
#find_package(LLVM 12 REQUIRED)
find_package(fmt REQUIRED)

add_library(codegen
  src/compiler.cpp
  src/module_builder.cpp
  src/statements.cpp
)
target_include_directories(codegen PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/>
  ${LLVM_INCLUDE_DIR}
)
target_compile_features(codegen PUBLIC cxx_std_17)
target_link_libraries(codegen PRIVATE ${CODEGEN_CXX_FLAGS})
target_compile_options(codegen PRIVATE ${CODEGEN_CXX_FLAGS})
# no need to link filesystem anly more
# target_link_libraries(codegen PUBLIC LLVM fmt::fmt ${CODEGEN_CXX_FILESYSTEM})
target_link_libraries(codegen PUBLIC LLVM fmt::fmt)

if(BUILD_TESTING)
  add_subdirectory(tests)
endif()
