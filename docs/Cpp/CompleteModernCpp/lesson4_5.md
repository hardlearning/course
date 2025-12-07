# Perfect Forwarding - I

- Integer.h

```cpp
#pragma once
#include <iostream>

class Integer {
    int *m_pInt;
public:
    // Default constructor
    Integer();
    // Parameterized constructor
    Integer(int value);
    // Copy constructor
    Integer(const Integer& other);
    // Move constructor
    Integer(Integer&& other);
    int GetValue() const;
    void SetValue(int value);
    ~Integer();
    friend std::ostream& operator<<(std::ostream& os, const Integer& obj) {
        os << obj.GetValue();
        return os;
    }
};
```

- Integer.cpp

```cpp
#include "Integer.h"

Integer::Integer() {
    std::cout << "Integer()" << std::endl;
    m_pInt = new int(0);
}

Integer::Integer(int value) {
    std::cout << "Integer(int)" << std::endl;
    m_pInt = new int(value);
}

Integer::Integer(const Integer &other) {
    std::cout << "Integer(const Integer&)" << std::endl;
    m_pInt = new int(*other.m_pInt);
}

Integer::Integer(Integer &&other) {
    std::cout << "Integer(Integer&&)" << std::endl;
    m_pInt = other.m_pInt;
    other.m_pInt = nullptr;
}

int Integer::GetValue() const {
    return *m_pInt;
}

void Integer::SetValue(int value) {
    *m_pInt = value;
}

Integer::~Integer() {
    std::cout << "~Integer()" << std::endl;
    delete m_pInt;
}
```

- main1.cpp

```cpp
#include <iostream>
#include "Integer.h"

class Employee {
    std::string m_Name;
    Integer m_Id;
public:
    Employee(const std::string &name, const Integer &id):
    m_Name(name), m_Id(id) {
        std::cout << "Employee(const std::string &name, const Integer &id)" << std::endl;
    }

    Employee(std::string &&name, Integer &&id):
    m_Name(name), m_Id(std::move(id)) {
        std::cout << "Employee(std::string &&name, Integer &&id)" << std::endl;
    }
};

int main() {
    Employee emp1{ "Umar", 100 };

    std::string name = "Umar";
    Employee emp2{ name, 100 };

    Integer val{ 100 };
    Employee emp3{ "Umar", val };
    return 0;
}
```

- main2.cpp

```cpp
#include <iostream>
#include "Integer.h"

class Employee {
    std::string m_Name;
    Integer m_Id;
public:
    template<typename T1, typename T2>
    Employee(T1 &&name, T2 &&id):
    m_Name(name), m_Id(std::move(id)) {
        std::cout << "Employee(std::string &&name, Integer &&id)" << std::endl;
    }
};

int main() {
    Employee emp1{ "Umar", 100 };

    Employee emp2{ "Umar", Integer{ 100 } };
    return 0;
}
```
