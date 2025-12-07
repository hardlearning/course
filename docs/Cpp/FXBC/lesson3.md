# 3 右值引用与模板

## 3.1 左值和右值的概念

- 左值：一般出现在赋值运算符左边，往往代表一个存储空间。因此左值是一个有名字，有固定地址的表达式。
- 右值：表示所谓的数据，不能被赋值，是一个和运算过程相匹配的临时对象，这个临时对象在所对应的语句执行完毕后就销毁了，所有我们无法从语法层面上直接访问。因此右值是匿名的，没有固定地址的对象。

```cpp
int a = 10;  // a是左值，10是右值
int b = 20;
int c = a + b;
// int a + b = 20; // a+b是右值，不能被赋值
```

汇编代码：

```
__asm{
    mov eax,a
    mov ebx,b
    add eax,ebx
    mov c,eax
}
```

- 右值引用：通过&&表示的语法就叫做右值引用，使得右值变成一个和左值完全相同的持久对象。

```cpp
int a = 10;
int &b = a;
// 为左值a命名一个常量的别名c
const int &c = a;

int x = 1000, y = 100;
// 为右值x+y命名一个常量的别名i
const int &i = x + y;

int tt = x + y;
// 左值可以修改
tt++;
// i不能修改
// i++;

int &&right = x + y;
// 右值引用可以修改
right++;
printf("%d", right);
```

## 3.2 右值引用与转移语义

- 浅拷贝会造成同一块内存地址释放两次
- 深拷贝可以新申请一块内存，但是临时对象没有释放掉

```cpp
#include <iostream>
using namespace std;

class Foo {
public:
    Foo(int x) {
        p = new int(x);
    }
    // 问题：浅拷贝p和r.p会释放两次
    Foo(const Foo& r) {
        this->p = r.p;  // 浅拷贝=>p和r.p指向了同一块内存地址
    }
    ~Foo() {
        if (p != NULL) {
            delete p;
        }
    }
    void show_p() {
        cout << p << " " << *p << endl;
    }
private:
    int *p;
};

int main() {
    Foo f1(50);
    Foo f2(f1);
    f1.show_p();
    f2.show_p();
    return 0;
}
```

- 右值引用拷贝构造函数

```cpp
#include <iostream>
using namespace std;

class Foo {
public:
    Foo(int x) {
        p = new int(x);
    }
    // 因为r是一个临时对象，那么使用浅拷贝是有巨大意义的
    // 当我们需要具有“转移语义”的拷贝构造函数时，浅拷贝的效率意义就凸显了
    Foo(const Foo& r) {
        // this->p = r.p;  // 浅拷贝=>p和r.p指向了同一块内存地址
        // 深拷贝
        this->p = new int;
        *p = *(r.p);
    }
    // 右值引用拷贝构造函数
    Foo(Foo&& r) {
        cout << "Foo(Foo&&)" << endl;
        this->p = r.p; // step1:p和r.p指向了同一块内存地址
        r.p = nullptr; // step2:r.p设为空，源对象释放了资源所有权
    }
    ~Foo() {
        if (p != NULL) {
            delete p;
        }
    }
    void show_p() {
        cout << p << " " << *p << endl;
    }
private:
    int *p;
};

Foo func() {
    Foo foo(100);
    return foo;
}

int main() {
    Foo f1(50);
    Foo f2(f1);
    f1.show_p();
    f2.show_p();
    cout << "==========================" << endl;

    // 使用右值引用拷贝构造，资源所有权发生转移，资源位置没有变化而所属对象变化了
    Foo f(func());
    f.show_p();
    return 0;
}
```

## 3.3 参数完美转发模板

完美转发：(1)转发的参数不能改变其特性；(2)不要产生额外的开销

模板参数类型推导的规则：左值推导为T&，右值推导为T

`T&& move(T& val)`: 接收一个参数val，然后返回这个参数的右值引用。

`forward`: T& => T&&

- 问题代码一：

```cpp
#include <iostream>
using namespace std;

// 问题：值传递一定会进行临时对象的创建并且对数据进行拷贝，产生额外开销
void Myfunc(int v) {
    cout << "Myfunc" << endl;
}

// 转发模板
template<typename T>
void Tmp(T a) {
    Myfunc(a);
}

int main() {
    Tmp(10);  // 10是右值传递
    int x = 1;
    Tmp(x);     // x是左值传递
    return 0;
}
```

- 问题代码二：

```cpp
#include <iostream>
using namespace std;

void RightFunc(int &v) {
    cout << "&函数调用" << endl;
}

void RightFunc(int &&v) {
    cout << "&&函数调用" << endl;
}

// 转发模板
template<typename T>
void Tmp(T &&a) {
    RightFunc(a);
}

int main() {
    // 问题：全部调用RightFunc(int &v)
    int x = 1;
    int &y = x;
    Tmp(x); // 实参为左值
    Tmp(y); // 实参为左值引用
    Tmp(100); // 实参为右值
    return 0;
}
```

- 完美转发

```cpp
#include <iostream>
using namespace std;

// 参数类型为左值引用的目标函数
void Func(int &x) {
    cout << "左值引用" << endl;
}

// 参数类型为右值引用的目标函数
void Func(int &&x) {
    cout << "右值引用" << endl;
}

// 参数类型为左值常引用
void Func(const int &x) {
    cout << "左值常引用" << endl;
}

// 参数类型为右值常引用
void Func(const int &&x) {
    cout << "右值常引用" << endl;
}

// 参数完美转发
template<typename T>
void FuncT(T &&a) {
    Func(std::forward<T>(a));
}

int main() {
    FuncT(10);  // 形参类型：右值
    int a;
    FuncT(a);  // 形参类型：左值
    FuncT(std::move(a));  // 形参类型：右值
    const int b = 8;
    FuncT(b);  // 形参类型：左值
    FuncT(std::move(b));  // 形参类型：右值
    // 总结：完美转发解决了临时对象的效率问题
    return 0;
}
```