# 5 C++20线程特性

## 5.1 std::barrier屏障

- arrive: 到达屏障并减少期待计数。
- wait: 阻塞等待，直到期待计数为0时其阶段完成。
- arrive_and_wait: 到达屏障并把期待计数减少一，然后阻塞直到当前阶段完成。
- arrive_and_drop: 将后继阶段的初始期待计数和当前阶段的期待计数都减少一。

```cpp
#include <iostream>
#include <barrier>
#include <chrono>
#include <thread>
using namespace std;

void TestBar(int i, barrier<> *bar) {
    this_thread::sleep_for(chrono::seconds(i));
    cout << i << " begin wait" << endl;
    // arrive 期待数-1
    // wait 阻塞等待，期待为0时返回
    bar->wait(bar->arrive());
    cout << i << " end wait" << endl;
}

int main(int argc, char *argv[]) {
    int count = 3;
    barrier bar(count);  // 初始数量
    for (int i = 0; i < count; i++) {
        thread th(&TestBar, i, &bar);
        th.detach();
    }

    getchar();
    return 0;
}
```