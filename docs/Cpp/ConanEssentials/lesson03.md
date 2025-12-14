# 3. The Flexibility of Using conanfile.py vs conanfile.txt

## From conanfile.txt to .py

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

- conanfile.py

```python
from conan import ConanFile
from conan.tools.cmake import cmake_layout

class FormatterRecipe(ConanFile):
    settings = "os", "compiler", "build_type", "arch"
    generators = "CMakeToolchain", "CMakeDeps"

    def requirements(self):
        self.requires("fmt/11.2.0")
    
    def layout(self):
        cmake_layout(self)
```

## Adding conditional requirements

- conanfile_complete.py

```python
from conan import ConanFile
from conan.tools.cmake import cmake_layout, CMakeToolchain, CMakeDeps
from conan.tools.build import valid_min_cppstd
from conan.errors import ConanInvalidConfiguration


class FormatterRecipe(ConanFile):
    settings = "os", "compiler", "build_type", "arch"

    # 1. Add an option to choose between fmt and C++ 20's std::format.
    options = {"with_std_format": [True, False]}
    default_options = {"with_std_format": False}

    def requirements(self):
        if not self.options.with_std_format:
            self.requires("fmt/11.2.0")

    # 2. Pass this information to the build system using the generate method.
    def generate(self):
        tc = CMakeToolchain(self)
        tc.cache_variables["USE_STD_FORMAT"] = self.options.with_std_format
        tc.generate()

        cd = CMakeDeps(self)
        cd.generate()

    # 3. Validate the configuration to ensure compatibility with C++ 20.
    def validate(self):
        if (self.options.with_std_format and not valid_min_cppstd(self, 20)):
            raise ConanInvalidConfiguration("std::format requires C++20")

    def layout(self):
        cmake_layout(self)
```

- CMakeListx.txt

```
cmake_minimum_required(VERSION 3.15)
project(formatter CXX)

add_executable(formatter src/main.cpp)

if (USE_STD_FORMAT)
    target_compile_definitions(formatter PRIVATE USE_STD_FORMAT)
else()
    find_package(fmt REQUIRED)
    target_link_libraries(formatter fmt::fmt)
endif()
```

- src/main.cpp

```cpp
#ifdef USE_STD_FORMAT
    #include <format>
    #include <iostream>
#else
    #include <fmt/core.h>
    #include <fmt/color.h>
#endif

int main() {

#ifdef USE_STD_FORMAT
    std::cout << std::format("Conan is a MIT-licensed, Open Source package manager for C and C++ development\n");
#else
    fmt::print(fmt::fg(fmt::color::cyan) | fmt::emphasis::bold, 
               "Conan is a MIT-licensed, Open Source ");
    fmt::print(fmt::fg(fmt::color::white),
               "package manager for C and C++ development\n");
#endif
    return 0;
}
```

## Running out project

```
# conan install conanfile_complete.py --build=missing -o="&:with_std_format=True"
conan install conanfile_complete.py --build=missing -o="&:with_std_format=True" -s="compiler.cppstd=20"

cmake --build --preset conan-release

cmake --build --preset conan-release

./build/Release/formatter
```