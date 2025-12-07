# 5 使用string传递和获取内存

```cpp
#include <iostream>
#include <cstring>
#include <string>
using namespace std;

void TestString(string &in_data, string &out_data) {
    cout << "in_data = " << in_data << endl;
    cout << "in_data.size() = " << in_data.size() << endl;
    cout << "in_data addr: " << (long long)in_data.data() << endl;

    out_data.resize(1024);
    cout << "out_data addr: " << (long long)out_data.data() << endl;
}

int main() {
    // 使用string传递和获取内存
    // 不需要考虑内存释放
    // 可以获取已申请内存大小
    string str1;
    // resize开辟1024个字节的内存空间，默认值为0
    str1.resize(1024);
    auto data = const_cast<char*>(str1.data());
    // cout << "addr: " << (int)data << endl;  // 打印32位系统的地址
    cout << "addr: " << (long long)data << endl;  // 打印64位系统的地址
    memset(data, 'A', 5);
    cout << str1 << endl;

    string out_str;
    TestString(str1, out_str);
    cout << "out_str.size() = " << out_str.size() << endl;
    cout << "out_str addr: " << (long long)out_str.data() << endl;

    return 0;
}
```

