#MIT License

#Copyright (c) 2019 Cristian Adam

#Permission is hereby granted, free of charge, to any person obtaining a copy
#of this software and associated documentation files (the "Software"), to deal
#in the Software without restriction, including without limitation the rights
#to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
#copies of the Software, and to permit persons to whom the Software is
#furnished to do so, subject to the following conditions:

#The above copyright notice and this permission notice shall be included in all
#copies or substantial portions of the Software.

#THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
#AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
#LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
#OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
#SOFTWARE.

cmake_minimum_required(VERSION 3.5)

#
# Download the latest Hunter gate and release
#

file(
  DOWNLOAD https://raw.githubusercontent.com/cpp-pm/gate/master/cmake/HunterGate.cmake
  ${CMAKE_BINARY_DIR}/HunterGate.cmake)
file(
  DOWNLOAD https://api.github.com/repos/cpp-pm/hunter/releases/latest
  ${CMAKE_BINARY_DIR}/LatestGate.json)

file(READ ${CMAKE_BINARY_DIR}/LatestGate.json latestGateJson)

string(REGEX MATCH "URL \\\\\"(.*)\\\\\".*SHA1 \\\\\"(.*)\\\\\".*"
  latestGateCMake "${latestGateJson}")

# Write the Hunter Configuration file
file(WRITE ${CMAKE_BINARY_DIR}/HunterConfig.cmake.in "")
foreach(package IN LISTS HUNTER_PACKAGES)
  file(APPEND ${CMAKE_BINARY_DIR}/HunterConfig.cmake.in
    "hunter_config(${package}
       VERSION ")
  if (HUNTER_${package}_VERSION)
    file(APPEND ${CMAKE_BINARY_DIR}/HunterConfig.cmake.in
      "${HUNTER_${package}_VERSION}")
  else()
    file(APPEND ${CMAKE_BINARY_DIR}/HunterConfig.cmake.in
      "\${HUNTER_${package}_VERSION}")
  endif()

  if (HUNTER_${package}_CMAKE_ARGS)
    file(APPEND ${CMAKE_BINARY_DIR}/HunterConfig.cmake.in "
      CMAKE_ARGS \${HUNTER_${package}_CMAKE_ARGS}")
  endif()

  file(APPEND ${CMAKE_BINARY_DIR}/HunterConfig.cmake.in ")\n\n")
endforeach()
configure_file(
  ${CMAKE_BINARY_DIR}/HunterConfig.cmake.in
  ${CMAKE_BINARY_DIR}/HunterConfig.cmake @ONLY)

if (NOT HUNTER_URL)
  set(HUNTER_URL ${CMAKE_MATCH_1})
endif()

if (NOT HUNTER_SHA1)
  set(HUNTER_SHA1 ${CMAKE_MATCH_2})
endif()

file(WRITE
  ${CMAKE_BINARY_DIR}/HunterSetup.cmake.in
  "include(${CMAKE_BINARY_DIR}/HunterGate.cmake)\n"
  "HunterGate(
    URL ${HUNTER_URL}
    SHA1 ${HUNTER_SHA1}
    FILEPATH ${CMAKE_BINARY_DIR}/HunterConfig.cmake.in
   )\n"
)

configure_file(
  ${CMAKE_BINARY_DIR}/HunterSetup.cmake.in
  ${CMAKE_BINARY_DIR}/HunterSetup.cmake @ONLY)

# Build only the current CMAKE_BUILD_TYPE
set(HUNTER_CONFIGURATION_TYPES ${CMAKE_BUILD_TYPE} CACHE STRING "" FORCE)

include(${CMAKE_BINARY_DIR}/HunterSetup.cmake)

#
# Create a toolchain file
#

file(WRITE ${CMAKE_BINARY_DIR}/HunterToolchain.cmake.in
[=[
# Add hunter install directory to the find_package variables
if (EXISTS "${CMAKE_BINARY_DIR}/_3rdParty/Hunter/install-root-dir")
  file(READ ${CMAKE_BINARY_DIR}/_3rdParty/Hunter/install-root-dir HunterInstall)
  list(APPEND CMAKE_FIND_ROOT_PATH \"${HunterInstall}\")
  list(APPEND CMAKE_PREFIX_PATH \"${HunterInstall}\")
endif()
]=])

if(ANDROID_ABI AND ANDROID_NATIVE_API_LEVEL)
  file(APPEND ${CMAKE_BINARY_DIR}/HunterToolchain.cmake.in
    "set(ANDROID_ABI ${ANDROID_ABI})\n"
    "set(ANDROID_NATIVE_API_LEVEL ${ANDROID_NATIVE_API_LEVEL})\n"
    # Android has RPATH, and we dont' have to relink when installed\n"
    "set(CMAKE_BUILD_WITH_INSTALL_RPATH ON)\n")
endif()

if (CMAKE_TOOLCHAIN_FILE)
  file(APPEND ${CMAKE_BINARY_DIR}/HunterToolchain.cmake.in
    "include(\"${CMAKE_TOOLCHAIN_FILE}\")\n")
endif()

configure_file(
  ${CMAKE_BINARY_DIR}/HunterToolchain.cmake.in
  ${CMAKE_BINARY_DIR}/HunterToolchain.cmake @ONLY)

# Setup a project, and add the packages
project(HunterSetup)

#
# Building the packages
#

# Build the hunter packages with the ANDROID_ABI / ANDROID_NATIVE_API_LEVEL settings
if (CMAKE_TOOLCHAIN_FILE)
  set(ORIGINAL_TOOLCHAIN_FILE ${CMAKE_TOOLCHAIN_FILE})
  set(CMAKE_TOOLCHAIN_FILE ${CMAKE_BINARY_DIR}/HunterToolchain.cmake)
endif()

foreach(package IN LISTS HUNTER_PACKAGES)
  unset(_package_components)
  if (HUNTER_${package}_COMPONENTS)
    set(_package_components COMPONENTS ${HUNTER_${package}_COMPONENTS})
  endif()
  hunter_add_package(${package} ${_package_components})
endforeach()

# Restore the original TOOLCHAIN_FILE
if (ORIGINAL_TOOLCHAIN_FILE)
  set(CMAKE_TOOLCHAIN_FILE "${ORIGINAL_TOOLCHAIN_FILE}")
endif()

#
# Make sure the parent find_package calls work as expected
#

set(CMAKE_FIND_ROOT_PATH ${CMAKE_FIND_ROOT_PATH} PARENT_SCOPE)
set(CMAKE_PREFIX_PATH ${CMAKE_PREFIX_PATH} PARENT_SCOPE)
