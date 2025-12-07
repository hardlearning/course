## 2.3 利用栈特性自动释放锁RAII

### 2.3.1 什么是RAII

RAII(Resource Acquisition Is Initialization): 使用局部对象来管理资源的技术称为资源获取即初始化；它的生命周期是由操作系统来管理的，无需人工介入；以防止资源的销毁容易忘记，造成死锁或内存泄漏。

手动实现RAII管理mutex资源

```cpp
#include <iostream>
#include <mutex>
using namespace std;

// RAII
class XMutex {
public:
    XMutex(mutex &mux) : mux_(mux) {
        cout << "Lock" << endl;
        mux_.lock();
    }
    ~XMutex() {
        cout << "Unlock" << endl;
        mux_.unlock();
    }
private:
    mutex &mux_;
};

static mutex mux;

void TestMutex(int status) {
    XMutex lock(mux);
    if (status == 1) {
        cout << "=1" << endl;
    } else {
        cout << "!=1" << endl;
    }
}

int main(int argc, char* argv[]) {
    TestMutex(1);
    TestMutex(2);
    return 0;
}
```

### 2.3.2 c++11支持的RAII管理互斥资源lock_guard

- 通过{}控制锁的临界区

```cpp
#include <thread>
#include <iostream>
#include <mutex>
using namespace std;

static mutex gmutex;

void TestLockGuard(int i) {
    for (;;) {
        {
            lock_guard<mutex> lock(gmutex);
            cout << "In " << i << endl;
        }
        this_thread::sleep_for(500ms);
    }
}

int main(int argc, char* argv[]) {
    for (int i = 0; i < 3; i++) {
        thread th(TestLockGuard, i + 1);
        th.detach();
    }
    getchar();
    return 0;
}
```

- 假设调用方已经拥有互斥锁，使用adopt_lock

```cpp
#include <thread>
#include <iostream>
#include <mutex>
using namespace std;

static mutex gmutex;

void TestLockGuard(int i) {
    gmutex.lock();
    {
        // 死锁
        // lock_guard<mutex> lock(gmutex);
        // 已经拥有锁，不lock
        lock_guard<mutex> lock(gmutex, adopt_lock);
        cout << "begin thread " << i << endl;
    }
}

int main(int argc, char* argv[]) {
    for (int i = 0; i < 3; i++) {
        thread th(TestLockGuard, i + 1);
        th.detach();
    }
    getchar();
    return 0;
}
```

### 2.3.3 unique_lock C++11

unique_lock实现可移动的互斥体所有权包装器

- 支持临时释放锁

```cpp
#include <iostream>
#include <mutex>
using namespace std;

int main(int argc, char* argv[]) {
    {
        static mutex mux;
        {
            unique_lock<mutex> lock(mux);
            lock.unlock();
            // 临时释放锁
            lock.lock();
        }  
    }
    return 0;
}
```

- 支持adopt_lock：已经拥有锁，不加锁，出栈区会释放。

```cpp
{
    mux.lock();
    // 已经拥有锁，不锁定，退出栈区解锁
    unique_lock<mutex> lock(mux, adopt_lock);
}
```

- 支持defer_lock：延后拥有锁，不加锁，出栈区不释放

```cpp
{
    // 延后加锁，不锁定，退出栈区不解锁
    unique_lock<mutex> lock(mux, defer_lock);
    // 加锁，退出栈区解锁
    lock.lock();
}
```

- 支持try_to_lock：尝试获得互斥的所有权而不阻塞，获取失败退出栈区不会释放，通过owns_lock()函数判断是否获取成功。

```cpp
{
    // mux.lock();
    // 尝试加锁，不阻塞，失败不拥有锁
    unique_lock<mutex> lock(mux, try_to_lock);
    if (lock.owns_lock()) {
        cout << "owns lock" << endl;
    } else {
        cout << "not owns lock" << endl;
    }
}
```

### 2.3.4 shared_lock C++14

shared_lock实现可移动的共享互斥体所有权封装器

```cpp
#include <iostream>
#include <mutex>
#include <shared_mutex>
using namespace std;

int main(int argc, char* argv[]) {
    {
        // 共享锁
        static shared_timed_mutex tmux;
        // 1 读取锁 共享锁
        {
            // 调用共享锁
            shared_lock<shared_timed_mutex> lock(tmux);
            cout << "read data" << endl;
            // 退出栈区，释放共享锁
        }
        // 2 写入锁 互斥锁
        {
            unique_lock<shared_timed_mutex> lock(tmux);
            cout << "write data" << endl;
        }
    }
    return 0;
}
```

### 2.3.5 scoped_lock C++17

scoped_lock用于多个互斥体的免死锁RAII封装器，类似c++11中的lock

- 模拟死锁

```cpp
#include <iostream>
#include <mutex>
#include <shared_mutex>
#include <thread>
using namespace std;

static mutex mux1;
static mutex mux2;

void TestScope1() {
    // 模拟死锁，停100ms等另一个线程锁住mux2
    this_thread::sleep_for(100ms);
    cout << "begin mux1 lock " << this_thread::get_id() << endl;
    mux1.lock();
    cout << "begin mux2 lock " << this_thread::get_id() << endl;
    mux2.lock();  // 死锁
    cout << "TestScope1" << endl;
    this_thread::sleep_for(1000ms);
    mux1.unlock();
    mux2.unlock();
}

void TestScope2() {
    cout << "begin mux2 lock " << this_thread::get_id() << endl;
    mux2.lock();
    this_thread::sleep_for(500ms);
    cout << "begin mux1 lock " << this_thread::get_id() << endl;
    mux1.lock();  // 死锁
    cout << "TestScope2" << endl;
    this_thread::sleep_for(1500ms);
    mux1.unlock();
    mux2.unlock();
}

int main(int argc, char* argv[]) {
    {
        {
            thread th(TestScope1);
            th.detach();
        }
        {
            thread th(TestScope2);
            th.detach();
        }
    }
    getchar();
    return 0;
}
```

- c++11使用lock防止死锁

```cpp
#include <iostream>
#include <mutex>
#include <shared_mutex>
#include <thread>
using namespace std;

static mutex mux1;
static mutex mux2;

void TestScope1() {
    // 模拟死锁，停100ms等另一个线程锁住mux2
    this_thread::sleep_for(100ms);
    cout << "begin mux1 and mux2 lock " << this_thread::get_id() << endl;
    // c++11 lock必须同时锁住两个才进行下一步操作，如果没有锁住不会占用两个锁
    lock(mux1, mux2);
    cout << "TestScope1" << endl;
    this_thread::sleep_for(1000ms);
    mux1.unlock();
    cout << "end mux1 lock " << this_thread::get_id() << endl;
    mux2.unlock();
    cout << "end mux2 lock " << this_thread::get_id() << endl;
}

void TestScope2() {
    cout << "begin mux2 lock " << this_thread::get_id() << endl;
    mux2.lock();
    this_thread::sleep_for(500ms);
    cout << "begin mux1 lock " << this_thread::get_id() << endl;
    mux1.lock();  // 死锁
    cout << "TestScope2" << endl;
    this_thread::sleep_for(1500ms);
    mux1.unlock();
    cout << "end mux1 lock " << this_thread::get_id() << endl;
    mux2.unlock();
    cout << "end mux2 lock " << this_thread::get_id() << endl;
}

int main(int argc, char* argv[]) {
    {
        {
            thread th(TestScope1);
            th.detach();
        }
        {
            thread th(TestScope2);
            th.detach();
        }
    }
    getchar();
    return 0;
}
```

- c++17使用scoped_lock防止死锁

```cpp
#include <iostream>
#include <mutex>
#include <shared_mutex>
#include <thread>
using namespace std;

static mutex mux1;
static mutex mux2;

void TestScope1() {
    // 模拟死锁，停100ms等另一个线程锁住mux2
    this_thread::sleep_for(100ms);
    cout << "begin mux1 and mux2 lock " << this_thread::get_id() << endl;
    // 与c++11中的lock区别是：scoped_lock会自动释放掉
    scoped_lock lock(mux1, mux2);
    cout << "TestScope1" << endl;
    this_thread::sleep_for(1000ms);
}

void TestScope2() {
    cout << "begin mux2 lock " << this_thread::get_id() << endl;
    mux2.lock();
    this_thread::sleep_for(500ms);
    cout << "begin mux1 lock " << this_thread::get_id() << endl;
    mux1.lock();  // 死锁
    cout << "TestScope2" << endl;
    this_thread::sleep_for(1500ms);
    mux1.unlock();
    cout << "end mux1 lock " << this_thread::get_id() << endl;
    mux2.unlock();
    cout << "end mux2 lock " << this_thread::get_id() << endl;
}

int main(int argc, char* argv[]) {
    {
        {
            thread th(TestScope1);
            th.detach();
        }
        {
            thread th(TestScope2);
            th.detach();
        }
    }
    getchar();
    return 0;
}
```