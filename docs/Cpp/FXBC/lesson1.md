# 1 模板机制剖析

## 1.1 函数模板和模板函数的辨析

```cpp
#include <iostream>
using namespace std;

// 函数模板
template<typename T>
void myswap(T &a, T &b) {
    T tmp;
    tmp = a;
    a = b;
    b = tmp;
}

int main() {
    int a = 2, b = 3;
    cout << "a=" << a << ", b=" << b << endl;
    myswap(a, b);  // 编译器会换成模板函数 myswap<int>
    cout << "a=" << a << ", b=" << b << endl;

    double x = 10, y = 8;
    cout << "x=" << x << ", y=" << y << endl;
    myswap(x, y);  // 编译器会换成模板函数 myswap<double>
    cout << "x=" << x << ", y=" << y << endl;
    return 0;
}
```

## 1.2 函数模板与隐式类型转换

```cpp
#include <iostream>
using namespace std;

template<typename T>
void myswap(T &a, T &b) {
    cout << "template的函数被调用了" << endl;
}

void myswap(char &a, int &b) {
    cout << "普通myswap被调用了" << endl;
}

// 类型严格匹配是模板函数调用的先决条件
int main() {
    char p = 'a';
    int data = 23;
    // 函数模板不提供隐式类型转换，因此必须严格遵循T的类型定义
    // myswap<int>(p, data);
    myswap(p, data);
    return 0;
}
```

## 1.3 函数模板调用优先级

编译器会对函数模板进行二次编译：在声明的地方对模板代码本身进行编译，在调用的地方对参数化以后的具体调用进行编译。

```cpp
#include <iostream>
using namespace std;

template<typename T>
void myswap(T &a, T &b) {
    cout << "template的函数被调用了" << endl;
}

void myswap(int &a, int &b) {
    cout << "int 普通myswap被调用了" << endl;
}

// 当函数模板和普通函数都符合调用规则的时候，优先调用普通函数。
// 因为普通函数在编译的期间就生成了函数体，
// 而模板函数的生成需要在调用的时候，编译器才会编译。
int main() {
    int data1 = 1;
    int data2 = 2;
    myswap(data1, data2);
    myswap<>(data1, data2);
    return 0;
}
```

## 1.4 模板命名混淆

- caller1.cpp

```cpp
#include <iostream>

template<typename T>
void MyFunc(T const &v) {
    std::cout << "MyFunc Caller1" << std::endl;
}

void caller1() {
    MyFunc(1); // myfunc<int>
    MyFunc(0.1); // myfunc<double>
}
```

- caller2.cpp

```cpp
#include <iostream>

template<typename T>
void MyFunc(T const &v) {
    std::cout << "MyFunc Caller2" << std::endl;
}

void caller2() {
    MyFunc(2); // myfunc<int>
    MyFunc(0.2f); // myfunc<double>
}
```

- main.cpp

```cpp
#include <iostream>
using namespace std;

// 重复模板示例 => name mangling 命名混淆
// 命名混淆的解决方案是加命名空间
void caller1();
void caller2();

int main() {
    caller1();
    caller2();
    return 0;
}
```