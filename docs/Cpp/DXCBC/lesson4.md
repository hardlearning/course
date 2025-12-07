# 4 线程池实现

## 4.1 线程池基础版本

- xthread_pool.h

```cpp
#pragma once

#include <condition_variable>
#include <functional>
#include <list>
#include <mutex>
#include <thread>
#include <vector>
#include <atomic>
#include <future>

class XTask {
public:
    virtual int Run() = 0;
    std::function<bool()> is_exit = nullptr;

    auto GetReturn() {
        // 阻塞等待
        return p_.get_future().get();
    }

    void SetValue(int value) {
        p_.set_value(value);
    }
private:
    // 用来接收返回值
    std::promise<int> p_;
};

class XThreadPool {
public:
    // 初始化线程池
    void Init(int num);

    // 启动所有线程，必须线调用Init
    void Start();

    // 线程池退出
    void Stop();

    // void AddTask(XTask *task);
    void AddTask(std::shared_ptr<XTask> task);

    // XTask* GetTask();
    std::shared_ptr<XTask> GetTask();

    // 线程池是否退出
    bool is_exit() { return is_exit_; }

    int task_run_count() { return task_run_count_; }

    ~XThreadPool() { Stop(); }
private:
    // 线程池线程的入口函数
    void Run();
    // 线程数量
    int thread_num_ = 0;
    std::mutex mux_;
    // std::vector<std::thread*> threads_;
    // 线程列表智能指针版本
    std::vector<std::shared_ptr<std::thread>> threads_;
    // std::list<XTask*> tasks_;
    std::list<std::shared_ptr<XTask>> tasks_;
    std::condition_variable cv_;
    // 线程池退出标识
    bool is_exit_ = false;
    // 正在运行的任务数量
    std::atomic<int> task_run_count_ = { 0 };
};
```

- xthread_pool.cpp

```cpp
#include "xthread_pool.h"
#include <iostream>

using namespace std;

void XThreadPool::Init(int num) {
    unique_lock<mutex> lock(mux_);
    this->thread_num_ = num;
    cout << "Thread pool Init " << this->thread_num_ << endl;
}

void XThreadPool::Start() {
    unique_lock<mutex> lock(mux_);
    if (thread_num_ <= 0) {
        cerr << "Please Init XThreadPool" << endl;
        return;
    }
    if (!threads_.empty()) {
        cerr << "Thread pool has started!" << endl;
        return;
    }
    // 启动线程
    for (int i = 0; i < thread_num_; i++) {
        // auto th = new thread(&XThreadPool::Run, this);
        auto th = make_shared<thread>(&XThreadPool::Run, this);
        threads_.push_back(th);
    }
}

void XThreadPool::Stop() {
    is_exit_ = true;
    cv_.notify_all();
    for (auto &th : threads_) {
        th->join();
    }
    unique_lock<mutex> lock(mux_);
    threads_.clear();
}

void XThreadPool::AddTask(std::shared_ptr<XTask> task) {
    unique_lock<mutex> lock(mux_);
    task->is_exit = [this] { return is_exit(); };
    tasks_.push_back(task);
    lock.unlock();
    cv_.notify_one();
}

std::shared_ptr<XTask> XThreadPool::GetTask() {
    unique_lock<mutex> lock(mux_);
    if (tasks_.empty()) {
        cv_.wait(lock);
    }
    if (tasks_.empty()) {
        return nullptr;
    }
    if (is_exit()) {
        return nullptr;
    }
    auto task = tasks_.front();
    tasks_.pop_front();
    return task;
}

void XThreadPool::Run() {
    cout << "begin XThreadPool Run" << endl;
    while (!is_exit()) {
        auto task = GetTask();
        if (!task) continue;
        ++task_run_count_;
        try {
            // task->Run();
            auto re = task->Run();
            task->SetValue(re);
        } catch (...) {

        }
        --task_run_count_;
    }
    cout << "end XThreadPool Run" << endl;
}
```

- main.cpp

```cpp
#include <iostream>

#include "xthread_pool.h"
using namespace std;

class MyTask : public XTask {
public:
    int Run() override {
        cout << "=========================================" << endl;
        cout << this_thread::get_id() << " MyTask " << name << endl;
        cout << "=========================================" << endl;
        for (int i = 0; i < 10; i++) {
            if (is_exit()) break;
            cout << "." << flush;
            this_thread::sleep_for(500ms);
        }
        return 100;
    }
    std::string name = "";
};

int main(int argc, char *argv[]) {
    XThreadPool pool;
    pool.Init(16);
    pool.Start();

    // MyTask task1;
    // task1.name = "test name 001";
    // pool.AddTask(&task1);
    {
        auto task1 = std::make_shared<MyTask>();
        task1->name = "test name 001";
        pool.AddTask(task1);

        auto task2 = std::make_shared<MyTask>();
        task2->name = "test name 002";
        pool.AddTask(task2);

        auto re = task2->GetReturn();
        cout << "task return value: " << re << endl;
    }

    this_thread::sleep_for(100ms);
    cout << "task run count = " << pool.task_run_count() << endl;

    this_thread::sleep_for(1s);
    pool.Stop();
    cout << "task run count = " << pool.task_run_count() << endl;

    getchar();
    return 0;
}
```

## 4.2 基于线程池实现音视频批量转码任务

- xvideo_task.h

```cpp
#pragma once
#include "xthread_pool.h"

class XVideoTask : public XTask {
public:
    std::string in_path;
    std::string out_path;
    int width = 0;
    int height = 0;
private:
    int Run() override;
};
```

- xvideo_task.cpp

```cpp
#include "xvideo_task.h"
#include <sstream>
using namespace std;

int XVideoTask::Run() {
    // ffmpeg -y -i test.mp4 -s 400x300 400.mp4 >log.txt 2>&1
    stringstream ss;
    ss << "ffmpeg -y -i " << in_path << " ";
    if (width > 0 && height > 0) {
        ss << " -s " << width << "x" << height << " ";
    }
    ss << out_path << " >" << this_thread::get_id() << ".txt 2>&1";
    return system(ss.str().c_str());
}
```

- main.cpp

```cpp
#include "xvideo_task.h"
using namespace std;

int main(int argc, char *argv[]) {
    XThreadPool pool;
    pool.Init(16);
    pool.Start();

    auto vtask1 = make_shared<XVideoTask>();
    vtask1->in_path = "test.mp4";
    vtask1->out_path = "640.mp4";
    vtask1->width = 640;
    vtask1->height = 480;
    pool.AddTask(vtask1);

    vtask1->GetReturn();

    return 0;
}
```

基于用户输入执行音视频转码任务

- main.cpp

```cpp
#include <iostream>
#include "xvideo_task.h"
using namespace std;

int main(int argc, char *argv[]) {
    XThreadPool pool;
    pool.Init(16);
    pool.Start();

    this_thread::sleep_for(200ms);
    for (;;) {
        this_thread::sleep_for(200ms);
        cout << "\n======================\n" << endl;
        auto task = make_shared<XVideoTask>();
        cout << "请输入命令(v e l): ";
        string cmd;
        cin >> cmd;
        if (cmd == "e") {
            break;
        } else if (cmd == "l") {
            cout << "task run count = " << pool.task_run_count() << endl;
            continue;
        }
        cout << "视频源: ";
        cin >> task->in_path;
        cout << "目标: ";
        cin >> task->out_path;
        cout << "输出宽: ";
        cin >> task->width;
        cout << "输出高: ";
        cin >> task->height;
        pool.AddTask(task);
    }
    pool.Stop();

    return 0;
}
```