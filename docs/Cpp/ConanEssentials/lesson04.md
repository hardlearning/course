# 4. Using build tools as Conan packages

## Check the changes in the code

- CMakeLists.txt

```
cmake_minimum_required(VERSION 3.15)
project(formatter CXX)

find_package(fmt REQUIRED)

message(NOTICE "==> Building with CMake version: ${CMAKE_VERSION} <== ")

add_executable(formatter src/main.cpp)
target_link_libraries(formatter fmt::fmt)
```

- conanfile.py

```python
from conan import ConanFile
from conan.tools.cmake import cmake_layout

class FormatterRecipe(ConanFile):
    settings = "os", "compiler", "build_type", "arch"
    generators = "CMakeToolchain", "CMakeDeps"

    def requirements(self):
        self.requires("fmt/11.2.0")

    def build_requirements(self):
        self.tool_requires("cmake/3.31.5")

    def layout(self):
        cmake_layout(self)
```

On conanfile.txt you can declare this requirement in the `[tool_requires]` section

## What are build requirements

- Dependencies for building
- Compilers, build systems, and tools
- Version-specific or missing tools
- Similar to dev dependencies
- Ensures reproducible builds

## Using build requirements

```
conan install . --build=missing

# This version should be the one installed in your system
cmake --version  

# By activating the conanbuild environment, the new cmake version will correspond to the one listed in build requirements
source build/Release/generators/conanbuild.sh
# Windows
# ./build/Release/generators/conanbuild.bat

# This version will be the one specified in our recipe
cmake --version

cmake --preset=conan-release
cmake --build --preset=conan-release
./build/Release/formatter
```