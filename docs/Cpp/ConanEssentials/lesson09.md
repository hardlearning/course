# 9. Dependencies, generators and building

## Check the code changes

- src/hello.cpp

```cpp
#include <iostream>
#include "hello.h"

#if USING_FMT == 1
#include <fmt/color.h>
#endif

void hello() {
    #if USING_FMT == 1
        fmt::print(fg(fmt::color::crimson) | fmt::emphasis::bold, 
                   "hello/1.0: Hello World! (with color!)\n");
    #else
        std::cout << "hello/1.0: Hello World! (without color)" 
                  << std::endl;
    #endif
}
```

- CMakeLists.txt

```
cmake_minimum_required(VERSION 3.15)
project(hello CXX)

add_library(hello src/hello.cpp)
target_include_directories(hello PUBLIC include)

set_target_properties(hello PROPERTIES PUBLIC_HEADER "include/hello.h")

if (WITH_FMT)
    find_package(fmt)
    target_link_libraries(hello fmt::fmt)
    target_compile_definitions(hello PRIVATE USING_FMT=1)
endif()

install(TARGETS hello)
```

- conanfile.py

```python
from conan import ConanFile
from conan.tools.cmake import CMakeToolchain, CMake, cmake_layout, CMakeDeps


class helloRecipe(ConanFile):
    name = "hello"
    version = "1.0"
    package_type = "library"

    # Optional metadata
    license = "<Put the package license here>"
    author = "<Put your name here> <And your email here>"
    url = "<Package recipe repository url here, for issues about the package>"
    description = "<Description of hello package here>"
    topics = ("<Put some tag here>", "<here>", "<and here>")

    # Binary configuration
    settings = "os", "compiler", "build_type", "arch"

    options = {"shared": [True, False], 
               "fPIC": [True, False], 
               "with_fmt": [True, False]}

    default_options = {"shared": False, 
                       "fPIC": True, 
                       "with_fmt": True}

    # Sources are located in the same place as this recipe, copy them to the recipe
    exports_sources = "CMakeLists.txt", "src/*", "include/*"

    def config_options(self):
        if self.settings.os == "Windows":
            self.options.rm_safe("fPIC")

    def configure(self):
        if self.options.shared:
            self.options.rm_safe("fPIC")

    def layout(self):
        cmake_layout(self)

    # 1. Adding the fmt library dependency
    def requirements(self):
        if self.options.with_fmt:
            self.requires("fmt/11.2.0")

    # 2. Preparing the build: generate() method
    def generate(self):
        deps = CMakeDeps(self)
        deps.generate()
        tc = CMakeToolchain(self)
        if self.options.with_fmt:
            # Cache Variables, like WITH_FMT option, become -D definitions passed to the CMake command line during the configuration step.
            tc.cache_variables["WITH_FMT"] = True
            fmt_dep = self.dependencies["fmt"]
            self.output.info(fmt_dep.description)
            self.output.info(fmt_dep.license)
            self.output.info(fmt_dep.cpp_info.libdirs)
        tc.generate()

    # 3. The build() method and build helpers
    def build(self):
        cmake = CMake(self)
        # cmake -G ... -DCMAKE_TOOLCHAIN_FILE=... -DWITH_FMT="ON"
        cmake.configure()
        # cmake --build ...
        cmake.build()

    def package(self):
        cmake = CMake(self)
        cmake.install()

    def package_info(self):
        self.cpp_info.libs = ["hello"]
```

## Creating our pakcage

```
conan create . --build=missing
```
