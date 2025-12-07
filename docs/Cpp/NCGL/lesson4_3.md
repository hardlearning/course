# 3 虚函数指针

## 3.1 虚函数表分析

```cpp
#include <iostream>
using namespace std;

// 虚函数表分析
class B {
public:
    virtual void Test1() {
        cout << "B test1" << endl;
    }
    virtual void Test2() {
        cout << "B test2" << endl;
    }
};

class A: public B {
public:
    void Test1() {
        cout << "A test1" << endl;
    }
    void Test2() {
        cout << "A test2" << endl;
    }
};

class C: public B {
public:
    void Test1() {
        cout << "C test1" << endl;
    }
    void Test2() {
        cout << "C test2" << endl;
    }
};

void TestClass(B* b) {
    cout << "in TestClass" << endl;
    b->Test1();
}

int main() {
    B b;
    b.Test1();
    A a;
    a.Test1();
    TestClass(&a);
    C c;
    TestClass(&c);

    return 0;
}
```

## 3.2 虚函数表指针直接访问函数

每个虚函数有一个指针，依次保存在虚函数表中。

```cpp
#include <iostream>
using namespace std;

class B {
public:
    virtual void Test1() {
        cout << "B test1" << endl;
    }
    virtual void Test2() {
        cout << "B test2" << endl;
    }
};

class A: public B {
public:
    void Test1() {
        cout << "A test1" << endl;
    }
    void Test2() {
        cout << "A test2" << endl;
    }
};

int main() {
    A a1;
    A a2;
    // 虚函数表的指针
    int *vftable1 = nullptr;
    vftable1 = (int*)(*((int*)&a1));
    int *vftable2 = (int*)(*((int*)&a2));
    // 同一个类的不同对象，虚函数表指向同一个
    cout << "a1 = " << (long long)&a1 << endl;
    cout << "vftable1 = " << (long long)vftable1 << endl;
    cout << "a2 = " << (long long)&a2 << endl;
    cout << "vftable2 = " << (long long)vftable2 << endl;

    // 取出虚函数表中的函数
    typedef void (*VFunc)();
    auto test1 = (VFunc)vftable1[0];
    auto test2 = (VFunc)vftable1[1];
    test1();
    test2();

    return 0;
}
```