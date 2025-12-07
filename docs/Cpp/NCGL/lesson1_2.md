# 2 数组的堆栈空间分配

## 2.1 数组的堆栈空间初始化和c++11的for遍历

```cpp
#include <cstring>
#include <iostream>
using namespace std;

class XData {
public:
    int x;
    int y;
};

int main() {
    // (1)栈空间
    int arr[10];  // 未初始化的数组
    // sizeof可以获取数组需要的内存大小
    memset(arr, 0, sizeof(arr));
    int arr2[5] = { 1, 2, 3, 4, 5 };
    // 设置部分值，其他为0
    int arr3[32] = { 1, 2, 3 };
    // 全部设置为0
    int arr4[1024] = { 0 };
    // 根据设置的值自动推导数组大小
    int arr5[] = { 1, 2, 3, 4 };
    char str1[] = "test001";  // 字符串包含结尾的\0
    cout << "sizeof(arr5) = " << sizeof(arr5) << endl;
    cout << "sizeof(str1) = " << sizeof(str1) << endl;
    // 栈空间可以for循环访问数组
    for (auto s : str1) {
        cout << s << "-" << flush;
    }
    cout << endl;

    // 查看数组的地址
    cout << "&arr2[0] = " << &arr2[0] << endl;
    cout << "&arr2[1] = " << &arr2[1] << endl;

    // (2)堆空间
    int *parr1 = new int[1024];
    int psize = 2048;
    auto parr2 = new unsigned char[psize];
    memset(parr2, 0, psize);
    auto parr3 = new int[psize];
    memset(parr3, 0, psize * sizeof(int));
    // 堆空间不能for循环访问数组，因为堆上的数组无法确定begin和end指针
    // for (auto s : parr2) {}

    int *parr4 = new int[3]{ 1, 2, 3 };
    int *parr5 = new int[]{ 1, 2, 3, 4, 5 };
    int *parr6 = new int[]{ 0};

    // 下标访问
    cout << "parr5[2] = " << parr5[2] << endl;
    cout << "*(parr5 + 3) = " << *(parr5 + 3) << endl;

    // 查看数组的地址
    cout << "&parr5[0] = " << &parr5[0] << endl;
    cout << "&parr5[1] = " << &parr5[1] << endl;
    cout << "&parr5 + 2 = " << parr5 + 2 << endl;

    // 释放堆空间
    delete[] parr1;
    parr1 = nullptr;
    delete[] parr2;
    parr2 = nullptr;
    delete[] parr3;
    parr3 = nullptr;
    delete[] parr4;
    parr4 = nullptr;
    delete[] parr5;
    parr5 = nullptr;
    delete[] parr6;
    parr6 = nullptr;

    XData *datas = new XData[1024];
    delete[] datas;
    datas = nullptr;
    return 0;
}
```

## 2.2 栈中二维数组的初始化和遍历

```cpp
#include <iostream>
using namespace std;

#define ARRSIZE 3
int main() {
    // 栈中二维数组的初始化
    unsigned char arr1[2][ARRSIZE] = {{1, 2, 3}, {4, 5, 6}};
    unsigned char arr2[][ARRSIZE] = {{1, 2, 3}, {2, 3, 4}, {3, 4, 5}};
    cout << "sizeof(arr1[2][3]) = " << sizeof(arr1) << endl;
    cout << "sizeof(arr2[][3]) = " << sizeof(arr2) << endl;

    // 三维数组
    int arr3[2][3][4] = {
        {{1, 2, 3, 4}, {2, 3, 4, 5}, {1, 2, 3, 4}},
        {{1, 2, 3, 4}, {2, 3, 4, 5}, {1, 2, 3, 4}}
    };
    int arr4[][3][4] = {
        {{1, 2, 3, 4}, {2, 3, 4, 5}, {1, 2, 3, 4}},
        {{1, 2, 3, 4}, {2, 3, 4, 5}, {1, 2, 3, 4}},
        {{1, 2, 3, 4}, {2, 3, 4, 5}, {1, 2, 3, 4}}
    };

    // 二维数组遍历
    for (auto arr : arr2) {
        for (int i = 0; i < 3; i++) {
            cout << static_cast<int>(arr[i]) << ' ' << flush;
        }
        cout << endl;
    }
    // 二维数组的维度
    int width = ARRSIZE;
    int height = sizeof(arr2) / ARRSIZE * sizeof(unsigned char);
    for (int i = 0; i < height; i++) {
        for (int j = 0; j < width; j++) {
            arr2[i][j]++;
            cout << (int) arr2[i][j] << '-' << flush;
        }
        cout << endl;
    }
    return 0;
}
```

## 2.3 堆中二维数组的初始化和遍历

```cpp
#include <iostream>
using namespace std;

int main() {
    // 堆中二维数组的两种方式
    // 1 连续空间
    int size = 2;
    int(*arr5)[3] = new int[size][3]{{1,1,2},{3,2,3}};
    for (int i = 0; i < size; i++) {
        for (auto a : arr5[i]) {
            cout << a << "=";
        }
        cout << endl;
    }
    delete[] arr5;
    arr5 = nullptr;

    // 2 不连续空间
    int width = 4;
    int height = 5;
    // 指针数组
    int **arr6 = new int*[height]{ 0 };
    for (int i = 0; i < height; i++) {
        arr6[i] = new int[width]{ 0 };
    }
    arr6[1][1] = 99;
    for (int i = 0; i < height; i++) {
        for (int j = 0; j < width; j++) {
            cout << arr6[i][j] << " ";
        }
        cout << endl;
    }
    // 释放空间
    for (int i = 0; i < height; i++) {
        // delete arr6[i];
        delete[] arr6[i];
    }
    delete[] arr6;
    arr6 = nullptr;

    return 0;
}
```