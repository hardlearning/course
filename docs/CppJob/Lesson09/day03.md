# 09.Linux网络编程

## TCP状态转换图

![tcp-status-1](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Network/tcp-status-1.png)

![tcp-status-2](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Network/tcp-status-2.png)

1. 三次握手过程:

- 客户端：SYN_SENT--connect()   
- 服务端：LISTEN--listen()  SYN_RCVD
- 当三次握手完成后，都处于ESTABLISHED状态   

2. 数据传输过程中状态不发生变化，都是ESTABLISHED状态
3. 四次挥手过程:

- 主动关闭方：FIN_WAIT_1  FIN_WAIT_2  TIME_WAIT
- 被动关闭方：CLOSE_WAIT  LAST_ACK