# Typedefs, Type Aliases & Alias Templates

## Type Definition (typedef)

- Introduces a name for an existing type
- This name becomes a synonym of that type
- Useful to construct shorter or more meaningful names for existing types
- Simplifies declaration of some types (e.g. function pointers &templates)
- Implementation details that may change can be encapsulated
- Does not introduce a new type; only introduces a new name for existing type

```cpp
typedef unsigned int UNINT;
UNINT val{};

typedef long long LLONG;
LLONG elem{};

typedef std::vector<std::list<Employee>> Teams;
Teams testingTeams;
Teams::iterrator it = testingTeams.begin();

typedef const char *(*ErrorFn)(int);
ErrorFn pfn = GetErrorMessage;
```

## Type Alias

- Creates a name that is a synonym of existing type
- Does not introduce a new type
- Same as a typedef declaration
- Created through the using keyword

```cpp
using UNINT = unsigned int;
UNINT val{};

using LLONG = long long;
LLONG elem{};

using Teams = std::vector<std::list<Employee>>;
Teams testingTeams;
Teams::iterrator it = testingTeams.begin();

using ErrorFn = const char *(*)(int);
ErrorFn pfn = GetErrorMessage;
```

## Alias Templates

```cpp
#include <iostream>
#include <vector>
#include <list>

// 类型定义
// typedef std::vector<std::list<std::string>> Names;

// 别名模板
template<typename T>
using Names = std::vector<std::list<T>>;

int main() {
    Names<std::string> names;
    Names<Names<std::string>> nnames;
    return 0;
}
```