# 15. Creating a recipe for prebuilt binaries

## Packaging prebuild binaries from a local directory

- prebuilt/conanfile.py

```python
import os
from conan import ConanFile
from conan.tools.files import copy


class helloRecipe(ConanFile):
    name = "hello"
    version = "0.1"
    settings = "os", "arch"

    def layout(self):
        _os = str(self.settings.os).lower()
        _arch = str(self.settings.arch).lower()
        self.folders.build = os.path.join("vendor_hello_library", _os, _arch)

    def package(self):
        copy(self, "*.h", os.path.join(self.build_folder, "include"),
            os.path.join(self.package_folder, "include"), keep_path=False)
        copy(self, "*.lib", self.build_folder,
            os.path.join(self.package_folder, "lib"), keep_path=False)
        copy(self, "*.a", self.build_folder,
            os.path.join(self.package_folder, "lib"), keep_path=False)

    def package_info(self):
        self.cpp_info.libs = ["hello"]
```

```
# inspect the vendor folder
cd prebuilt && tree vendor_hello_library

# Create packages for different configurations
conan export-pkg . -s os='Linux' -s arch='x86_64'

conan export-pkg . -s os='Linux' -s arch='armv8'

# Inspect the packages created in the cache
conan list hello/0.1#:*
```

## Packaging prebuild binaries from a remote repository

- prebuilt_remote/conanfile.py

```python
import os
from conan.tools.files import get, copy
from conan import ConanFile


class HelloConan(ConanFile):
    name = "hello"
    version = "0.1"
    settings = "os", "arch"

    def build(self):
        base_url = "https://github.com/conan-io/libhello/releases/download/0.0.1/"
        _os = {
            "Windows": "win",
            "Linux": "linux",
            "Macos": "macos"}.get(str(self.settings.os))
        _arch = str(self.settings.arch).lower()
        url = "{}/{}_{}.tgz".format(base_url, _os, _arch)
        get(self, url)

    def package(self):
        copy(self, "*.h", self.build_folder,
            os.path.join(self.package_folder, "include"))
        copy(self, "*.lib", self.build_folder,
            os.path.join(self.package_folder, "lib"))
        copy(self, "*.a", self.build_folder,
            os.path.join(self.package_folder, "lib"))

    def package_info(self):
        self.cpp_info.libs = ["hello"]
```

```
cd prebuilt_remote

# Create packages with binaries from a remote
conan create . -s os='Linux' -s arch='x86_64'

conan create . -s os='Linux' -s arch='armv8'

# List the packages created in the cache
conan list hello/0.1#:*
```

## Packaging prebuild binaries while developing locally

- development/mylib/conanfile.py

```python
from conan import ConanFile
from conan.tools.cmake import CMakeToolchain, CMake, cmake_layout, CMakeDeps


class MyLib(ConanFile):
    name = "mylib"
    version = "1.0.0"
    package_type = "library"

    # Optional metadata
    license = "<Put the package license here>"
    author = "<Put your name here> <And your email here>"
    url = "<Package recipe repository url here, for issues about the package>"
    description = "<Description of mylib package here>"
    topics = ("<Put some tag here>", "<here>", "<and here>")

    # Binary configuration
    settings = "os", "compiler", "build_type", "arch"
    options = {"shared": [True, False], "fPIC": [True, False]}
    default_options = {"shared": False, "fPIC": True}

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
    
    def generate(self):
        deps = CMakeDeps(self)
        deps.generate()
        tc = CMakeToolchain(self)
        tc.generate()

    def build(self):
        cmake = CMake(self)
        cmake.configure()
        cmake.build()

    def package(self):
        cmake = CMake(self)
        cmake.install()

    def package_info(self):
        self.cpp_info.libs = ["mylib"]
```

- src/mylib.cpp

```cpp
#include <iostream>
#include "mylib.h"

void mylib() {
    std::cout << "Hi there! This is mylib" << std::endl;
}
```

```
cd development/mylib

# Install and build normally
conan install .

cmake --preset conan-release

cmake --build --preset conan-release

# Export the binaries to the cache
conan export-pkg .

cd ../consumer
# Check that the consumer works
conan create .
```