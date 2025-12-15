# 12. Testing Conan packages

- test_package/src/example.cpp

```cpp
#include "hello.h"
#include <vector>
#include <string>

int main() {
    hello();

    std::vector<std::string> vec;
    vec.push_back("test_package");

    hello_print_vector(vec);
}
```

- test_package/CMakeLists.txt

```
cmake_minimum_required(VERSION 3.15)
project(PackageTest CXX)

find_package(hello CONFIG REQUIRED)

add_executable(example src/example.cpp)
target_link_libraries(example hello::hello)
```

- test_package/conanfile.py

```python
import os

from conan import ConanFile
from conan.tools.cmake import CMake, cmake_layout
from conan.tools.build import can_run


class helloTestConan(ConanFile):
    settings = "os", "compiler", "build_type", "arch"
    generators = "CMakeDeps", "CMakeToolchain"

    def requirements(self):
        self.requires(self.tested_reference_str)

    def build(self):
        cmake = CMake(self)
        cmake.configure()
        cmake.build()

    def layout(self):
        cmake_layout(self)

    def test(self):
        if can_run(self):
            cmd = os.path.join(self.cpp.build.bindir, "example")
            self.run(cmd, env="conanrun")
```

## Running with conan test

```
# test the already created package for the default profile
conan test test_package hello/1.0

# test the already created package for Debug
# use a profile or you can also use -s="build_type=Debug"
conan test test_package hello/1.0 --profile=./debug 
```