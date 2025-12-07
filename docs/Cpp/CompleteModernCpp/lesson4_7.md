# Variadic Templates

```cpp
#include <iostream>

// initializer_list要求类型必须统一
template<typename T>
void Print(std::initializer_list<T> args) {
    for (const auto &x: args) {
        std::cout << x << " ";
    }
}

int main() {
    Print({ 1, 2, 3, 4 });
    return 0;
}
```

- 可变参数模板

```cpp
#include <iostream>

void Print() {
    std::cout << std::endl;
}

// Template parameter pack
template<typename T, typename ...Params>
// Function parameter pack
void Print(T a, Params... args) {
    // std::cout << sizeof...(args) << std::endl;
    // std::cout << sizeof...(Params) << std::endl;
    std::cout << a;
    if (sizeof...(Params) != 0) {
        std::cout << ",";
    }
    Print(args...);
}
/*
1. Print(1, 2.5, 3, "4");
2. Print(2.5, 3, "4");
3. Print(3, "4");
4. Print("4");
5. Print();
 */
int main() {
    Print(1, 2.5, 3, "4");
    return 0;
}
```

- 使用常引用

```cpp
#include <iostream>
#include "Integer.h"

void Print() {
    std::cout << std::endl;
}

template<typename T, typename ...Params>
void Print(const T &a, const Params&... args) {
    std::cout << a;
    if (sizeof...(Params) != 0) {
        std::cout << ",";
    }
    Print(args...);
}

int main() {
    Integer val{ 1 };
    Print(0, val, Integer{ 2 });
    return 0;
}
```

- 使用右值引用作为参数

```cpp
#include <iostream>
#include "Integer.h"

void Print() {
    std::cout << std::endl;
}

template<typename T, typename ...Params>
void Print(T &&a, Params&&... args) {
    std::cout << a;
    if (sizeof...(Params) != 0) {
        std::cout << ",";
    }
    Print(std::forward<Params>(args)...);
}

int main() {
    Integer val{ 1 };
    Print(0, val, Integer{ 2 });
    return 0;
}
```