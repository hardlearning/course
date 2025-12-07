## 2.5 线程异步和通信

### 2.5.1 promise和future

- promise用于异步传输变量：std::promise提供存储异步通信的值，再通过其对象创建的std::future异步获取结果。std::promise只能使用一次，void set_value(_Ty&& _Val)设置值传递，只能调用一次。
- future提供访问异步操作结果的机制：get()阻塞等待promise set_value的值。

```cpp
#include <future>
#include <iostream>
#include <string>
using namespace std;

void TestFuture(promise<string> p) {
    cout << "begin TestFuture" << endl;
    this_thread::sleep_for(3s);
    cout << "begin set value" << endl;
    p.set_value("TestFuture value");
    this_thread::sleep_for(3s);
    cout << "end TestFuture" << endl;
}

int main(int argc, char *argv[]) {
    // 异步传输变量
    promise<string> p;
    // 用来获取异步线程值
    future<string> future = p.get_future();

    auto th = thread(TestFuture, move(p));

    cout << "begin future.get()" << endl;
    cout << "future get() = " << future.get() << endl;
    cout << "end future.get()" << endl;
    th.join();
    return 0;
}
```

### 2.5.2 packaged_task异步调用函数打包

- package_task包装函数为一个对象，用于异步调用。其返回值能通过std::future对象访问。
- 与bind的区别：可异步调用，函数访问和获取返回值分开调用

```cpp
#include <future>
#include <iostream>
#include <string>
using namespace std;

string TestPack(int index) {
    cout << "begin TestPack " << index << endl;
    this_thread::sleep_for(2s);
    return "TestPack value";
}

int main(int argc, char *argv[]) {
    packaged_task<string(int)> task(TestPack);
    future<string> result = task.get_future();
    // 直接调用
    // task(100);

    // 多线程中调用
    thread th(move(task), 101);

    cout << "begin result get" << endl;
    cout << "result get " << result.get() << endl;

    th.join();
    getchar();
    return 0;
}
```

### 2.5.3 async

async是c++11中异步运行一个函数的方法，它返回保有其结果的std::future

- launch::defered延迟执行，在调用wait和get时，调用函数代码。
- launch::async创建线程（默认）。
- future.get()获取结果，会阻塞等待。

```cpp
#include <future>
#include <thread>
#include <iostream>
#include <string>
using namespace std;

string TestAsync(int index) {
    cout << index << " begin in TestAsync " << this_thread::get_id() << endl;
    this_thread::sleep_for(2s);
    return "TestAsync value";
}

int main(int argc, char *argv[]) {
    cout << "main thread id " << this_thread::get_id() << endl;
    // 不创建线程启动异步任务
    future<string> future1 = async(launch::deferred, TestAsync, 100);
    this_thread::sleep_for(100ms);
    cout << "begin future1 get" << endl;
    cout << "future1.get() = " << future1.get() << endl;
    cout << "end future1 get" << endl;

    cout << "======创建异步线程======" << endl;
    future<string> future2 = async(TestAsync, 101);
    this_thread::sleep_for(100ms);
    cout << "begin future2 get" << endl;
    cout << "future2.get() = " << future2.get() << endl;
    cout << "end future2 get" << endl;

    getchar();
    return 0;
}
```
