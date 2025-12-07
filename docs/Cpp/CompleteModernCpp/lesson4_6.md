# Perfect Forwarding - II

函数模板规则：函数模板接受右值引用参数时，如果传入左值作为参数，那么参数类型将变为左值引用；如果传入右值作为参数，那么参数类型将变为右值引用。

```cpp
// 如果传入左值：T1 name => T1 &name, T2 id => T2 &id
// 如果传入右值：T1 name => T1 &&name, T2 id => T2 &&id
template<typename T1, typename T2>
Employee(T1 &&name, T2 &&id) {

}
```

main.cpp

```cpp
#include <iostream>
#include "Integer.h"

class Employee {
    std::string m_Name;
    Integer m_Id;

public:
    template<typename T1, typename T2>
    Employee(T1 &&name, T2 &&id) : m_Name(std::forward<T1>(name)), m_Id(std::forward<T2>(id)) {
        std::cout << "Employee(std::string &&name, Integer &&id)" << std::endl;
    }
};

template<typename T1, typename T2>
Employee *Create(T1 &&a, T2 &&b) {
    return new Employee(std::forward<T1>(a), std::forward<T2>(b));
}

int main() {
    Employee emp1{"Umar", Integer{100}};

    Integer val{100};
    Employee emp2{std::string{"Umar"}, val};

    auto emp = Create("Umar", Integer{100});
    return 0;
}
```