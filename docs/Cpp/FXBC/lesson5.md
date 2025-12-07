# 5 仿函数与lambda表达式

## 5.1 仿函数

我们把重载了()运算符的类产生的对象，称为仿函数，也叫函数对象(Function Object)。

```cpp
vector<int> myVec;
// greater<int>()就是仿函数
sort(myVec.begin(), myVec.end(), greater<int>());
```

- 仿函数实现

```cpp
#include <iostream>
using namespace std;

// STL=算法+数据结构
class Add {
public:
    Add(int temp) : m_x(temp) {}
    int operator()(int a, int b) {
        return a + b + m_x;
    }
private:
    int m_x;
};

int main() {
    // 为什么不用函数指针？
    // 因为使用仿函数实际上是在调用一个对象，我们可以为这个函数附加额外的信息，
    // 从而可以充分利用C++内存对象布局中的代码绑定能力
    Add myAdd(100);
    cout << myAdd(1, 2) << endl;

    Add myAdd2(200);
    cout << myAdd(1, 2) << endl;
    return 0;
}
```

## 5.2 lambda表达式

```cpp
// lambda表达式是C++11的一种语法糖，让我们便捷地创建临时函数
[](int x) { cout << x * 2 << endl; } (10);

auto f = [](int x) { cout << x * 2 << endl; };
f(20);
```

## 5.3 智能指针

```cpp
#include <iostream>
using namespace std;

// RAII(Resource Acquisition is Initialization)
// 资源分配应当与对象生命周期绑定起来
// 创建对象的时候，我们就分配资源=>构造函数
// 销毁对象的时候，我们就回收资源=>析构函数
template<class T>
class MyPtr {
private:
    T* _ptr;
public:
    MyPtr(T* ptr): _ptr(ptr) {
        cout << "构造时分配资源" << endl;
    }
    ~MyPtr() {
        cout << "销毁时回收资源" << endl;
        delete _ptr;
    }

    T& operator*() {
        return *_ptr;
    }

    T* operator->() {
        return _ptr;
    }
};

int main() {
    int a = 10;
    int *pInt = new int(20);
    {
        // 错误：栈中的变量不需要delete
        // MyPtr<int> p1(&a);
        // cout << *p1 << endl;

        // 模拟堆中的变量通过智能指针释放
        MyPtr<int> p2(pInt);
        cout << *p2 << endl;
    }
    return 0;
}
```

- 可共享资源的智能指针

```cpp
#include <iostream>
using namespace std;

// 多个指针指向同一个资源，此时资源的释放不能取决于一个对象的生命周期
// 只要还有指针在持有资源，就不能释放
// 引用计数：有指针在引用，那么引用计数+1，指针变量生命周期结束，引用计数-1
template<typename T> class SharedPtr;
// 定义计数模板
template<typename T>
class ResPtr {
private:
    T* res_p; // 用以指向资源的裸指针
    int use_num; // 引用计数器
    ResPtr(T* p): res_p(p), use_num(1) {
        cout << "ResPtr构造函数" << endl;
    }
    ~ResPtr() {
        delete res_p;
        cout << "ResPtr析构函数" << endl;
    }
    friend class SharedPtr<T>;
};

template<typename T>
class SharedPtr {
public:
    SharedPtr(T *p, T i): ptr(new ResPtr<T>(p)), val(i) {
        cout << "SharedPtr构造函数 use_num=" << ptr->use_num << endl;
    }
    SharedPtr(const SharedPtr<T>& other): ptr(other.ptr), val(other.val) {
        ++ptr->use_num;
        cout << "SharedPtr拷贝构造函数 use_num=" << ptr->use_num << endl;
    }
    ~SharedPtr() {
        cout << "SharedPtr析构函数 use_num=" << ptr->use_num << endl;
        if (--ptr->use_num == 0) {
            delete ptr;
        }
    }
private:
    ResPtr<T>* ptr;  // 指向计数类ResPtr
    T val;
};

int main() {
    {
        SharedPtr<int> hpA = SharedPtr<int>(new int(42), 100);
        {
            SharedPtr<int> hpB(hpA);
            SharedPtr<int> hpC(hpB);
            SharedPtr<int> hpD = hpA;
        }
        cout << "内层括号结束" << endl;
    }
    cout << "外层括号结束" << endl;
    return 0;
}
```