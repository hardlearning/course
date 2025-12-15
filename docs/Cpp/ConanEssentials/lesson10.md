# 10. The package() and package_info() methods

## Project Setup

```
.
├── CMakeLists.txt
├── conanfile.py
├── license.txt
├── include
│   └── hello.h
├── src
│   └── hello.cpp
└── test_package
    ├── CMakeLists.txt
    ├── conanfile.py
    └── src
        └── example.cpp
```

## Check the changes in the recipe

- conanfile.py

```python
import os

from conan import ConanFile
from conan.tools.cmake import CMakeToolchain, CMake, cmake_layout, CMakeDeps
from conan.tools.files import copy


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
    options = {"shared": [True, False], "fPIC": [True, False]}
    default_options = {"shared": False, "fPIC": True}

    # Sources are located in the same place as this recipe, copy them to the recipe
    exports_sources = "CMakeLists.txt", "src/*", "include/*", "license.txt"

    def config_options(self):
        if self.settings.os == "Windows":
            self.options.rm_safe("fPIC")

    def configure(self):
        if self.options.shared:
            self.options.rm_safe("fPIC")

    def layout(self):
        cmake_layout(self)
    
    def generate(self):
        deps = CMakeDeps(self)
        deps.generate()
        tc = CMakeToolchain(self)
        tc.generate()

    def build(self):
        cmake = CMake(self)
        cmake.configure()
        cmake.build()

    # 1. The package() method
    def package(self):
        copy(self,
             "license.txt",
             src=self.source_folder,
             dst=os.path.join(self.package_folder, "licenses"))
        cmake = CMake(self)
        cmake.install()

    # 2. The package_info() method
    def package_info(self):
        # Binaries to link
        self.cpp_info.libs = ["hellolib"]
        # Directories (values by default)
        # self.cpp_info.includedirs = ['include']  # Ordered list of include paths
        # self.cpp_info.libdirs = ['lib']  # Where libraries can be found
        # self.cpp_info.bindirs = ['bin']  # Where executables and shared libs can be found
```

Define library information in cpp_info

```python
# Binaries to link
self.cpp_info.libs = []  # Libs to link
self.cpp_info.system_libs = []  # System libs
self.cpp_info.frameworks = []  # 0SX frameworks
self.cpp_info.objects = []  # 0bject files(.obj, .o)
# Directories
self.cpp_info.includedirs = ['include']  # Include paths
self.cpp_info.libdirs = ['lib']  # Library paths
self.cpp_info.bindirs = ['bin']  # Binary/shared lib paths
self.cpp_info.resdirs = []  # Resource paths
self.cpp_info.srcdirs = []  # Source paths(debug)
self.cpp_info.builddirs = []  # Build script paths
self.cpp_info.frameworkdirs = []  # OSX framework paths
# Flags
self.cpp_info.defines = []  # Defines
self.cpp_info.cflags = []  # C flags
self.cpp_info.cxxflags = []  # C++ flags
self.cpp_info.sharedlinkflags = []  # Shared linker flags
self.cpp_info.exelinkflags = []  # Executable linker flags
```

## Example for package() & pachage_info() methods

```
# Create the package, it will fail
conan create .

# Check how it fails, then change hellolib for hello in the conanfile.py
# Create the package again, it succeeds
conan create .
```