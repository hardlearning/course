# 1 指针入门

## 1.1 第一个指针程序

```cpp
#include <iostream>
using namespace std;

int main() {
    // 指针类型变量p1存在栈中
    // new int存在堆中
    int *p1 = new int;
    // *叫间接符号，表示间接访问int类型的值
    *p1 = 101;
    int i = 10;
    // &获取变量的地址
    int *p2 = &i;
    *p2 = 102;
    cout << "i = " << i << endl;
    cout << "p1 = " << p1 << endl;
    // 查看指针自己所占空间的大小（字节数）
    cout << "sizeof(p1) = " << sizeof(p1) << endl;
    // 查看指针指向空间的大小（字节数）
    cout << "sizeof(*p1) = " << sizeof(*p1) << endl;
    delete p1;
    cout << "after delete p1 = " << p1 << endl;
    // 将p1赋值为空指针后两次delete程序不会崩溃
    p1 = nullptr;
    if (!p1) {
        cout << "p1 is empty!" << endl;
    }
    delete p1;
    return 0;
}
```

## 1.2 程序的内存地址划分

- 保留区：0地址开始，c库
- 代码区：.text函数代码
- 全局变量区：.data非0值和.bss未初始化
- 堆区：new分配空间，delete释放空间
- 栈区：{}内定义的变量，出了括号自动清理
- 命令行参数环境变量
- 内核空间：高地址，一般windows占2G，Linux占1G

```cpp
#include <iostream>
using namespace std;

// 全局变量
int g1;
int g2 = 0;
int g3 = 1;
int g4 = 2;
static int sg1 = 3;

int main() {
    static int s1 = 4;
    // 代码区 .text
    cout << "代码区 main = " << main << endl;
    // g1和g2是连续分配的
    cout << "全局变量未初始化 g1 = " << &g1 << endl;
    cout << "全局变量初始化为0 g2 = " << &g2 << endl;
    // g3、g4、sg1、s1是连续分配的
    cout << "全局变量初始化为1 g3 = " << &g3 << endl;
    cout << "全局变量初始化为2 g4 = " << &g4 << endl;
    cout << "静态全局变量初始化为3 sg1 = " << &sg1 << endl;
    cout << "静态局部变量初始化为4 s1 = " << &s1 << endl;

    int *p1 = new int;
    int *p2 = new int;
    cout << "堆空间地址 p1 = " << p1 << endl;
    cout << "堆空间地址 p2 = " << p2 << endl;
    cout << "指针变量的栈空间地址 p1 = " << &p1 << endl;
    cout << "指针变量的栈空间地址 p2 = " << &p2 << endl;
    int i1 = 100;
    int i2 = 101;
    cout << "栈空间地址 i1 = " << &i1 << endl;
    cout << "栈空间地址 i2 = " << &i2 << endl;
    return 0;
}
```
