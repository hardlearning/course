# 8. Creating your first Conan package

## Project Setup

```
# Create a template project of a CMake library
conan new cmake_lib -d name=hello -d version=1.0
```

## Project Structure

```
.
├── CMakeLists.txt
├── conanfile.py
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

## Creating out first package

Execution order:
1. exports sources
2. config_options()
3. configure()
4. layout()
5. generate()
6. build()
7. package()
8. package_info()
9. Launch test_package

```
# Create the package with default configuration
conan create .
```

## Listing our created package

```
# List the created packages
conan list "*:*"
```