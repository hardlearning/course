## 2.2 互斥体和锁mutex

### 2.2.1 互斥锁mutex

- lock()：阻塞当前线程，直到获取锁

```cpp
#include <thread>
#include <iostream>
#include <mutex>
using namespace std;

static mutex mtx;

void TestThread() {
    // 获取锁资源，如果没有则阻塞等待
    mtx.lock();
    cout << "===============================" << endl;
    cout << "test 001" << endl;
    cout << "test 002" << endl;
    cout << "test 003" << endl;
    cout << "===============================" << endl;
    mtx.unlock();
}

int main(int argc, char* argv[]) {
    for (int i = 1; i < 10; i++) {
        thread th(TestThread);
        th.detach();
    }
    getchar();
    return 0;
}
```

- try_lock()：不阻塞线程，返回是否成功获取到锁

```cpp
#include <thread>
#include <iostream>
#include <mutex>
using namespace std;

static mutex mux;

void TestThread() {
    for (;;) {
        // try_lock返回true表示成功获取锁，返回false表示锁已被占用
        if (!mux.try_lock()) {
            cout << "." << flush;
            this_thread::sleep_for(100ms);
            continue;
        }
        cout << "===============================" << endl;
        cout << "test 001" << endl;
        cout << "test 002" << endl;
        cout << "test 003" << endl;
        cout << "===============================" << endl;
        mux.unlock();
        this_thread::sleep_for(1000ms);
    }
}

int main(int argc, char* argv[]) {
    for (int i = 0; i < 10; i++) {
        thread th(TestThread);
        th.detach();
    }
    getchar();
    return 0;
}
```

### 2.2.2 互斥锁的坑：线程抢占不到资源

```cpp
#include <thread>
#include <iostream>
#include <mutex>
using namespace std;

static mutex mux;

void ThreadMainMux(int i) {
    for (;;) {
        mux.lock();
        cout << i << "[in]" << endl;
        this_thread::sleep_for(1000ms);
        mux.unlock();
        // 注释下一行，会出现一个线程一直占用锁的情况，因为线程切换也有开销
        this_thread::sleep_for(1ms);
    }
}

int main(int argc, char* argv[]) {
    for (int i = 0; i < 3; i++) {
        thread th(ThreadMainMux, i + 1);
        th.detach();
    }
    getchar();
    return 0;
}
```

### 2.2.3 超时锁的应用timed_mutex（避免长时间死锁）

```cpp
#include <thread>
#include <iostream>
#include <mutex>
using namespace std;

timed_mutex tmux;

void ThreadMainTime(int i) {
    for (;;) {
        if (!tmux.try_lock_for(chrono::milliseconds(500))) {
            cout << i << "[try_lock_for timeout]" << endl;
            continue;
        }
        cout << i << "[in]" << endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(2000));
        tmux.unlock();
        std::this_thread::sleep_for(std::chrono::milliseconds(1));
    }
}

int main(int argc, char* argv[]) {
    for (int i = 0; i < 3; i++) {
        thread th(ThreadMainTime, i + 1);
        th.detach();
    }
    getchar();
    return 0;
}
```

### 2.2.4 递归锁（可重入）recursive_mutex和recursive_timed_mutex

- 同一个线程中的同一把锁可以锁多次。避免了一些不必要的死锁

```cpp
#include <thread>
#include <iostream>
#include <mutex>
using namespace std;

recursive_mutex rmux;

void Task1() {
    rmux.lock();
    cout << "task1 [in]" << endl;
    rmux.unlock();
}

void Task2() {
    rmux.lock();
    cout << "task2 [in]" << endl;
    rmux.unlock();
}

void ThreadMainRec(int i) {
    for (;;) {
        rmux.lock();
        Task1();
        cout << i << "[in]" << endl;
        this_thread::sleep_for(2000ms);
        Task2();
        rmux.unlock();
        this_thread::sleep_for(1ms);
    }
}

int main(int argc, char* argv[]) {
    for (int i = 0; i < 3; i++) {
        thread th(ThreadMainRec, i + 1);
        th.detach();
    }
    getchar();
    return 0;
}
```

### 2.2.5 共享锁shared_mutex

- c++14共享超时互斥锁shared_timed_mutex
- c++17共享互斥锁shared_mutex
- 如果有一个线程在写，其他线程既不能读也不能写；如果有一个线程在读，其他线程都可以读，但是要等全部读线程结束才能写。

```cpp
#include <thread>
#include <iostream>
#include <shared_mutex>
using namespace std;

// c++17
// shared_mutex smux;

// c++14
shared_timed_mutex stmux;

void ThreadRead(int i) {
    for (;;) {
        stmux.lock_shared();
        cout << i << " Read" << endl;
        this_thread::sleep_for(500ms);
        stmux.unlock_shared();
        this_thread::sleep_for(1ms);
    }
}

void ThreadWrite(int i) {
    for (;;) {
        stmux.lock_shared();
        // 读取数据
        stmux.unlock_shared();
        // 互斥锁 写入
        stmux.lock();
        cout << i << " Write" << endl;
        this_thread::sleep_for(300ms);
        stmux.unlock();
        this_thread::sleep_for(1ms);
    }
}

int main(int argc, char* argv[]) {
    for (int i = 0; i < 3; i++) {
        thread th(ThreadWrite, i + 1);
        th.detach();
    }
    for (int i = 0; i < 3; i++) {
        thread th(ThreadRead, i + 1);
        th.detach();
    }
    getchar();
    return 0;
}
```