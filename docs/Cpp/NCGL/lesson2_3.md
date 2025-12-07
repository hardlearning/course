# 3 weak_ptr

## 3.1 shared_ptr循环引用问题

```cpp
#include <iostream>
#include <memory>
using namespace std;

class B;

class A {
public:
    A() { cout << "Create A" << endl; }
    ~A() { cout << "Drop A" << endl; }
    shared_ptr<B> b1;
};

class B {
public:
    B() { cout << "Create B" << endl; }
    ~B() { cout << "Drop B" << endl; }
    shared_ptr<A> a1;
};

int main() {
    // 5 shared_ptr循环引用问题原理
    {
        auto a = make_shared<A>();  // =1
        auto b = make_shared<B>();  // =1
        a->b1 = b; // +1 =2
        cout << "a->b1 = b; b.use_count() = " << b.use_count() << endl;
        b->a1 = a; // +1 =2
        cout << "b->a1 = a; a.use_count() = " << a.use_count() << endl;
    }
    // a出作用域 a.use_count()-1=1; b.use_count()=2;
    // b出作用域 b.use_count()-1=1; a.use_count()=1;
    cout << "after AB }" << endl;

    return 0;
}
```

## 3.2 weak_ptr用法

- use_count: 返回管理该对象的shared_ptr对象数量
- lock: 创建管理被引用对象的shared_ptr

```cpp
#include <iostream>
#include <memory>
using namespace std;

class B;

class A {
public:
    A() { cout << "Create A" << endl; }
    ~A() { cout << "Drop A" << endl; }
    void Do() {
        cout << "Do b2.use_count() = " << b2.use_count() << endl;
        // 复制一个shared_ptr，引用计数+1
        auto b = b2.lock();
        cout << "after lock Do b2.use_count() = " << b2.use_count() << endl;
    }
    weak_ptr<B> b2;
};

class B {
public:
    B() { cout << "Create B" << endl; }
    ~B() { cout << "Drop B" << endl; }
    shared_ptr<A> a1;
};

int main() {
    {
        auto a = make_shared<A>();  // =1
        auto b = make_shared<B>();  // =1
        a->b2 = b; // =1
        a->Do();
        cout << "a->b2 = b; b.use_count() = " << b.use_count() << endl;
        b->a1 = a; // +1 =2
        cout << "b->a1 = a; a.use_count() = " << a.use_count() << endl;
    }

    return 0;
}
```
