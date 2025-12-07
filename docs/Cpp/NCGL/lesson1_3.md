# 3 void指针和指针类型转换

```cpp
#include <iostream>
using namespace std;

int main() {
    // void指针和指针类型转换
    void *ptr1 = malloc(1024);
    free(ptr1);

    int num = 1;
    void *ptr = &num;
    // c++中的转换
    int *ptr2 = static_cast<int *>(ptr); // void*
    // c语言中的转换
    int *ptr3 = (int *) ptr;
    // static_cast增加了一些静态验证
    const int *cptr1 = new int[1024];
    // static_cast转换时不能去掉const
    // int *ptr4 = static_cast<int*>(cptr1);
    int *ptr4 = (int *) cptr1;
    int *ptr6 = const_cast<int *>(cptr1);  // 去掉const
    delete[] cptr1;

    unsigned char *ucptr = new unsigned char[1024];
    // int *ptr5 = static_cast<int*>(ucptr);
    auto ptr5 = (int *) ucptr;
    // 重新定义存储空间的使用
    auto ptr7 = reinterpret_cast<int *>(ucptr); // 替换指针类型
    delete[] ucptr;

    return 0;
}
```