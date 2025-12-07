## 2.6 使用互斥锁+list模拟线程通信

- 封装线程基类XThread控制线程启动和停止
- 模拟消息服务器线程，接收字符串消息，并模拟处理
- 通过unique_lock和mutex互斥访问list<string>消息队列
- 主线程定时发送消息给子线程

- xthread.h

```cpp
#pragma once
#include <thread>

class XThread {
public:
    // 启动线程
    virtual void Start();
    // 设置线程退出标志并等待
    virtual void Stop();
    // 等待线程退出（阻塞）
    virtual void Wait();
    // 线程是否退出
    bool is_exit();
private:
    // 线程入口
    virtual void Main() = 0;
    bool is_exit_ = false;
    std::thread th_;
};
```

- xthread.cpp

```cpp
#include "xthread.h"

void XThread::Start() {
    is_exit_ = false;
    th_ = std::thread(&XThread::Main, this);
}

void XThread::Stop() {
    is_exit_ = true;
    Wait();
}

void XThread::Wait() {
    if (th_.joinable()) {
        th_.join();
    }
}

bool XThread::is_exit() {
    return is_exit_;
}
```

- xmsg_server.h

```cpp
#pragma once
#include <condition_variable>
#include <list>
#include <mutex>
#include "xthread.h"

class XMsgServer : public XThread {
public:
    // 给当前线程发消息
    void SendMsg(std::string msg);

    void Stop() override;
private:
    // 处理消息的线程入口函数
    void Main() override;
    // 消息队列缓冲
    std::list<std::string> msgs_;
    // 互斥访问消息队列
    std::mutex mux_;

    std::condition_variable cv_;
};
```

- xmsg_server.cpp

```cpp
#include "xmsg_server.h"
#include <iostream>

void XMsgServer::SendMsg(std::string msg) {
    std::unique_lock<std::mutex> lock(mux_);
    msgs_.push_back(msg);
    lock.unlock();
    cv_.notify_one();
}

void XMsgServer::Stop() {
    is_exit_ = true;
    cv_.notify_all();
    Wait();
}

void XMsgServer::Main() {
    while (!is_exit()) {
        std::unique_lock<std::mutex> lock(mux_);
        cv_.wait(lock, [this]() {
            std::cout << "wait cv" << std::endl;
            return !msgs_.empty() || is_exit();
        });
        while (!msgs_.empty()) {
            // 消息处理业务逻辑
            std::cout << "recv: " << msgs_.front() << std::endl;
            msgs_.pop_front();
        }
    }
}
```

- main.cpp

```cpp
#include <iostream>
#include <sstream>
#include "xmsg_server.h"

int main(int argc, char* argv[]) {
    XMsgServer server;
    server.Start();
    for (int i = 0; i < 10; i++) {
        std::stringstream ss;
        ss << " msg: " << i + 1;
        server.SendMsg(ss.str());
        std::this_thread::sleep_for(std::chrono::milliseconds(500));
    }
    server.Stop();
    std::cout << "Server stopped!" << std::endl;
    return 0;
}
```