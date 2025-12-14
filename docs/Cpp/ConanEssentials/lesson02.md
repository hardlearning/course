# 2. Building for Multiple Configuration with Conan and CMake Presets

## Check the changes in the code

- src/main.cpp

```cpp
#include <fmt/core.h>
#include <fmt/color.h>

int main() {
    fmt::print(fmt::fg(fmt::color::cyan) | fmt::emphasis::bold, 
               "Conan is a MIT-licensed, Open Source ");
    fmt::print(fmt::fg(fmt::color::white), 
               "package manager for C and C++ development\n");

    #ifdef NDEBUG
    fmt::print(fmt::fg(fmt::color::green) | fmt::emphasis::italic,
               "Release configuration!\n");
    #else
    fmt::print(fmt::fg(fmt::color::yellow) | fmt::emphasis::italic,
               "Debug configuration!\n");
    #endif

    return 0;
}
```

## Creating a Debug Profile

```
conan config home
cp /home/training/.conan2/profiles/default /home/training/.conan2/profiles/debug
nano /home/training/.conan2/profiles/debug
```

Change the `build_type` to `Debug`:

```
[settings]
arch=x86_64
build_type=Debug
compiler=gcc
compiler.cppstd=gnu14
compiler.libcxx=libstdc++11
compiler.version=10
os=Linux
```

## Settings: Build the Project in Debug

```
conan install . --build=missing -pr=debug
```

You could also pass the `build_type` in the command line using `-s="build type=Debug"` instead of passing the profile.

```
cmake --preset=conan-debug
# If using Windows or Multi Config generators, run 
# cmake --preset=conan-default

cmake --build --preset=conan-debug
./build/Debug/formatter
```

## Options: Use Dependencies as Shared Libraries

``` 
conan install . --build=missing --options="*:shared=True"
```

You could also set this option from the `[options]` section of the profile

```
cmake --preset=conan-default
cmake --build --preset=conan-release
# ./build/Release/formatter this would fail without the conanrun

# To activate virtual evironment
. build/Release/generators/conanrun.sh
# If using Windows CMD, run 
# build\generators\conanrun.bat

./build/Release/formatter

# To deactivate virtual evironment
./build/generators/deactivate_conanrun.sh
```

If using Powershell, add --conf to generate .ps1 files:

```
conan install . --build=missing --options="*:shared=True" --conf="tools.env.virtualenv:powershell=pwsh"
```

## Difference between Settings and Options

- Settings: Global configurations (e.g., OS, compiler, build type)
  - `conan install . --profile=debug`
  - `conan install . --settings="build type=Debug"`
- Options: Package-specific feature (e.g., shared or static linkage, specific package options)
  - `conan install . --options="*:shared=True"`
  - `conan install . profile=release-shared`