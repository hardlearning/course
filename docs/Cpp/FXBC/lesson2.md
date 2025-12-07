# 2 类模板

## 2.1 类模板的概念与应用

```cpp
#include <iostream>
using namespace std;

// STL中通常会定义两种数据类型：有两个占位符
// class和typename在指定模板参数时是等价的
template<class T1, class T2>
class MyClass {
public:
    T1 t11;
    T2 t22;
    MyClass(T1 t, T2 tt) : t11(t), t22(tt) {}

    void print() {
        cout << t11 << "," << t22 << endl;
    }
};

int main() {
    MyClass<int, double> m(10, 22.2);
    m.print();
    MyClass<double, std::string> m2(20.8, "helloTemplate");
    m2.print();
    return 0;
}
```

## 2.2 STL中Array容器的实现

- MyArr.hpp

```cpp
#pragma once

template<class T, int n>
class Array {
public:
    Array();
    Array(int length);
    ~Array();
    int size();
    T get(int num);
    T& operator[](int num);
    void set(int num, T data);
public:
    T *pt;
};

// n是定长的，所有数组一旦生成，不能变化
template<class T, int n>
Array<T, n>::Array() {
    this->pt = new T[n];
}

template<class T, int n>
Array<T, n>::Array(int length) {
    this->pt = new T[length];
}

template<class T, int n>
Array<T, n>::~Array() {
    delete[] this->pt;
}

template<class T, int n>
int Array<T, n>::size() {
    return n;
}

template<class T, int n>
T Array<T, n>::get(int num) {
    if (num >= n || num < 0) {
        // 异常
    } else {
        return *(this->pt + num);
    }
}

template<class T, int n>
void Array<T, n>::set(int num, T data) {
    if (num >= n || num < 0) {
        // 异常
    } else {
        *(this->pt + num) = data;
    }
}

template<class T, int n>
T& Array<T, n>::operator[](int num) {
    if (num >= n || num < 0) {
        // 异常
    } else {
        return *(this->pt + num);
    }
}
```

- main.cpp

```cpp
#include <iostream>
#include "MyArr.hpp"
using namespace std;

int main() {
    Array<int, 5> myIntArr;
    for (int i = 0; i < myIntArr.size(); i++) {
        myIntArr.set(i, i * 10);
        cout << myIntArr[i] << endl;
    }
    return 0;
}
```

## 2.3 类模板继承类模板

```cpp
#include <iostream>
using namespace std;

template<class T>
class MyClass {
public:
    T x;
    MyClass(T t): x(t) {}
    virtual void print() = 0;
};

template<class T>
class NewClass: public MyClass<T> {
public:
    T y;
    NewClass(T t1, T t2): MyClass<T>(t1), y(t2) {}
    void print() {
        cout << "x=" << this->x << ", y=" << this->y << endl;
    }
};

int main() {
    MyClass<int> *p = new NewClass<int>(10, 8);
    p->print();
    return 0;
}
```

## 2.4 模板类与普通类的继承

```cpp
#include <iostream>
using namespace std;

class XYZ {
public:
    int x, y, z;
    XYZ(int a, int b, int c): x(a), y(b), z(c) {}
    void print() {
        cout << x << ' ' << y << ' ' << z << endl;
    }
};

// 模板类继承普通类
template <class T>
class GoodsXYZ: public XYZ {
public:
    T t;
    GoodsXYZ(T t1, int a, int b, int c): XYZ(a, b, c), t(t1) {}
    void print() {
        cout << "T类型的值: " << t << endl;
        cout << x << ' ' << y << ' ' << z << endl;
    }
};

// 普通类继承模板类
class RunClass: public GoodsXYZ<int> {
public:
    int dd = 1000;
    RunClass(int a2, int b2, int c2, int d2): GoodsXYZ<int>(a2, b2, c2, d2) {}
    void print() {
        cout << dd << ' ' << x << ' ' << y << ' ' << z << ' ' << t << endl;
    }
};

int main() {
    RunClass run1(1, 2, 3, 4);
    run1.print();

    string str1 = "hello template";
    GoodsXYZ<string> zuobiao(str1, 1024, 768, 300);
    zuobiao.print();
    return 0;
}
```

## 2.5 嵌套模板类

```cpp
#include <iostream>
using namespace std;

template<class T>
class MyTestNestClass {
public:
    // 嵌套普通类
    class nClass {
    public:
        int num;
    } newObj;
    nClass newObj2;

    // 嵌套类模板
    template<class V>
    class RunClass {
    public:
        V v1;
    }; // 嵌套类模板，不能直接初始化
    RunClass<T> t1;
    RunClass<double> t2;
};

int main() {
    MyTestNestClass<int> myObj1;
    myObj1.newObj.num = 10;
    myObj1.t1.v1 = 13;
    myObj1.t2.v1 = 6.18;
    cout << myObj1.t1.v1 << endl;
    cout << myObj1.t2.v1 << endl;
    return 0;
}
```