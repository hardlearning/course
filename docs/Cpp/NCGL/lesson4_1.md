# 1 限制栈中创建对象，限制调用delete销毁对象

```cpp
#include <iostream>
using namespace std;

class TestMem {
public:
    static TestMem* Create() {
        cout << "static Create" << endl;
        return new TestMem();
    }

    static void Drop(TestMem* tm) {
        cout << "static Drop" << endl;
        delete tm;
    }
protected:
    // 限制栈中直接创建对象
    TestMem() { cout << "Create TestMem" << endl; }
    // 限制调用delete销毁对象
    virtual ~TestMem() { cout << "Drop TestMem" << endl; }
};

int main() {
    // TestMem tm1;
    // auto tm2 = new TestMem();
    TestMem *tm3 = TestMem::Create();
    // delete tm3;
    TestMem::Drop(tm3);

    return 0;
}
```

