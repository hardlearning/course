# 5 opencv灰度图做反色示例

- 指针操作二维数组对opencv灰度图做反色

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace std;
using namespace cv;

int main() {
    // 400x400的灰度图 做反色 黑色->白色
    // 一个字节unsigned char表示灰度
    // 255-灰度 做反色
    auto img = imread("lena_hed.jpg", IMREAD_GRAYSCALE);
    int height = img.rows;
    int width = img.cols;
    cout << "img.elemSize() = " << img.elemSize() << endl;
    imshow("test1", img);
    // 通过指针访问连续的二维数组
    for (int i = 0; i < height; i++) {
        for (int j = 0; j < width; j++) {
            auto c = img.data[i * width + j];
            img.data[i * width + j] = 255 - c;
        }
    }
    imshow("test2", img);
    waitKey(5000);
    return 0;
}
```