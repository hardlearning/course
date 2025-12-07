# 3 C++17多核并行计算

实现base16编码：
- 二进制转为字符串
- 一个字节8位，拆分为两个4位字节（最大值16）
- 拆分后的字节映射到0123456789abcdef

```cpp
#include <future>
#include <thread>
#include <iostream>
#include <string>
#include <vector>
#include <chrono>
#include <execution>
using namespace std;
using namespace chrono;

static const char base16[] = "0123456789abcdef";

void Base16Encode(const unsigned char* data, int size, unsigned char* out) {
    for (int i = 0; i < size; i++) {
        unsigned char d = data[i];
        // 1234 5678 >>4 => 0000 1234
        char a = base16[d >> 4];
        // 1234 5678 & 0000 1111 => 0000 5678
        char b = base16[d & 0x0F];
        out[i * 2] = a;
        out[i * 2 + 1] = b;
    }
}

// C++11 多核base16编码
void Base16EncodeThread(const vector<unsigned char> &data, vector<unsigned char> &out) {
    int size = data.size();
    // 系统支持的线程核心数
    int th_count = thread::hardware_concurrency();
    // 切片数据
    int slice_count = size / th_count;  // 余数丢弃
    if (size < th_count) {  // 只切一片
        th_count = 1;
        slice_count = size;
    }

    // 准备好线程
    vector<thread> ths;
    ths.resize(th_count);
    // 任务分配到各个线程
    for (int i = 0; i < th_count; i++) {
        // 1234 5678 9abc defg hi
        int offset = i * slice_count;
        int count = slice_count;
        // 最后一个线程
        if (th_count > 1 && i == th_count - 1) {
            count = slice_count + size % th_count;
        }
        // cout << offset << ":" << count << endl;
        ths[i] = thread(Base16Encode, data.data() + offset, count, out.data());
    }
    // 等待所有线程处理结束
    for (auto &th : ths) {
        th.join();
    }
}

int main(int argc, char *argv[]) {
    string test_data = "测试base16编码";
    unsigned char out[1024] = { 0 };
    Base16Encode((unsigned char*) test_data.data(), test_data.size(), out);
    cout << "base16:" << out << endl;

    // 初始化测试数据
    vector<unsigned char> in_data;
    in_data.resize(1024 * 1024 * 20);  // 20M
    // in_data.resize(32);  // 改小测试输出结果是否一致
    for (int i = 0; i < in_data.size(); i++) {
        in_data[i] = i % 256;
    }
    vector<unsigned char> out_data;
    out_data.resize(in_data.size() * 2);

    // 测试单线程base16编码效率
    {
        cout << "单线程base16开始计算" << endl;
        auto start = chrono::steady_clock::now();
        Base16Encode(in_data.data(), in_data.size(), out_data.data());
        auto end = chrono::steady_clock::now();
        auto duration = duration_cast<milliseconds>(end - start);
        cout << "编码：" << in_data.size() << "字节数据花费" << duration.count() << "毫秒" << endl;
        // 打印输出结果
        // cout << out_data.data() << endl;
    }

    // 测试c++11多线程Base16编码
    {
        cout << "c++11 多线程base16开始计算" << endl;
        auto start = chrono::steady_clock::now();
        Base16EncodeThread(in_data, out_data);
        auto end = chrono::steady_clock::now();
        auto duration = duration_cast<milliseconds>(end - start);
        cout << "编码：" << in_data.size() << "字节数据花费" << duration.count() << "毫秒" << endl;
        // 打印输出结果
        // cout << out_data.data() << endl;
    }

    // 测试c++17多线程Base16编码
    {
        cout << "c++17 多线程base16开始计算" << endl;
        auto start = chrono::steady_clock::now();
        unsigned char* idata = in_data.data();
        unsigned char* odata = out_data.data();
        // std::execution::par表示多核并行计算
        // 这里for_each耗时较长主要是因为匿名函数出栈入栈开销
        std::for_each(std::execution::par, in_data.begin(), in_data.end(), [&](auto &d) {
            char a = base16[(d >> 4)];
            char b = base16[(d & 0x0F)];
            int index = &d - idata;
            odata[index * 2] = a;
            odata[index * 2 + 1] = b;
        });

        auto end = chrono::steady_clock::now();
        auto duration = duration_cast<milliseconds>(end - start);
        cout << "编码：" << in_data.size() << "字节数据花费" << duration.count() << "毫秒" << endl;
        // 打印输出结果
        // cout << odata << endl;
    }
    return 0;
}
```