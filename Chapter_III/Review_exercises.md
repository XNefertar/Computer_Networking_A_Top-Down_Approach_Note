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

## 3.4

这一节的主要内容就是关于可靠性传输原理的讲解，可以看作是对后面介绍TCP协议的铺垫了；
通过一步步优化，说明了可靠性传输在实际应用中所遇到的问题以及步步迭代优化最终实现了一个较为可靠的稳定实现：

> 1. 第一代 rdt 1.0：经完全可靠信道的可靠数据传输，在理想情况下，无需额外设计，数据传输就是可靠的；
> 2. 第二代 rdt 2.0：第二代考虑了传输过程中可能出现的`比特差错`问题，应对这种情况，第二代引入了重传（ARQ，自动重传请求协议）、确认机制（ACK, NCK）和校验和机制（常见为CRC校验和检验），为了保证数据的有序性到达，二代协议必须要阻塞式等待对应的ACK确认报文到来，所以第二代协议也被称为`停等协议`（stop-and-wait）；但是存在一个存在一个致命缺陷，如果确认报文发生丢失怎么办？数据会被错误的认为已经传递，由此带来了二代协议的改进版本；
> 3. rdt 2.1 改进版：为了解决确认报文（ACK、NAK）丢失问题，将每个数据包都赋予编号【或称序号】（编号 = 0 or 1，两种状态交替），发送方可以根据对应编号判断是否式对应数据的ACK/NCK；
> 4. 第三代 rdy 3.0：经具有比特差错的丢包信道的可靠数据传输，这一代考虑了传输过程中的丢包问题：如果发生了丢包问题，应该经过多长时间才执行重传操作呢？是不是需要用一个重传计时器？由此第三代引入了一个`倒计时定时器(countdown timer)`；因为分组序号在 0 和 1 之间交替，因此 rdt 3.0 有时被称为`比特交替协议(alternating-bit protocol)`

综上，我们便完成了对 rdt 协议迭代优化过程的简单讨论；

### R9

序号是为了处理在数据传输过程中确认报文（ACK/NCK）丢失的情况，在第二代协议中引入，在第三代协议中，序号也可以用来解决重传计时器设置不当导致的`冗余数据分组`问题；

### R10

定时器是第二代为了配合重传机制引入的特性，如果重传时间过短，会导致大量重复报文留存在收发信道；而重传时间过长，则会导致接收端长时间等待重传报文，降低传输效率；在第三代协议中，可用于检测丢包情况，当触发重传计时器，就考虑可能发生了丢包情况；

### R11

是必需的；

即使RTT已知，发送方依然需要某种机制来检测`丢包`或`ACK丢失`等情况，若没有定时器机制，发送方无法准确得知在RTT时间后有无ACK报文到达，且无法主动判断在给定时间内是否要进行数据重传；而ACK丢失可能会导致无限等待问题，导致协议停滞；

### R12

[动画链接](https://media.pearsoncmg.com/ph/esm/ecs_kurose_compnetwork_8/cw/content/interactiveanimations/go-back-n-protocol/index.html)

a. 乱序报文未确认，在经过一段时间后，报文重传；
b. 直接跳过了未收到确认的报文段，窗口继续向前滑动；
c. 超过窗口大小的部分无法发送；

### R13

[动画链接](https://media.pearsoncmg.com/ph/esm/ecs_kurose_compnetwork_8/cw/content/interactiveanimations/selective-repeat-protocol/index.html)

当删除传输中的ACK报文时，选择重传会记录下未收到ACK报文的下标（base值），等到重传时间后，从base发送所有未被确认的报文段；

## 3.5

### R14

| 题目 | 判断 | 解析                                                         |
| ---- | ---- | ------------------------------------------------------------ |
| a    | ❌    | 即使B没有数据要发，B仍然要发送ACK确认报文。TCP是全双工的，确认和数据是独立的。 |
| b    | ❌    | rwnd是接收窗口，会动态变化，受接收方缓存使用情况影响。       |
| c    | ✅    | 发送未确认的数据量受到接收窗口的限制。                       |
| d    | ❌    | 序号的增长取决于实际发送的数据长度，不一定是m+1，可能是m+数据长度。 |
| e    | ❌    | TCP首部有`window`字段（表示接收窗口大小），但没有名为`rwnd`的字段，rwnd是概念上的。 |
| f    | ✅    | TimeoutInterval是基于EstimatedRTT和DevRTT计算的，必定大于等于SampleRTT。 |
| g    | ❌    | 确认号是对对方数据的确认，不是对自己发送数据的确认，和序号无直接关系。 |

详细解释一下第f题：

首先明确`TimeoutInterval`确实基于`EstimatedRTT`和`DevRTT`计算的，相关的计算公式如下所示

> **EstimatedRTT** = (1 - α) * EstimatedRTT + α * SampleRTT
>
> **DevRTT** = (1 - β) * DevRTT + β * |SampleRTT - EstimatedRTT|
>
> **TimeoutInterval** = EstimatedRTT + 4 * DevRTT

_**为什么TimeoutInterval一定大于等于SampleRTT？**_

> - 即使网络非常稳定，`DevRTT`至少是0（这是理论极限），也就是说**TimeoutInterval = EstimatedRTT**。
> - `EstimatedRTT`本身就是对SampleRTT的平滑估计，它不会比SampleRTT还低。
> - 现实中，DevRTT基本上不会是0，意味着TimeoutInterval会比EstimatedRTT还要稍微大一些。

### R15

a. 20 字节；
b. 90；
注意，TCP序号是只针对数据的，不考虑其首部字节

> **序号不考虑报头部分** 
> **TCP序号编号的是 payload 中的每个字节**
> **TCP序号只关心数据，不关心数据**

### R16

总共发送了两个报文段；
假设之前的序号为`x`，则第一个报文段的序号为`x`，第二个报文段为`x + 1`；
确认字段两次都是一样的——为什么？
报文段的确认字段表示的是对端的确认，而对端B并未发送任何报文段，所以两次的确认字段是一致的；

## 3.7

### R17

平均分配，每条链路`R/2 bps`；

为什么？——这涉及到`TCP的带宽公平性`问题：

1. 拥塞控制算法会根据丢包和延迟反馈动态调整发送速率；
2. 两条链路共享瓶颈链路且对等竞争，TCP的`AIMD（加性增大/乘性减小）`机制会使得两条链路逐渐收敛到`平分带宽`的状态；
3. 这种带宽分享的机制，也是`TCP友好性(TCP Friendliness)`的体现；

_**TCP友好性（TCP Friendliness）**_

_**定义**_
**TCP友好性**指的是：

> 一个新协议或新的传输算法，在共享网络带宽时，它的行为与传统TCP（如TCP Reno）相“友好”，不会过于**抢占**带宽，也不会被TCP过度压制。

_**意义**_
协议的友好性是互联网发展至今的前提；

_**反例**_
UDP，不友好的，无拥塞控制，尽可能的占据网络带宽；

### R18

正确，具体解释见3.7节拥塞控制；

_**常见考点**_

1. 发送方定时器超时，`ssthresh`会被设置为原来的一半；

   - `ssthresh = cwnd / 2`

   - `cwnd -> 1`

   - `重新慢启动`

2. 三个重复ACK报文，`ssthresh`减半，进入`快速重传`和`快速恢复`状态；

   - `ssthresh = cwnd / 2`

   - `cwnd = ssthresh + 3（提前补偿三个确认）`

   - `进入快速恢复（非慢启动）`

3. 慢启动和拥塞避免的分界点是`ssthresh`控制的

   - `cwnd < ssthresh`：慢启动（指数增长）

   - `cwnd >= ssthresh`：拥塞避免（线性增长）

4. `TCP Reno`的慢启动阶段，`cwnd`每`RTT`窗口翻倍

   - `慢启动阶段是指数增长，每次ACK到来，cwnd + 1`

   - `每个RTT窗口翻倍`

5. 拥塞避免阶段，每收到一个`ACK`，`cwnd + (1 / cwnd)`

   - `拥塞避免阶段，cwnd每个RTT加1（线性增长），而不是每个ACK + 1`

6. `TCP Reno`的快速恢复阶段，`cwnd` 先减半再慢慢增加

   - `ssthresh = cwnd / 2`

   - `cwnd = ssthresh + 3`

   - `继续发送新数据，等待新ACK`

   - `每个ACK到来，cwnd + 1`

7. `TCP Tahoe`没有快速恢复阶段

   - `Tahoe的策略更保守`

   - `3个DupACK就直接进入慢启动`
   - `Reno改进了这一点，3个DupACK进入快速恢复`

### R19

_**TCP分岔**_
可以看作是一种负载均衡式的优化策略，主要用于**中间设备**（如代理、NAT、负载均衡等），用于加速TCP连接的转发；
该断言可以解释为：

​	**TCP分岔的延迟 = 建立连接的往返时间 + 数据确认 + 中间处理时间；**

可用于大致估计，适合`理论分析`或`定性判断`，但不适合精确测量；

