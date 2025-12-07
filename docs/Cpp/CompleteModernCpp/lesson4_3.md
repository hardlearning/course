# Explicit Specialization

- Template specialized for a particular type
- Provides correct semantics for some datatype
- Or implement an algorithm optimally for a specific type
- Explicitly specialized functions must be defined in a .cpp file
- Primary template definition should occur before specialization

```cpp
#include <cstring>
#include <iostream>

// Primary Template
template<typename T>
T Max(T x, T y) {
    std::cout << typeid(T).name() << std::endl;
    return x > y ? x : y;
}

// Explicit Instantiation
// template char Max(char, char);

// Explicit Specialization
template<> const char* Max(const char *x, const char *y) {
    std::cout << "Max<const char*>" << std::endl;
    return strcmp(x, y) > 0 ? x : y;
}

int main() {
    const char *b{ "B" };
    const char *a{ "A" };
    auto s = Max(a, b);
    std::cout << s << std::endl;
    return 0;
}
```