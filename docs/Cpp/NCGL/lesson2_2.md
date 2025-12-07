# 2 shared_ptr

```cpp
#include <iostream>
#include <memory>
using namespace std;

class XData {
public:
    XData() {
        cout << "Create XData" << endl;
    }
    ~XData() {
        cout << "Drop XData" << endl;
    }
    int index1 = 0;
    int index2 = 0;
};

void DelData(XData* p) {
    cout << "Call DelData" << endl;
    delete p;
}

int main() {
    {
        // 1 shared_ptr的使用
        shared_ptr<int> sp1(new int);
        shared_ptr<int[]> sp2(new int[1024]);
        *sp1 = 10;
        sp2[0] = 100;
        auto d1 = sp1.get();
        auto sp3 = make_shared<XData>();
        cout << "sp3.use_count() = " << sp3.use_count() << endl;

        // 2 可以拷贝构造和赋值操作
        auto sp4 = sp3;
        cout << "sp4 = sp3; sp3.use_count() = " << sp3.use_count() << endl;
        cout << "sp4.use_count() = " << sp4.use_count() << endl;
        auto sp5 = make_shared<XData>();
        sp4 = sp5;
        cout << "sp4 = sp5; sp3.use_count() = " << sp3.use_count() << endl;
        {
            auto sp6 = sp3;
            cout << "sp6 = sp3; sp3.use_count() = " << sp3.use_count() << endl;
        }
        cout << "} sp3.use_count() = " << sp3.use_count() << endl;
        // 引用计数减1，如果为0则清理空间
        sp3.reset();
        cout << "sp3.reset(); sp3.use_count() = " << sp3.use_count() << endl;
    }
    cout << "after }" << endl;

    // 3 shared_ptr定制删除函数
    {
        shared_ptr<XData> sp7(new XData, DelData);
        shared_ptr<XData> sp8(new XData, [](auto p) {
            cout << "Call delete lambda" << endl;
            delete p;
        });
    }
    cout << "after delete" << endl;

    // 4 shared_ptr智能指针指向同一个对象不同成员
    {
        shared_ptr<XData> sc1(new XData);
        cout << "sc1.use_count() = " << sc1.use_count() << endl;
        shared_ptr<int> sc2(sc1, &sc1->index2);
        shared_ptr<int> sc3(sc1, &sc1->index1);
        cout << "sc1.use_count() = " << sc1.use_count() << endl;
    }

    return 0;
}
```