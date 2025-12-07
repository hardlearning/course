# Templates - Intruduction

## Function Templates

- Function that accepts template type arguments
- Always begins with template keyword
- Template type argument is called type name
- Type name is a placeholder for the actual type
- Can accept any type
- The template type can be used as return type

```cpp
template<typename T>
T Function(T arg) {
    // Implementation
}
```

```cpp
#include <iostream>

template<typename T>
T Max(T x, T y) {
    return x > y ? x : y;
}

int main() {
    // 函数模板被调用时才会生成对应代码Max<float>和Max<int>
    auto num = Max(3.3f, 5.8f);
    std::cout << num << std::endl;
    auto num2 = Max(38, 12);
    std::cout << num2 << std::endl;
    return 0;
}
```
