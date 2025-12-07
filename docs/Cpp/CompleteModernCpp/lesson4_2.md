# Template Argument Deduction & Instantiation

## Template Argument Deduction

- Process of deducing the types
- Each function argument is examined
- The corresponding type argument is deduced from the argument
- The type argument deduction should lead to same type
- Type conversions are not performed
- After deduction, the template is instantiated
- Override deduction by specifying types in template argument list: `Max<int>(3,5);`

## Template Instantiation

- A template function or class only acts as a blueprint
- The compiler generates code from the blueprint at compile time
- Known as template instantiation
- Occurs implicitly when
    - a function template is invoked
    - taking address of a function template
    - using explicit instantiation
    - creating explicit specialization
- Full definition of template should be available
- Define in header file

```cpp
#include <iostream>

template<typename T>
T Max(T x, T y) {
    std::cout << typeid(T).name() << std::endl;
    return x > y ? x : y;
}

// 显式实例化函数模板
template char Max(char, char);

int main() {
    Max(3, 5);
    Max(3.1, 6.2);
    Max(static_cast<float>(3), 5.5f);
    Max<double>(3, 6.2);
    // 获取函数模板的地址时，函数模板也会被实例化
    int (*pfn)(int, int) = Max;
    return 0;
}
```