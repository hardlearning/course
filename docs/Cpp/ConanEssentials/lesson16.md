# 16. Creating Tool Require Packages

## Anatomy of a Tool Requires Recipe

- conanfile.py

```python
import os
from conan import ConanFile
from conan.tools.cmake import CMakeToolchain, CMake, cmake_layout
from conan.tools.files import copy


class SecureScannerRecipe(ConanFile):
    name = "secure-scanner"
    version = "1.0"
    package_type = "application"

    settings = "os", "compiler", "build_type", "arch"

    exports_sources = "CMakeLists.txt", "src/*"

    def layout(self):
        cmake_layout(self)

    def generate(self):
        tc = CMakeToolchain(self)
        tc.generate()

    def build(self):
        cmake = CMake(self)
        cmake.configure()
        cmake.build()

    def package(self):
        copy(self, f"*secure-scanner*", self.build_folder,
             os.path.join(self.package_folder, "bin"), keep_path=False)

    def package_info(self):
        self.buildenv_info.define("MY_VAR", "42")

    def package_id(self):
        del self.info.settings.compiler
        del self.info.settings.build_type
```

## Testing and Consuming the Tool

- test_package/conanfile.py

```python
from conan import ConanFile


class SecureScannerTestRecipe(ConanFile):
    settings = "os", "compiler", "build_type", "arch"

    def build_requirements(self):
        self.tool_requires(self.tested_reference_str)

    def build(self):
        var_cmd = (
            "set MY_VAR"
            if self.settings_build.os == "Windows"
            else "echo MY_VAR=$MY_VAR"
        )
        self.run(var_cmd)

    def test(self):
        self.run("secure-scanner mypath")
```

## Creating the Tool Require

```
# create the tool requires for Release, use the --build-require argument 
conan create . --build-require -s:b build_type=Release

# check package id 
conan list 'secure-scanner/1.0#*:*'

# create the tool requires for Debug, use the --build-require argument 
conan create . --build-require -s:b build_type=Debug

# check package id, it will be the same
conan list 'secure-scanner/1.0#*:*'
```