## Hunter Gate
This project has been upstreamed to https://github.com/cpp-pm/gate!

# SetupHunter
Easy setup of [Hunter Package Manager](https://github.com/cpp-pm/hunter) via [FetchContent](https://cmake.org/cmake/help/latest/module/FetchContent.html).

The project will download the latest [HunterGate.cmake](https://github.com/cpp-pm/gate), and the latest [packages archive release](https://github.com/cpp-pm/hunter/releases/) so that the project is easy to setup, and always up to date!

SetupHunter will also issue `hunter_add_package` to all packages specified in `HUNTER_PACKAGES` list.

For every package it will create a `hunter_config` in which `VERSION` and `CMAKE_ARGS` variables will be set. SetupHunter will check for `HUNTER_<package>_VERSION|CMAKE_ARGS`
variables for appropriate values.

SetupHunter will set the `CMAKE_BUILD_TYPE` as `HUNTER_CONFIGURATION_TYPES`, so that if you're building `Release` you won't be getting a `Debug` build of the packages.

SetupHunter will create a `HunterToolchain.cmake` file which will set up the `CMAKE_PREFIX_PATH` and `CMAKE_FIND_ROOT_PATH` variables with the installed path of the Hunter packages.
If a `CMAKE_TOOLCHAIN_FILE` was used during compilation, it will be included in the `HunterToolchain.cmake` file.

## Usage

Setting up Hunter, and adding a package list is as easy as:

```cmake
# FetchContent is available since CMake version 3.11
cmake_minimum_required(VERSION 3.11)

# Setting up dlib as an external package
set(HUNTER_PACKAGES dlib)

include(FetchContent)
FetchContent_Declare(SetupHunter GIT_REPOSITORY https://github.com/cristianadam/SetupHunter)
FetchContent_MakeAvailable(SetupHunter)

# Using the dlib package
find_package(dlib REQUIRED)
target_link_libraries(myapp PRIVATE dlib::dlib)
```

## Usage with custom CMake variables per package
```cmake
set(HUNTER_pcre2_CMAKE_ARGS
  PCRE2_BUILD_PCRE2_8=OFF
  PCRE2_BUILD_PCRE2_16=ON
  PCRE2_BUILD_PCRE2_32=OFF
  PCRE2_SUPPORT_JIT=ON)

set(HUNTER_PACKAGES pcre2)
```

## Usage with package components
```cmake
set(HUNTER_Boost_COMPONENTS Filesystem Python)
set(HUNTER_PACKAGES Boost)
```

## Usage with specific package version
```cmake
set(HUNTER_fmt_VERSION 5.3.0)
set(HUNTER_PACKAGES fmt)
```

## Usage with specific Hunter Gate
```cmake
set(HUNTER_URL "https://github.com/cpp-pm/hunter/archive/v0.23.224.tar.gz")
set(HUNTER_SHA1 "18e57a43efc435f2e1dae1291e82e42afbf940be")

set(HUNTER_PACKAGES fmt ZLIB)

include(FetchContent)
FetchContent_Declare(SetupHunter GIT_REPOSITORY https://github.com/cristianadam/SetupHunter)
FetchContent_MakeAvailable(SetupHunter)
```

## Usage with HunterToolchain.cmake

This allows building the 3rd party packages separate than the user projects, and simply use CMake's
`find_package` with no usage of Hunter at all.

```cmake
cmake_minimum_required(VERSION 3.11)

set(HUNTER_PACKAGES freetype ZLIB PNG double-conversion pcre2)

set(HUNTER_pcre2_CMAKE_ARGS
    PCRE2_BUILD_PCRE2_8=OFF
    PCRE2_BUILD_PCRE2_16=ON
    PCRE2_BUILD_PCRE2_32=OFF
    PCRE2_SUPPORT_JIT=ON)

include(FetchContent)
FetchContent_Declare(SetupHunter GIT_REPOSITORY https://github.com/cristianadam/SetupHunter)
FetchContent_MakeAvailable(SetupHunter)

project(3rdparty)
```

Then compile your project with:
```
cmake -DCMAKE_TOOLCHAIN_FILE=/path/to/your/3rdparty/build/HunterToolchain.cmake
```

## Usage with classic Hunter setup
```cmake

file(WRITE ${CMAKE_BUILD_DIR}/HunterConfig.cmake [=[
hunter_config(zlib VERSION 1.2.8)

hunter_config(pcre2
  VERSION ${HUNTER_pcre2_VERSION}
  CMAKE_ARGS
    PCRE2_BUILD_PCRE2_8=OFF
    PCRE2_BUILD_PCRE2_16=ON
    PCRE2_BUILD_PCRE2_32=OFF
    PCRE2_SUPPORT_JIT=ON
)
]=])

set(HUNTER_URL "https://github.com/cpp-pm/hunter/archive/v0.23.224.tar.gz")
set(HUNTER_SHA1 "18e57a43efc435f2e1dae1291e82e42afbf940be")
set(HUNTER_FILEPATH_CONFIG ${CMAKE_BUILD_DIR}/HunterConfig.cmake)

include(FetchContent)
FetchContent_Declare(SetupHunter GIT_REPOSITORY https://github.com/cristianadam/SetupHunter)
FetchContent_MakeAvailable(SetupHunter)

hunter_add_package(ZLIB)
hunter_add_package(pcre2)

find_package(ZLIB CONFIG REQUIRED)
find_package(pcre2 CONFIG REQUIRED)

add_executable(boo main.c)
target_link_libraries(boo PRIVATE ZLIB::zlib PCRE2::PCRE2)
```
