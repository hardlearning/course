# 11. Configuring settings and options, and the package_id method

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
    options = {"shared": [True, False], "fPIC": [True, False]}
    default_options = {"shared": False, "fPIC": True}

    # Sources are located in the same place as this recipe, copy them to the recipe
    exports_sources = "CMakeLists.txt", "src/*", "include/*"

    # 1. The config_options() method
    def config_options(self):
        if self.settings.os == "Windows":
            self.options.rm_safe("fPIC")

    # 2. The configure() method
    def configure(self):
        if self.options.shared:
            self.options.rm_safe("fPIC")

    def layout(self):
        cmake_layout(self)
    
    def generate(self):
        deps = CMakeDeps(self)
        deps.generate()
        tc = CMakeToolchain(self)
        # 3. Accessing possibly deleted options
        tc.cache_variables["MY_EXAMPLE_VARIABLE"] = self.options.get_safe("fPIC")
        tc.generate()

    def build(self):
        cmake = CMake(self)
        cmake.configure()
        cmake.build()

    # 4. The pakcage_id() method
    def package_id(self):
        # We modify self.info to our liking
        pass

    def package(self):
        cmake = CMake(self)
        cmake.install()

    def package_info(self):
        self.cpp_info.libs = ["hello"]
```

The & before the option names is used to denote that the options should only affect the package we are creating

```
# If you try this on Windows, it will fail
conan create . -o='&:fPIC=True'

# Other platforms like Linux or macOS will succeed
conan create . -o='&:shared=True' -o='&:fPIC=True'
```

## Packaging applications

```python
def package id(self):
    # Another common setting to remove for applications
    # is self.info.build_type and always assume Release builds
    del self.info.compiler
```

## Packaging C libraries

```python
def configure(self):
    if self.options.shared:
        self.options.rm_safe("fPIC")
    self.settings.rm_safe("compiler.cppstd")
    self.settings.rm_safe("compiler.libcxx")
```

## Packaging header-only libraries

```python
class helloRecipe(ConanFile):
    name = "hello"
    version = "1.0"
    package_type = "header-library"

    settings = "os", "compiler", "build_type", "arch"

    def package_id(self):
        self.info.clear()

    ...
```