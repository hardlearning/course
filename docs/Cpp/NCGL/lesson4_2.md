# 2 多继承中的二义性内存分析

```cpp
#include <iostream>
using namespace std;

// 多继承二义性问题
class C {
public:
    int c1;
};

class B1: public C {
public:
    int b1;
};

class B2: public C {
public:
    int b2;
};

/*
 C  C
 B1 B2
   A3
 */
class A3: public B1, public B2 {
public:
    int a1 = 0;
};

// 虚继承
class B3: virtual public C {
public:
    int b1;
};

class B4: virtual public C {
public:
    int b2;
};

/*
   C
 B3 B4
   A4
 */
class A4: public B3, public B4 {
public:
    int a1 = 0;
};

int main() {
    {
        // 多继承二义性问题
        A3 a;
        // a.C::c1 = 1;
        // cout << "&a.C::c1 = " << (long long)&a.C::c1 << endl;
        cout << "&a.B1::c1 = " << (long long)&a.B1::c1 << endl;
        cout << "&a.B2::c1 = " << (long long)&a.B2::c1 << endl;

        A4 a4;
        a4.c1 = 1;
        cout << "&a4.C::c1 = " << (long long)&a4.C::c1 << endl;
        cout << "&a4.B3::c1 = " << (long long)&a4.B3::c1 << endl;
        cout << "&a4.B4::c1 = " << (long long)&a4.B4::c1 << endl;
    }

    return 0;
}
```