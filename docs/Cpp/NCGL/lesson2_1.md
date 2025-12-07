# 1 unique_ptr

```cpp
#include <iostream>
#include <memory>
using namespace std;

class Data {
public:
    static int count;
    int index;
    Data() {
        index = ++count;
        cout << "Create Data: " << index << endl;
    }
    ~Data() {
        cout << "Drop Data: " << index << endl;
    }
};
int Data::count = 0;

struct XPacket {
    unsigned char *data = nullptr;
    int size = 0;
};

class PacketDelete {
public:
    void Close() {
        cout << "Call PacketDelete Close" << endl;
    }
    // 智能指针出作用域时会调用这个操作符
    void operator()(XPacket *p) const {
        cout << "Call PacketDelete()" << endl;
        delete p->data;
        delete p;
    }
};

int main() {
    {
        // 1 unique_ptr初始化
        unique_ptr<int> p1(new int);
        unique_ptr<Data> p2(new Data);
        cout << "=====1=====" << endl;
        // make_unique底层也是new
        auto p3 = make_unique<Data>();
        // 空智能指针
        unique_ptr<Data> p4;
        // 数组
        unique_ptr<int[]> pa1 = make_unique<int[]>(1024);
        unique_ptr<int[]> pa2(new int[1024]);
        unique_ptr<Data[]> pa3(new Data[3]);

        // 2 unique_ptr智能指针访问和移动构造赋值
        *p1 = 10;
        cout << "p1 value = " << *p1 << endl;
        cout << "(*p2).index = " << (*p2).index << endl;
        cout << "p2->index = " << p2->index << endl;

        auto d1 = new Data();
        unique_ptr<Data> p5(d1);
        cout << "p5 address: " << p5 << endl;
        cout << "d1 address: " << d1 << endl;
        // get返回智能指针指向的空间地址
        cout << "p5.get(): " << p5.get() << endl;
        // 数组
        pa2[1] = 100;
        pa2[0] = 1;
        cout << "pa3[0].index = " << pa3[0].index << endl;
        cout << "pa3.get()[2].index = " << pa3.get()[2].index << endl;

        // 3 重置和移动内存资源
        // unique_ptr的拷贝构造和赋值操作符都是delete了
        unique_ptr<Data> p6(new Data);
        // 不可以拷贝构造
        // unique_ptr<Data> p7 = p6;
        // 不可以赋值
        // p7 = p6;
        // 支持移动构造：p6释放所有权，转移到p7
        unique_ptr<Data> p7 = move(p6);
        // 支持移动赋值
        unique_ptr<Data> p8(new Data);
        p7 = move(p8);
        // 重置空间，原空间释放
        p7.reset(new Data());

        // 4 释放所有权
        cout << "begin p7 = nullptr" << endl;
        // 主动释放空间
        p7 = nullptr;
        cout << "end p7 = nullptr" << endl;

        unique_ptr<Data> p9(new Data);
        cout << "begin p9.release()" << endl;
        // release释放所有权，返回的裸指针需要手动释放
        auto ptr9 = p9.release();
        cout << "end p9.release()" << endl;
        delete ptr9;

        // 5 自定义空间删除方法
        // 假设XPacket是第三方库
        unique_ptr<XPacket, PacketDelete> pd1(new XPacket);
        unique_ptr<XPacket, PacketDelete> pd2(new XPacket);
        pd2.get_deleter().Close();
        pd2.get_deleter()(pd2.get());
        pd2.release();
    }
    cout << "=====2=====" << endl;
    return 0;
}
```