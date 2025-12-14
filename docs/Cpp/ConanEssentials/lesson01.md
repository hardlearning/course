# 1. Building a Simple CMake Project Using Conan

## Setting up the project's code

- src/main.cpp

```cpp
#include <fmt/core.h>
#include <fmt/color.h>

int main() {
    fmt::print(fmt::fg(fmt::color::cyan) | fmt::emphasis::bold, 
               "Conan is a MIT-licensed, Open Source ");
    fmt::print(fmt::fg(fmt::color::white), 
               "package manager for C and C++ development\n");
    return 0;
}
```

- CMakeLists.txt

```
cmake_minimum_required(VERSION 3.15)
project(formatter CXX)

find_package(fmt REQUIRED)

add_executable(formatter src/main.cpp)
target_link_libraries(formatter fmt::fmt)
```

- conanfile.txt

```
[requires]
fmt/11.2.0

[generators]
CMakeDeps
CMakeToolchain

[layout]
cmake_layout
```

## Understanding Conan Profiles

```
# generate default conan profile
conan profile detect --force
# check the location of the conan home in your system
conan config home
```

## Install Dependencies with Conan

```
conan install . --build=missing
```

## Building and Running the Application

- Linux and macOS

```
# show configure presets
cmake --list-presets

# configure the project with conan-release preset
cmake --preset=conan-release
# If using Windows or Multi Config generators
# cmake --preset=conan-default

# build the project with conan-release preset
cmake --build --preset=conan-release
# run the application
./build/Release/formatter
```
