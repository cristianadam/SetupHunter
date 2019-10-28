# SetupHunter
Easy setup of [Hunter Package Manager](https://github.com/cpp-pm/hunter) via [FetchContent](https://cmake.org/cmake/help/latest/module/FetchContent.html).

The project will download the latest [HunterGate.cmake](https://github.com/cpp-pm/gate), and the latest [packages archive release](https://github.com/cpp-pm/hunter/releases/) so that the project is easy to setup and always up to date!

# Usage

Setting up the latest HunterGate, and adding a package is as easy as:

```cmake
# Setting up dlib as an external package
set(HUNTER_PACKAGES dlib)

include(FetchContent)
FetchContent_Declare(HunterSetup GIT_REPOSITORY https://github.com/cristianadam/SetupHunter.git)
FetchContent_MakeAvailable(HunterSetup)

# Using 
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
