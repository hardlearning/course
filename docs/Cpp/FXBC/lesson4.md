# 4 Traits技术

## 4.1 内嵌数据类型表

```cpp
#include <iostream>
using namespace std;

// Traits: 利用typedef语法实现了类型萃取器的作用
typedef long double LDBL;
typedef unsigned int UINT32;

// 希望实现在类的外部形成数据类型的访问能力
// 模板名<模板实参>::类型名 变量名

template<typename T>
struct map {
    typedef T value_type;
    typedef T& reference;
    typedef T* pointer;
};

// 内嵌数据类型表实现：我们内嵌了数据类型=>从模板传入的数据类型衍生出了其对象相关类型的能力
int main() {
    map<int>::value_type a = 100.9;
    cout << "a=" << a << endl;
    map<double>::value_type b = 100.9;
    cout << "b=" << b << endl;
    return 0;
}
```

如果一个类模板中，全部的成员都是公有数据类型，这个模板就是一个独立的数据类型表，用来规范数据。

```cpp
#include <iostream>
using namespace std;

// 第一步：我们定义一个规范类模板类型表的基类
// 第二步：凡是继承了这个类型表的模板，它的访问类型就被确定了
template<typename T, typename U>
class TypeTb1 {
public:
    typedef T value_type1;
    typedef T& reference1;
    typedef U value_type2;
    typedef U& reference2;
};

// 我用一个类继承这个数据类型表
template<typename T, typename U>
class A: public TypeTb1<T,U> {

};

int main() {
    A<int, double>::value_type1 a = 200;
    A<int, double>::reference1 b = a;
    cout << a << endl;
    cout << b << endl;
    return 0;
}
```

## 4.2 “泛型”的内涵

```cpp
#include <iostream>
using namespace std;

template<class _A1, class _A2, class _R>
struct binary {
    typedef _A1 Arg1;
    typedef _A2 Arg2;
    typedef _R Rtn;
};

// 设计一个继承上面binary接口的类模板
template<class TR, class T1, class T2>
class Add: public binary<TR,T1,T2> {
public:
    TR bFunction(const T1& x, const T2& y) const {
        return x + y;
    }
};

int main() {
    // 因为Add包含了binary的数据类型表，因此系统中的其他模块就可以使用Add::Arg1、Add::Arg2、Add::Rtn这种方式和Add本身进行对接。
    // 通过这种数据类型的抽象，达到了多个系统模块之间的数据类型统一
    double a = 100.01, b = 20.2;
    Add<double, double, double> addObj;
    cout << addObj.bFunction(a, b) << endl;
    // 使用Arg1定义一个变量x
    typename Add<int, int, double>::Arg1 x = 1000;
    cout << x << endl;
    return 0;
}
```

## 4.3 非侵入式的STL类型设计

- 侵入式代码设计：

```cpp
#include <iostream>
using namespace std;

// 这两个类都只有一个compute函数，而且这两个函数的逻辑完全相同
// 不同的仅仅是函数参数类型
class Test1 {
public:
    char compute(int x, double y) {
        return x;
    }
};

class Test2 {
public:
    double compute(double x, double y) {
        return x;
    }
};

// Test的设计中，侵入了test1和test2的设计，没有很好地复用已经设计好的函数
template<typename Arg1, typename Arg2, typename Ret>
class Test {
public:
    Ret compute(Arg1 x, Arg2 y) {
        return x;
    }
};

int main() {
    Test<int, double, char> t1;
    cout << t1.compute(65, 6.18) << endl;
    return 0;
}
```

- 特化数据类型表

```cpp
#include <iostream>
using namespace std;

class Test1;
class Test2;

// 两个类模板规范一个统一的接口
template<typename T>
class TypeTb1 {

};

// 特化模板
template<>
class TypeTb1<Test1> {
public:
    typedef char ret_type;
    typedef int par1_type;
    typedef double par2_type;
};

template<>
class TypeTb1<Test2> {
public:
    typedef double ret_type;
    typedef double par1_type;
    typedef int par2_type;
};

// Test1和Test2的模板
template<typename T>
class Test {
public:
    typename TypeTb1<T>::ret_type compute(
        typename TypeTb1<T>::par1_type x,
        typename TypeTb1<T>::par2_type y
        ) {
        return x;
    }
};

int main() {
    Test<Test1> t1;
    cout << t1.compute(65, 6.18) << endl;
    return 0;
}
```

## 4.4 Traits技术原理

Traits可以实现类型萃取

泛型的特点是可以约定代码中的复杂数据类型的基本特征

指针天生不具备向外提供数据类型的能力，指针不是类，他就是一个变量，不能约束类型

```cpp
#include <iostream>
using namespace std;

// Iterator_1和Iterator_2代表不同迭代器，但是可以定义其他操作方法不同，这里省略了
template<typename T>
class Iterator_1 {
public:
    typedef T value_type;
    typedef value_type* pointer;
    typedef value_type& reference;
};

template<typename T>
class Iterator_2 {
public:
    typedef T value_type;
    typedef value_type* pointer;
    typedef value_type& reference;
};

template<typename T>
struct Traits {

};

template<typename T>
struct Traits<T*> {
    typedef T value_type;
    typedef value_type* pointer;
    typedef value_type& reference;
};

class A {
public:
    void show() {
        cout << "A::show()" << endl;
    }
};

int main() {
    Iterator_1<int>::value_type t1 = 100;
    cout << t1 << endl;
    Iterator_2<double>::value_type t2 = 6.18;
    cout << t2 << endl;
    Traits<double*>::value_type t3 = 4.44;
    cout << t3 << endl;

    // 通过Traits的数据类型，可以有效地规范类型型别统一，从而避免了类型转换问题
    // 这也就是泛型的基石
    Iterator_1<A>::pointer iter = new A();
    iter->show();
    return 0;
}
```