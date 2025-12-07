# Non-Type Template Arguments

- Expression that is computed at compile time within atemplate argument list
- Must be a constant expression (addresses, references,integrals, nullptr, enums)
- Part of the template type
- Used by std::begin & std::end functions

- 非类型参数必须是常量表达式

```cpp
#include <iostream>

template<int size>
void Print() {
    // size++; // error
    char buffer[size];
    std::cout << size << std::endl;
}

int main() {
    Print<3>(); // ok
    int i = 3;
    // Print<i>(); // error
    Print<sizeof(i)>(); // ok
    return 0;
}
```

- 使用非类型模板参数指定数组大小

```cpp
#include <iostream>

// 显式指定数组大小
template<typename T>
T Sum(T *parr, int size) {
    T sum{};
    for (int i = 0; i < size; ++i) {
        sum += parr[i];
    }
    return sum;
}

// 使用非类型模板参数
template<typename T, int size>
T Sum(T (&parr)[size]) {
    T sum{};
    for (int i = 0; i < size; ++i) {
        sum += parr[i];
    }
    return sum;
}

int main() {
    int arr[]{ 3, 1, 9, 7 };
    int sum = Sum(arr, 4);
    std::cout << sum << std::endl;

    // 数组的引用
    int (&ref)[4] = arr;
    int sum2 = Sum(arr);
    std::cout << sum2 << std::endl;
    return 0;
}
```