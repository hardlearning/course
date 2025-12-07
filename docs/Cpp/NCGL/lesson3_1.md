# 1 重载new和delete

## 1.1 为什么要重载

- 监测内存创建销毁，统计和监控泄露
- 内存对齐的处理
- 特点应用：例如多进程内存共享

```cpp
#include <iostream>
using namespace std;

// 1 重载全局的new和delete
void *operator new(size_t size) {
    cout << "operator new: " << size << endl;
    auto mem = malloc(size);
    if (!mem) {
        throw new std::bad_alloc();
    }
    return mem;
}

void *operator new[](size_t size) {
    cout << "operator new[]: " << size << endl;
    auto mem = malloc(size);
    if (!mem) {
        throw new std::bad_alloc();
    }
    return mem;
}

void operator delete(void *ptr) {
    cout << "operator delete" << endl;
    std::free(ptr);
}

void operator delete[](void *ptr) {
    cout << "operator delete[]" << endl;
    std::free(ptr);
}

class TestMem {
public:
    TestMem() { cout << "Create TestMem" << endl; }
    ~TestMem() { cout << "Drop TestMem" << endl; }
    int index = 0;
    // 2 重载类的new和delete
    void *operator new(size_t size) {
        cout << "TestMem new: " << size << endl;
        auto mem = malloc(size);
        if (!mem) {
            throw new std::bad_alloc();
        }
        return mem;
    }

    void *operator new[](size_t size) {
        cout << "TestMem new[]: " << size << endl;
        auto mem = malloc(size);
        if (!mem) {
            throw new std::bad_alloc();
        }
        return mem;
    }

    void operator delete(void *ptr) {
        cout << "TestMem delete" << endl;
        std::free(ptr);
    }

    void operator delete[](void *ptr) {
        cout << "TestMem delete[]" << endl;
        std::free(ptr);
    }
};

int main() {
    int *i1 = new int;
    delete i1;

    auto mem1 = new TestMem();
    delete mem1;

    int *arr1 = new int[1024];
    delete[] arr1;

    TestMem *memarr1 = new TestMem[2];
    delete[] memarr1;
    return 0;
}
```

## 1.2 placement new重载

放置placement new和delete，new的空间指向已有的地址

- 普通new申请空间是从堆中分配空间，一些特殊的需求要在已分配的空间中创建对象，就可以用放置操作
- placement new生成对象既可以在栈上，也可以在堆上
- placement new生成对象销毁要手动调用析构函数

```cpp
#include <iostream>
using namespace std;

class TestMem {
public:
    TestMem() { cout << "Create TestMem" << endl; }
    ~TestMem() { cout << "Drop TestMem" << endl; }
    int index = 0;
    // 重载类placement new
    void *operator new(size_t size, void *ptr) {
        cout << "TestMem placement new: " << size << endl;
        return ptr;
    }
};

int main() {
    int buf1[1024] = { 0 };
    // 栈中调用
    TestMem *mem2 = ::new(buf1) TestMem;
    cout << "buf1 addr: " << buf1 << endl;
    cout << "mem2 addr: " << mem2 << endl;
    mem2->~TestMem();

    // 堆中调用
    int *buf2 = new int[1024]{ 0 };
    auto mem3 = ::new(buf2) TestMem;
    delete mem3;

    int *buf3 = new int[1024]{ 0 };
    auto mem4 = new(buf3) TestMem;
    cout << "buf3 addr: " << buf3 << endl;
    cout << "mem4 addr: " << mem4 << endl;
    mem4->~TestMem();
    delete[] buf3;

    return 0;
}
```
