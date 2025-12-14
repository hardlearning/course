# 5. Cross-Compiling Your Applications with Conan: Host and Build Contexts

## Check the changes in the code

- profiles/raspberry

```
[settings]
os=Linux
arch=armv7hf
compiler=gcc
build_type=Release
compiler.cppstd=gnu14
compiler.libcxx=libstdc++11
compiler.version=12

[buildenv]
CC=arm-linux-gnueabihf-gcc-12
CXX=arm-linux-gnueabihf-g++-12
LD=arm-linux-gnueabihf-ld
```

If using Windows, use the host profile at `profiles/raspberry-win` and install the cross-building toolchain indicated in the comments inside it.

- profiles/raspberry-win

```
# Download the toolchain (for example https://gnutoolchains.com/raspberry/)
# Install it and make it available in your path.
# Then use this as the host profile in the install command:
# $ conan install . --build missing --profile:build=default --profile:host=./profiles/raspberry-win

[settings]
os=Linux
arch=armv7hf
compiler=gcc
build_type=Release
compiler.cppstd=gnu14
compiler.libcxx=libstdc++11
compiler.version=12
[buildenv]
CC=arm-linux-gnueabihf-gcc-12
CXX=arm-linux-gnueabihf-g++
LD=arm-linux-gnueabihf-ld

[tool_requires]
# This will download the Ninja build tool (not needed if it is already install in your system)
*: ninja/1.12.1

[conf]
# This will set the Ninja as the generator for CMake
tools.cmake.cmaketoolchain:generator=Ninja
```

## The Two Profiles Model

```
conan install . --profile:build=default --profile:host=someprofile
```

- Build profile: <conan home>/profiles/default
- Host profile: <local folder>/profiles/raspberry

## Cross-building our project

```
# install the dependencies with two profiles, fmt will be built for the raspberry profile
conan install . --build missing --profile:build=default --profile:host=./profiles/raspberry

# activate the build environment so that we use the selected CMake version for building
source build/Release/generators/conanbuild.sh

# build our application for the Raspberry Pi
cmake --preset conan-release
cmake --build --preset conan-release

# check that we built the correct architecture
file ./build/Release/formatter
```