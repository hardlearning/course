# Class Templates Explicit Specialization - II

- 为vector类型实现特化版本

```cpp
#include <iostream>
#include <vector>

template<typename T>
class PrettyPrinter {
    T *m_pData;
public:
    PrettyPrinter(T *data) : m_pData(data) {

    }
    void Print() {
        std::cout << "{" << *m_pData << "}" << std::endl;
    }
    T* GetData() {
        return m_pData;
    }
};

// 为vector类型实现特化版本
template<>
class PrettyPrinter<std::vector<int>> {
    std::vector<int> *m_pData;
public:
    PrettyPrinter(std::vector<int> *data) : m_pData(data) {

    }
    void Print() {
        std::cout << "{";
        for (const auto &x : *m_pData) {
            std::cout << x;
        }
        std::cout << "}" << std::endl;
    }
    std::vector<int>* GetData() {
        return m_pData;
    }
};

int main() {
    std::vector<int> v{1,2,3,4,5};
    PrettyPrinter<std::vector<int>> pv(&v);
    pv.Print();
    return 0;
}
```

- 在类外实现vector打印的特化方法

```cpp
#include <iostream>
#include <vector>

template<typename T>
class PrettyPrinter {
    T *m_pData;
public:
    PrettyPrinter(T *data) : m_pData(data) {

    }
    void Print() {
        std::cout << "{" << *m_pData << "}" << std::endl;
    }
    T* GetData() {
        return m_pData;
    }
};

template<>
void PrettyPrinter<std::vector<int>>::Print() {
    std::cout << "{";
    for (const auto &x : *m_pData) {
        std::cout << x;
    }
    std::cout << "}" << std::endl;
}

int main() {
    std::vector<int> v{1,2,3,4,5};
    PrettyPrinter<std::vector<int>> pv(&v);
    pv.Print();
    return 0;
}
```