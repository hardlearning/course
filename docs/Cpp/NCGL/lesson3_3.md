# 3 未初始化内存算法uninitialized_copy

- 将范围内的对象复制到未初始化的内存区域
- memcpy或者std::copy不会调用拷贝构造，而uninitialized_copy会调用拷贝构造对对象进行复制

```cpp
#include <cstring>
#include <iostream>
#include <memory>
using namespace std;

class XData {
public:
    XData() { cout << "Create XData" << endl; }
    XData(const XData& d) {
        this->index = d.index;
        cout << "Copy XData: " << index << endl;
    }
    XData& operator=(const XData& d) {
        this->index = d.index;
        cout << "=XData: " << index << endl;
        return *this;
    }
    ~XData() { cout << "Drop XData: " << index << endl; }
    int index = 0;
};

int main() {
    XData datas[3];
    unsigned char buf[1024] = { 0 };
    cout << "===============memcpy================" << endl;
    // memcpy不会调用拷贝构造
    memcpy(buf, &datas, sizeof(datas));
    cout << "===============copy==================" << endl;
    // copy也不会调用拷贝构造，但调用赋值操作符
    std::copy(std::begin(datas), std::end(datas), (XData*)buf);
    cout << "=======uninitialized_copy============" << endl;
    // uninitialized_copy会调用拷贝构造
    uninitialized_copy(std::begin(datas), std::end(datas), (XData*)buf);

    return 0;
}
```