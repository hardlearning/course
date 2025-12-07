## 2.4 条件变量

## 2.4.1 condition_variable

```cpp
#include <condition_variable>
#include <iostream>
#include <list>
#include <mutex>
#include <sstream>
#include <thread>
using namespace std;

list<string> msgs_;
mutex mux;
condition_variable cv;

void ThreadWrite() {
    for (int i = 0;; i++) {
        stringstream ss;
        ss << "Write msg " << i;
        unique_lock<mutex> lock(mux);
        msgs_.push_back(ss.str());
        lock.unlock();
        cv.notify_one();  // 发送信号
        // cv.notify_all();
        this_thread::sleep_for(chrono::seconds(1));
    }
}

void ThreadRead(int i) {
    for (;;) {
        cout << "read msg " << endl;
        unique_lock<mutex> lock(mux);
        cv.wait(lock);  // 解锁、阻塞等待信号
        // 获取信号后锁定
        while (!msgs_.empty()) {
            cout << i << " read " << msgs_.front() << endl;
            msgs_.pop_front();
        }
    }
}

int main(int argc, char* argv[]) {
    thread thw(ThreadWrite);
    thw.detach();
    for (int i = 0; i < 3; i++) {
        thread thr(ThreadRead, i + 1);
        thr.detach();
    }
    getchar();
    return 0;
}
```

- 使用lambda表达式处理wait

```cpp
#include <condition_variable>
#include <iostream>
#include <list>
#include <mutex>
#include <sstream>
#include <thread>
using namespace std;

list<string> msgs_;
mutex mux;
condition_variable cv;

void ThreadWrite() {
    for (int i = 0;; i++) {
        stringstream ss;
        ss << "Write msg " << i;
        unique_lock<mutex> lock(mux);
        msgs_.push_back(ss.str());
        lock.unlock();
        cv.notify_one();  // 发送信号
        this_thread::sleep_for(chrono::seconds(3));
    }
}

void ThreadRead(int i) {
    for (;;) {
        cout << "read msg " << endl;
        unique_lock<mutex> lock(mux);
        // lambda返回true时不阻塞，返回false时阻塞
        cv.wait(lock, [i] {
            cout << i << " wait" << endl;
            // msgs_为空时阻塞等待信号
            // 可以把这里看成一个if条件，如果msgs_不为空代码才往下面走
            return !msgs_.empty();
        });
        // 获取信号后锁定
        while (!msgs_.empty()) {
            cout << i << " read " << msgs_.front() << endl;
            msgs_.pop_front();
        }
    }
}

int main(int argc, char* argv[]) {
    thread thw(ThreadWrite);
    thw.detach();
    for (int i = 0; i < 3; i++) {
        thread thr(ThreadRead, i + 1);
        thr.detach();
    }
    getchar();
    return 0;
}
```