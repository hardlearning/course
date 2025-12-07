# Class Templates

```cpp
#include <iostream>

template<typename T, int size>
class Stack {
    T m_Buffer[size];
    int m_Top{ -1 };
public:
    Stack() = default;
    Stack(const Stack& other) {
        m_Top = other.m_Top;
        for (int i = 0; i <= m_Top; ++i) {
            m_Buffer[i] = other.m_Buffer[i];
        }
    }
    void Push(T elem) {
        m_Buffer[++m_Top] = elem;
    }
    void Pop();
    const T& Top() const {
        return m_Buffer[m_Top];
    }
    bool IsEmpty() const {
        return m_Top == -1;
    }
    // 简写表示法：Stack
    // 完整表示法：Stack<T, size>
    static Stack Create();
};

template<typename T, int size>
void Stack<T, size>::Pop() {
    --m_Top;
}

// 简写表示法只在类内有效，在类外无效
template<typename T, int size>
Stack<T, size> Stack<T, size>::Create() {
    return Stack<T, size>();
}

int main() {
    Stack<float, 10> s = Stack<float, 10>::Create();
    s.Push(3);
    s.Push(1);
    s.Push(6);
    s.Push(9);
    auto s2(s);
    while (!s2.IsEmpty()) {
        std::cout << s2.Top() << " ";
        s2.Pop();
    }
    return 0;
}
```