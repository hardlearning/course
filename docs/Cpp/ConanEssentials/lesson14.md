# 14. Creating a recipe for header-only libraries

## Project Setup

```
.
├── conanfile.py
├── include
│   └── hello.h
└── test_package
    ├── CMakeLists.txt
    ├── conanfile.py
    └── src
        └── example.cpp
```

- include/hello.h

```cpp
#pragma once

#include <iostream>

void hello(){
    std::cout << "hello/1.0 header only called" << std::endl;
}
```

- conanfile.py

```python
from conan import ConanFile
from conan.tools.layout import basic_layout
from conan.tools.files import copy

class helloRecipe(ConanFile):
    name = "hello"
    version = "1.0"
    package_type = "header-library"

    # Optional metadata
    license = "<Put the package license here>"
    author = "<Put your name here> <And your email here>"
    url = "<Package recipe repository url here, for issues about the package>"
    description = "<Description of hello package here>"
    topics = ("<Put some tag here>", "<here>", "<and here>")

    # Sources are located in the same place as this recipe, copy them to the recipe
    exports_sources = "include/*"

    def package_id(self):
        self.info.clear()

    def layout(self):
        basic_layout(self)

    def package(self):
        copy(self, "include/*", self.source_folder, self.package_folder)

    def package_info(self):
        self.cpp_info.bindirs = []
        self.cpp_info.libdirs = []
```

## Creating our package

```
conan create . -s=os=Linux

conan create . -pr=win-profile
```

The generated package ID is always the same.