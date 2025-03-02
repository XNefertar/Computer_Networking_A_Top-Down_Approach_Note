## 3.1 - 3.3

### R1

#### a:

要求设计最简单的传输层协议，且要求必须可以使应用程序数据到达位于目的主机的对应进程，所以只需要在报文段首部加上应用进程的目标端口号（4字节）即可，所以每次可传输数据的最大长度为`1200 - 4 = 1196` bytes；

`|  目的端口号（4字节）   |   数据（最大接收1196字节）  |`

#### b:

要求提供一个返回地址，只需要在报文段首部加上一个源端口号即可，此时可传输的最大数据量为`1200 - 4 - 4 = 1192`bytes

`|  源端口号（4字节）  |  目的端口号（4字节）   |   数据（最大接收1196字节）  |`

#### c:

不需要

1. 传输层的端到端原则：传输层的功能仅在通信的终端主机实现
2. 传输层只在应用层和网络层担任数据传输的桥梁作用，其底层的数据传输有其他底层协议负责（IP协议、PPP协议等），无需参与网络核心的工作；

### R2

#### a:

参考复习题R1，设计一个协议处理两家的信件收发问题：

**发送方:**

1. 发送方家庭将信件和对应的接收人姓名交由家庭代表
2. 家庭代表将接收人姓名标识在信件上
3. 家庭代表封装信件，并在信封上标识寄信的地址，交由邮政服务发送

**接收方**

1. 目的家庭收到信封后，验证发信地址，无误后拆开信封
2. 根据接收人姓名派发信件

#### b:

不需要，因为对应的寄信地址已经标识在信封上，无需拆开信封检查

### R3

y;x

### R4

1. 数据传输的实时性，比如游戏服务等，由于TCP的可靠性传输，必然会带来较大的时间成本和资源的消耗，对于一些要求高实时性的服务来说，UDP无疑是更好的选择；
2. 支持广播与多播，与TCP面向连接不同，UDP无连接性，支持广播；
3. 广泛用于DNS查询，可快速解析域名；
4. 可自定义可靠传输，可根据需要自定义传输的特性；

### R5

这是非实时性传输，要求数据的可靠性传输，所以优先采用TCP协议，而对于如语音通话这类对实时性要求较高的服务，则优先考虑UDP；

### R6

可能；
如何在UDP上实现类似TCP的数据可靠性传输？首先需要考虑TCP是如何实现数据可靠性传输的？

1. 确认应答机制

2. 数据完整性检验（CRC校验和）

3. 超时重传

4. 拥塞控制

5. 滑动窗口

   ......

我们需要参考这些实现一个UDP上数据的可靠性传输，下面是一个简单的演示：

1. 数据报编号（参考序列号）与顺序控制

   1. 序列号

      每个包都编以唯一递增的序列号，用于标识数据包顺序

   2. 乱序处理

      根据序列号进行乱序重组

2. 确认与重传机制（ACK && Retransmission）

   1. 确认应答（ACK）

      类似TCP

   2. 超时重传（RTO）

      动态维护一个重传计时器，用于判断是否超时

      `根据网络往返时间（RTT）动态调整超时阈值`

3. 流量控制与窗口机制

   1. 滑动窗口

      接收窗口（rwnd），通过ACK告知发送方剩余缓冲区大小

4. 拥塞控制

   1. 拥塞窗口

      拥塞窗口（cwnd），用于TCP的拥塞控制算法

      1. 慢启动
      2. 拥塞避免
      3. 快速恢复

5. 数据完整性校验

   1. 校验和

      CRC校验和算法检验

### R7

会，UDP套接字（socket）只由`本地IP地址`和`本地端口号`唯一标识：

- 本地IP地址：
  1. 如果套接字绑定到通配地址（0.0.0.0），则会接收所有IP发往对应端口的报文；
  2. 如果绑定到指定IP地址，则仅接收发往对应IP和端口的报文；

区别两台不同的主机则是通过其`源IP地址`和`源端口号`实现的，可以通过`recvfrom`系统调用获取对应信息；

_函数原型_

```c++
#include <sys/socket.h>

// ssize_t recv(int sockfd, void buf[.len], size_t len,
//                  int flags);       
ssize_t recvfrom(int sockfd, void buf[restrict .len], size_t len,
                 int flags,
                 struct sockaddr *_Nullable restrict src_addr,
                 socklen_t *_Nullable restrict addrlen);
//ssize_t recvmsg(int sockfd, struct msghdr *msg, int flags);
```

_获取示例_

```c++
struct sockaddr_in sender_addr;
socklen_t addr_len = sizeof(sender_addr);
char buffer[1024];
ssize_t recv_size = recvfrom(sock_fd, buffer, sizeof(buffer), 0, 
                            (struct sockaddr*)&sender_addr, &addr_len);

// 提取源IP和源端口
char sender_ip[INET_ADDRSTRLEN];
inet_ntop(AF_INET, &sender_addr.sin_addr, sender_ip, sizeof(sender_ip));
uint16_t sender_port = ntohs(sender_addr.sin_port);
```

### R8

不是；

首先，我们需要知道TCP和UDP协议的标识方式是不同的：

1. UDP：只需目的端口和目的IP即可标识一个UDP套接字，其具体的执行全过程如图所示：

   ![img](https://i-blog.csdnimg.cn/blog_migrate/08c7d6800f2911ad26b177fdbe0a4939.png)

2. TCP，不同于UDP无连接特性，一个TCP的建立不仅需要服务端的`bind绑定`，还需要客户端调用`connect`连接函数，所以一个TCP是由一个四元组`<本地IP，本地端口，远程IP，远程端口>`所唯一标识的，其具体执行过程如图所示：

![img](https://i-blog.csdnimg.cn/blog_migrate/8f9baa76483eb59f158a4302e6507f89.png)

> 图片来自CSDN Blog，[原文链接](https://blog.csdn.net/m0_37925202/article/details/80286946)

此外，Web服务器是基于TCP实现的应用程序，所以我们可以得出结论，由于主机A和B具有不同的源端口和目的端口，TCP建立套接字并不相同；

二者都具有端口号80，可以共享本地端口，但是由于A,B的其他信息不一样，四元组唯一，依然可以正常完成TCP通信；