# 4 常量指针和指针常量

```cpp
#include <iostream>
using namespace std;

int main() {
    // 常量指针和指针常量
    const int i1 = 100;
    // 常量指针：指向常量的指针，指向的值不能修改
    const int *pi1 = &i1;
    // (*pi1)++;
    int const *pi2 = &i1; // 同上
    // (*pi2)++;

    // 指针常量：指针本身是常量，指向不能修改，指向的值可以修改
    int *const pi3 = new int;
    // pi3++;
    // pi3 = new int;
    *pi3 = 200;
    delete pi3;

    // 指向和值都不能修改
    const int *const pi4 = &i1;
    // pi4 = new int;
    // *pi4 = 300;

    return 0;
}
```