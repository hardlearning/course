# 4 指针作为函数参数和返回值

## 4.1 指针作为参数设计说明

- 传递输入内存，需要提供内存大小（数组到指针无法知道大小），并设置为const
- 传递输出内存，同样需要提供内存大小，防止溢出
- 如果指向函数中创建堆内存，需要传递指针引用或者指向指针的指针
- 通过返回值只能返回堆中内存，比较危险，需要函数中注释说明，防止内存泄漏

### 4.1.1 指针作为输入参数

```cpp
#include <iostream>
using namespace std;

void PtrFunction(const char *data, int size) {
    // 这里返回的是指针的内存大小
    cout << "sizeof(data) = " << sizeof(data) << endl;
    cout << "size = " << size << endl;
}

// 经过参数传递，数组转成指针
void PtrFunction(const char data[]) {
    cout << "sizeof(data) = " << sizeof(data) << endl;
}

// 传递模板参数获取数组大小
template <class Ty, size_t Size>
void TestMemArr(Ty(&data)[Size]) {
    cout << "sizeof(data) = " << sizeof(data) << endl;
}

int main() {
    char data[] = "test mem ptr";
    PtrFunction(data, sizeof(data));
    PtrFunction(data);
    TestMemArr(data);

    return 0;
}
```

### 4.1.2 指针作为输出参数

```cpp
#include <iostream>
using namespace std;

// 指针输出
// 1 修改原内存
// 2 使用函数内创建的内存
char *TestMem() {
    // error 不能返回栈中的空间
    char buf[1024] = "test";
    return buf;
}

void TestMem(char *out) {
    // error 修改的是局部变量，内存泄露
    out = new char[1024];
}

// 返回的空间需要注释，由用户释放
int TestMem(char **out) {
    int size = 1024;
    *out = new char[size];
    return size;
}

int main() {
    // 异常：空间已经释放，结果不确定
    // cout << TestMem() << endl;

    char *out = nullptr;
    // 修改的是复制后的形参，所有out没变
    TestMem(out);
    cout << "out = " << (long long)out << endl;
    TestMem(&out);
    cout << "out = " << (long long)out << endl;
    delete out;
    out = nullptr;

    return 0;
}
```

## 4.2 智能指针作为函数参数和返回值

```cpp
#include <iostream>
#include <memory>
using namespace std;

class XData {
public:
    XData() { cout << "Create XData" << endl; }
    ~XData() { cout << "Drop XData" << endl; }
    int index = 0;
};

// 智能指针作为函数参数和返回值
unique_ptr<XData> TestUnique(unique_ptr<XData> ptr) {
    cout << "ptr->index = " << ptr->index << endl;
    unique_ptr<XData> re(new XData);
    re->index = 222;
    // 编译器优化：如果返回值支持move，编译器会自动调用move，不支持则调用拷贝构造
    return re;
}


int main() {
    {
        unique_ptr<XData> ptr1(new XData);
        ptr1->index = 111;
        cout << "ptr1->index = " << ptr1->index << endl;
        auto re = TestUnique(move(ptr1));
        cout << "re->index = " << re->index << endl;
        if (!ptr1) {
            cout << "ptr1 is empty" << endl;
        } else {
            cout << "ptr1->index = " << ptr1->index << endl;
        }
    }

    return 0;
}
```

