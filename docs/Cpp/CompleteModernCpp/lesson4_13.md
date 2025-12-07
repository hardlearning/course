# Type Traits

- Type traits give the ability to introspect
    - find the characteristics of types at compile time
    - transform the properties of the type
- Useful in template metaprogramming
- Will either return a boolean or a type when inspecting types
- Provides template-based interface and defined in header `<type_traits>`
- Some traits require support from the compiler
    - compiler provides intrinsics for such traits

```cpp
is_void
is_null_pointer
is_integral
is_floating_point
is_array
is_enum
is_union
is_class
is_function
is_pointer
```

- 类型特征简单使用

```cpp
#include <iostream>
#include <type_traits>

template<typename T>
T Divide(T a, T b) {
    if (std::is_floating_point<T>::value == false) {
        std::cout << "Use floating point types only\n";
        return 0;
    }
    return a / b;
}

template<typename T>
void Check(T&&) {
    std::cout << std::boolalpha;
    std::cout << "Is reference? " << std::is_reference<T>::value << std::endl;

    std::cout << "After removing:" << std::is_reference<typename std::remove_reference<T>::type>::value << std::endl;
}

int main() {
    std::cout << std::boolalpha << "Is integer? "
    << std::is_integral<int>::value << std::endl;

    std::cout << Divide(5.1, 7.3) << std::endl;

    Check(5);
    int value{};
    Check(value);
    return 0;
}
```