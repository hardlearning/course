# 4 construct_at对象构造和destroy对象销毁

- C++17 destroy对象销毁
- C++20 construct_at对象构造

```cpp
#include <cstring>
#include <iostream>
using namespace std;

class XData {
public:
    XData() { cout << "Create XData" << endl; }
    ~XData() { cout << "Drop XData" << endl; }
};

int main() {
    {
        int size = 3;
        auto data = static_cast<XData*>(malloc(sizeof(XData) * size));
        for (int i = 0; i < size; i++) {
            if (data) {
                // 调用构造函数
                construct_at(&data[i]);
            }
        }
        // 调用析构函数
        destroy(data, data + size);
        free(data);
    }

    return 0;
}
```