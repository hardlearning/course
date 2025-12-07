# 1 C++17内存池

#### memory_resource
- allocate: 申请内存，字节大小要是2的次方
- deallocate: 释放内存

#### unsynchronized_pool_resource: 线程不安全的内存池
- 属性:
    - _Chunks: 大块内存链表
    - _Pools: 普通大小内存链表
    - _Options: 内存池配置
- 方法:
    - release: 清空内存池所申请的内存

#### synchronized_pool_resource: 线程安全的内存池
- mutex: 互斥访问线程安全

#### pool_options: 内存池配置
- max_blocks_per_chunk: 内存池每块大小，用完则几何倍数增长
- largest_required_pool_block: 超过此大小使用大块内存链表

```cpp
#include <iostream>
#include <memory_resource>
#include <vector>
#include <thread>
using namespace std;
using namespace pmr;

// C++17 memory_resource内存池
int main() {
    pool_options opt;
    // 大数据块的字节数
    opt.largest_required_pool_block = 1024 * 1024 * 10;
    // 普通数据块，每块的字节数
    opt.max_blocks_per_chunk = 1024 * 1024 * 100;
    // 线程安全的内存池
    synchronized_pool_resource mpool(opt);
    int size = 1024 * 1024;
    std::vector<void*> datas;
    for (int i = 0; i < 1000; i++) {
        try {
            // 从内存池申请一块空间
            auto data = mpool.allocate(size);
            datas.push_back(data);
            cout << "+" << flush;
            this_thread::sleep_for(50ms);
        } catch (exception& e) {
            cerr << "mpool.allocate failed: " << e.what() << endl;
            exit(0);
        }
    }

    // 申请大块空间
    auto b1 = mpool.allocate(1024 * 1024 * 20);
    // 释放空间
    mpool.deallocate(b1, 1024 * 1024 * 20);

    for (auto d: datas) {
        mpool.deallocate(d, size);
        cout << "-" << flush;
        this_thread::sleep_for(50ms);
    }
    // 释放线程池的所有内存
    mpool.release();

    getchar();
    return 0;
}
```