# 6. Introduction to versioning

## Recipe revisions

```
# List all the revisions of a given reference in a remote
conan list "fmt/11.0.2#*" -r=conancenter
```

## Version ranges

- name/[version-range]

|Range|Versions in|Versions out|
|--|--|--|
|fmt/[>=11 <12]|11.0,11.5|12.0|
|fmt/[~11.1]|11.1.0,11.1.3|11.2|
|fmt/[>=10 <11 \|\| >=11.1]|10.3,11.2|11.0|

- conanfile.py

```python
from conan import ConanFile
from conan.tools.cmake import cmake_layout

class FormatterRecipe(ConanFile):
    settings = "os", "compiler", "build_type", "arch"
    generators = "CMakeToolchain", "CMakeDeps"

    def requirements(self):
        self.requires("fmt/[~11]")

    def build_requirements(self):
        self.tool_requires("cmake/[>=3.23 <4]")

    def layout(self):
        cmake_layout(self)
```

```
# Install our version range dependencies
conan install . -b=missing
```

## Lockfiles

```
# Create a lockfile with the pinned versions
conan lock create .

cat conan.lock

# Install the dependencies once we add the version ranges back
# This time the lockfile will be picked up
conan install . -b=missing
```

Automatic lockfile detection can be suppressed by using the `--lock=""` argument