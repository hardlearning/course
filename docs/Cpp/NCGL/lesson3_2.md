# 2 分配器allocator

## 2.1 allocator用法

- 分配器用于实现容器算法时，将其与存储细节隔离从而解耦合
- 分配器提供存储分配与释放的标准方法
- 对象内存分配和构造分离

```cpp
#include <iostream>
using namespace std;

class XData {
public:
    XData() { cout << "Create XData" << endl; }
    ~XData() { cout << "Drop XData" << endl; }
};

int main() {
    /*
     * std::allocator的方法：
     * allocate 分配未初始化的存储
     * deallocate 解分配存储
     */
    allocator<XData> xdata_alloc;
    int size = 3;
    // 给对象分配空间，不调用构造函数
    auto dataarr = xdata_alloc.allocate(size);
    // 清理空间，不调用析构函数
    xdata_alloc.deallocate(dataarr, size);

    for (int i = 0; i < size; i++) {
        // allocator_traits类模板提供访问分配器allocator各种属性的标准方式
        // 调用构造函数
        // allocator_traits<allocator<XData>>::construct(xdata_alloc, &dataarr[i]);
        allocator_traits<decltype(xdata_alloc)>::construct(xdata_alloc, &dataarr[i]);
        // 调用析构函数
        allocator_traits<decltype(xdata_alloc)>::destroy(xdata_alloc, &dataarr[i]);
    }

    return 0;
}
```

## 2.2 自定义分配器

- 可以实现内存共享、内存泄漏探测，预分配对象存储、内存池
- 演示自定义vector和list分配器

```cpp
#include <iostream>
#include <vector>
#include <list>
using namespace std;

class XData {
public:
    XData() { cout << "Create XData" << endl; }
    XData(const XData& d) {
        this->index = d.index;
        cout << "Copy XData: " << index << endl;
    }
    ~XData() { cout << "Drop XData: " << index << endl; }
    int index = 0;
};

template<typename Ty>
class MyAllocator {
public:
    using value_type = Ty;

    MyAllocator() {}

    template<class Other>
    MyAllocator(const MyAllocator<Other>&) {}

    // 分配内存
    Ty* allocate(const size_t count) {
        cout << "allocate " << count << endl;
        cout << typeid(Ty).name() << endl;
        return static_cast<Ty*>(malloc(sizeof(Ty) * count));
    }

    // 内存释放
    void deallocate(Ty* const ptr, const size_t count) {
        cout << "deallocate " << count << endl;
        free(ptr);
    }
};

int main() {
    vector<XData, MyAllocator<XData>> vd;
    XData d;
    d.index = 111;
    vd.push_back(d);
    d.index = 222;
    vd.push_back(d);
    d.index = 333;
    vd.push_back(d);
    for (auto &xd: vd) {
        cout << xd.index << endl;
    }

    cout << "===========list==============" << endl;
    list<XData, MyAllocator<XData>> ld;
    d.index = 444;
    ld.push_back(d);
    d.index = 555;
    ld.push_back(d);
    d.index = 666;
    ld.push_back(d);
    for (auto &d: ld) {
        cout << d.index << endl;
    }

    return 0;
}
```



