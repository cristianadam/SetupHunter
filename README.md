# SetupHunter
Easy setup of [Hunter Package Manager](https://github.com/cpp-pm/hunter) via [FetchContent](https://cmake.org/cmake/help/latest/module/FetchContent.html).

The project will download the latest [HunterGate.cmake](https://github.com/cpp-pm/gate), and the latest [packages archive release](https://github.com/cpp-pm/hunter/releases/) so that the project is easy to setup, and always up to date!

SetupHunter will also issue `hunter_add_package` to all packages specified in `HUNTER_PACKAGES` list.

For every package it will create a `hunter_config` in which `VERSION` and `CMAKE_ARGS` variables will be set. SetupHunter will check for `HUNTER_<package>_VERSION|CMAKE_ARGS`
variables for appropriate values.

SetupHunter will set the `CMAKE_BUILD_TYPE` as `HUNTER_CONFIGURATION_TYPES`, so that if you're building `Release` you won't be getting a `Debug` build of the packages.

# Usage

Setting up Hunter, and adding a package is as easy as:

```cmake
# Setting up dlib as an external package
set(HUNTER_PACKAGES dlib)

include(FetchContent)
FetchContent_Declare(SetupHunter GIT_REPOSITORY https://github.com/cristianadam/SetupHunter)
FetchContent_MakeAvailable(SetupHunter)

# Using the dlib package
find_package(dlib REQUIRED)
target_link_libraries(myapp PRIVATE dlib::dlib)
```

# Usage with custom CMake variables per package
```cmake
set(HUNTER_pcre2_CMAKE_ARGS
  PCRE2_BUILD_PCRE2_8=OFF
  PCRE2_BUILD_PCRE2_16=ON
  PCRE2_BUILD_PCRE2_32=OFF
  PCRE2_SUPPORT_JIT=ON)

set(HUNTER_PACKAGES pcre2)
```

# Usage with package components
```cmake
set(HUNTER_Boost_COMPONENTS Filesystem Python)
set(HUNTER_PACKAGES Boost)
```

# Usage with specific package version
```cmake
set(HUNTER_fmt_VERSION 5.3.0)
set(HUNTER_PACKAGES fmt)
```

# Usage with specific Hunter Gate
```cmake
set(HUNTER_URL "https://github.com/cpp-pm/hunter/archive/v0.23.222.tar.gz")
set(HUNTER_SHA1 "0b88baaa2a9a35b8ce632c57ade66be8dd918afd")

include(FetchContent)
FetchContent_Declare(SetupHunter GIT_REPOSITORY https://github.com/cristianadam/SetupHunter)
FetchContent_MakeAvailable(SetupHunter)
```

# Usage with classic Hunter setup
```cmake
include(FetchContent)
FetchContent_Declare(SetupHunter GIT_REPOSITORY https://github.com/cristianadam/SetupHunter)
FetchContent_MakeAvailable(SetupHunter)

hunter_add_package(ZLIB)
find_package(ZLIB CONFIG REQUIRED)

add_executable(boo main.c)
target_link_libraries(boo PRIVATE ZLIB::zlib)
```

# Minimum CMake version

[FetchContent](https://cmake.org/cmake/help/latest/module/FetchContent.html) is avaiable since CMake 3.11.
