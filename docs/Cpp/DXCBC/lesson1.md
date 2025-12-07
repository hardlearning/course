# 1 C++11多线程快速入门

## 1.1 第一个C++11多线程程序

```cpp
#include <thread>
#include <iostream>
using namespace std;

void ThreadMain() {
    cout << "begin sub thread main: " << this_thread::get_id() << endl;
    for (int i = 0; i < 10; i++) {
        cout << "in thread " << i << endl;
        this_thread::sleep_for(chrono::seconds(1));
    }
    cout << "end sub thread main: " << this_thread::get_id() << endl;
}

int main(int argc, char* argv[]) {
    cout << "main thread ID: " << this_thread::get_id() << endl;
    // 线程创建启动
    thread th(ThreadMain);
    cout << "begin wait sub thread" << endl;
    // 阻塞等待子线程退出
    th.join();
    cout << "end wait sub thread" << endl;
    return 0;
}
```

## 1.2 std::thread对象生命周期以及线程等待和分离

```cpp
#include <thread>
#include <iostream>
using namespace std;

bool is_exit = false;
void ThreadMain() {
    cout << "begin sub thread main: " << this_thread::get_id() << endl;
    for (int i = 0; i < 10; i++) {
        if (is_exit) break;
        cout << "in thread " << i << endl;
        this_thread::sleep_for(chrono::seconds(1));
    }
    cout << "end sub thread main: " << this_thread::get_id() << endl;
}

int main(int argc, char* argv[]) {
    {
        // 出错，thread对象被销毁，子线程还在运行
        // thread th(ThreadMain);
    }
    {
        cout << "begin main thread" << endl;
        thread th(ThreadMain);
        // 子线程与主线程分离，守护线程
        th.detach();  // 坑：主线程推出后，子线程不一定退出
        cout << "end main thread" << endl;
    }
    {
        thread th(ThreadMain);
        this_thread::sleep_for(chrono::seconds(1));
        // 通知子线程退出
        is_exit = true;
        cout << "waiting for sub thread exit" << endl;
        // 主线程阻塞，等待子线程退出
        th.join();
        cout << "sub thread exited" << endl;
    }
    getchar();
    return 0;
}
```

## 1.3 C++11线程创建的多种方式

### 1.3.1 全局函数作为线程入口

```cpp
#include <thread>
#include <iostream>
using namespace std;

class Para {
public:
    Para() { cout << "Create Para" << endl; }
    Para(const Para &p) { cout << "Copy Para" << endl; }
    ~Para() { cout << "Drop Para" << endl; }
    string name;
};

void ThreadMain(int p1, float p2, string p3, Para p4) {
    this_thread::sleep_for(100ms);
    cout << "ThreadMain " << p1 << " " << p2 << " " << p3 << " " << p4.name << endl;
}

int main(int argc, char* argv[]) {
    thread th;
    {
        Para p;
        p.name = "test Para class";
        float f1 = 12.1f;
        // 所有参数都是拷贝，所以f1销毁不影响子线程中打印
        th = thread(ThreadMain, 101, f1, "test string", p);
    }
    th.join();
    return 0;
}
```

#### 参数传递的一些坑

- 传递空间已经销毁
- 多线程共享访问同一块空间
- 传递的指针变量的生命周期小于线程

1. 传递指针

```cpp
#include <thread>
#include <iostream>
using namespace std;

class Para {
public:
    Para() { cout << "Create Para" << endl; }
    Para(const Para &p) { cout << "Copy Para" << endl; }
    ~Para() { cout << "Drop Para" << endl; }
    string name;
};

void ThreadMainPtr(Para *p) {
    this_thread::sleep_for(100ms);
    cout << "ThreadMainPtr name = " << p->name << endl;
}

int main(int argc, char* argv[]) {
    {
        // 给线程传递指针
        Para p;
        p.name = "test";
        thread th(ThreadMainPtr, &p);
        // th.join();
        th.detach();  // 错误：线程访问的p空间会提前释放
    }
    // Para释放
    getchar();
    return 0;
}
```

2. 传递引用

```cpp
#include <thread>
#include <iostream>
using namespace std;

class Para {
public:
    Para() { cout << "Create Para" << endl; }
    Para(const Para &p) { cout << "Copy Para" << endl; }
    ~Para() { cout << "Drop Para" << endl; }
    string name;
};

void ThreadMainRef(Para &p) {
    this_thread::sleep_for(100ms);
    cout << "ThreadMainRef name = " << p.name << endl;
}

int main(int argc, char* argv[]) {
    {
        // 传递引用
        Para p;
        p.name = "test ref";
        thread th(ThreadMainRef, ref(p));
        th.join();
    }
    getchar();
    return 0;
}
```

### 1.3.2 成员函数作为线程入口

```cpp
#include <thread>
#include <iostream>
using namespace std;

class MyThread {
public:
    // 入口线程函数
    void Main() {
        cout << "MyThread Main " << name << ":" << age << endl;
    }
    string name;
    int age = 100;
};

int main(int argc, char* argv[]) {
    MyThread myth;
    myth.name = "Test name 001";
    myth.age = 20;
    thread th(&MyThread::Main, &myth);
    th.join();
    return 0;
}
```

#### 线程基类封装

```cpp
#include <thread>
#include <iostream>
using namespace std;

class XThread {
public:
    virtual void Start() {
        is_exit_ = false;
        th_ = std::thread(&XThread::Main, this);
    }
    virtual void Stop() {
        is_exit_ = true;
        Wait();
    }
    virtual void Wait() {
        if (th_.joinable()) {
            th_.join();
        }
    }
    bool is_exit() { return is_exit_; }
private:
    virtual void Main() = 0;
    std::thread th_;
    bool is_exit_ = false;
};

class TestXThread : public XThread {
public:
    void Main() override {
        cout << "TestXThread Main begin" << endl;
        while (!is_exit()) {
            this_thread::sleep_for(100ms);
            cout << "." << flush;
        }
        cout << "TestXThread Main end" << endl;
    }
    string name;
};

int main(int argc, char* argv[]) {
    TestXThread testth;
    testth.name = "TestXThread name";
    testth.Start();

    this_thread::sleep_for(3s);
    testth.Stop();

    testth.Wait();
    getchar();
    return 0;
}
```

### 1.3.3 lambda临时函数作为线程入口函数

lambda函数基本格式：[捕捉列表](参数) mutable -> 返回值类型 {函数体}

#### lambda线程传递参数

```cpp
#include <thread>
#include <iostream>
using namespace std;

int main(int argc, char* argv[]) {
    thread th(
        [](int i){ cout << "test lambda " << i << endl; },
        123
    );
    th.join();
    return 0;
}
```

#### 线程入口为类成员lambda函数

```cpp
#include <thread>
#include <iostream>
using namespace std;

class TestLambda {
public:
    void Start() {
        thread th([this]() { cout << "name = " << name << endl; });
        th.join();
    }
    string name = "test lambda";
};

int main(int argc, char* argv[]) {
    TestLambda test;
    test.Start();
    return 0;
}
```