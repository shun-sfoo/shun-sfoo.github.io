---
title: "「TCP IP网络编程」读书笔记1-网络编程"
date: 2023-02-05T11:24:23+08:00
tags: ["socket"]
categories: ["阅读"]
draft: false
---

# socket

套接字是网络数据传输用的软件设备

## 网络编程中接受请求的套接字创建过程

1. 调用socket函数创建套接字
2. 调用bind函数分配ip地址和端口号
3. 调用listen函数将套接字转为可接受请求状态
4. 调用accept函数受理连接请求，如果在没有连接请求的情况下调用该函数，则不会返回，直到有连接请求为止。
5. write函数用于传输数据

### 服务器端套接字（监听套接字）

1. 调用socket函数创建套接字
2. 调用connect函数向服务器端发送连接请求

创建套接字，但此时并不马上分为服务端，客户端。如果紧接着调用bind，listen函数，将会成为服务器端套接字；如果调用connect函数将会成为客户端套接字。

## 基于linux的文件操作

在linux中socket操作与文件操作没有区别，socket也被认为是文件的一种，因此在网络数据传输过程中自然可以使用文件I/O的相关函数。

### 底层文件访问(Low-Level File Access)和文件描述符(File Descriptor)

底层表达可以理解为“与标准无关的操作系统独立提供的”。文件描述符指的是系统分配给文件或套接字的整数。标准输入输出和标准错误都是系统分配的文件描述符。

## 套接字类型和协议设置

协议“计算机间对话必备的通信规则”，为了完成数据交换而定好的约定。

```c
#include <sys/scoket.h>

int socket(int domain, int type, int protocol);
```

domain: 套接字中使用的协议族(protocol Family)信息。

type: 套接字数据传输类型信息。

protocl: 计算机间通信中使用的协议信息。

### 协议族(protocol family)

PF_INET
对应ipv4协议族，套接字中实际采用的最终协议是通过第三个参数传递的。在指定的协议族范围内通过第一个参数决定第三个参数。

### 套接字类型(type)

套接字类型是套接字的数据传输方式，通过这样socket函数的第二个参数传递，只有这样才能决定创建套接字的数据传输方式。因为决定了协议族并不能决定传输方式，换言之，协议族会有多种传输方式。两种代表性的数据传输方式

### 面向连接的套接字(SOCK_STREAM)

流水线的象征

特征如下：

1. 传输过程中数据不会消失
2. 按序传输
3. 传输的数据不存在数据边界

收发数据的套接字内部有缓冲，简而言之就是字符数组。通过套接字传输的数据将保存在该数组。因此收到数据并不意味着马上调用read函数。只要不超过数据容量，则有可能在填充缓冲后一次read函数调用读取全部，也有可能分成多次read函数调用读取。也就是说面向连接的套接字中，read和write函数调用的次数并无多大意义。所以说面向连接的套接字不存在边界。

面向连接除了特殊情况外不会发生数据丢失，面向连接的套接字必须与另外一个同样特性的套接字连接。缓冲满了之后，接受端无法接受数据，传输端也会停止传输。

### 面向消息的套接字(SOCK_DGRAM)

摩托车送快递的象征

1. 强调快速传输而非传输顺序
2. 传输的数据可能丢失也可能损坏
3. 传输的数据有数据边界
4. 限制每次传输的数据大小

用摩托车分别发送两次包裹，则接收者需要分两次接收，这种特性就是输出的数据有数据边界。面向消息的套接字不存在连接的概念。

### 协议的最终选择

大部分情况下可以向第三个参数传递0，除非遇到同一协议族中存在多个数据传递方式相同的协议。

PF_INET 中 SOCK_STREAM满足的协议只有 `IPPROTO_TCP`

`int tcp_socket = socket(PF_INET, SOCK_STREAM, IPPROTO_TCP);`

这种套接字称为tcp套接字

SOCK_DGRAM 只有`IPPROTO_UDP`

`int tcp_socket = socket(PF_INET, SOCK_STREAM, IPPROTO_UDP);`

这种套接字称为udp套接字。

### 表示IPV4的结构体 sockaddr_in(只为保存IPV4结构信息)

![sockaddr_in](/images/Socket/sockaddr_in.png)

大端序：高位存放在低位地址 0x1234 高位的12存放在低位 因此是 1234

小端序：高位存放在高位地址 intel 主流 0x1234 → 3412

**网络字节序统一为大端序**

填充sockaddr_in前将数据转换成网络字节序

### 字节序转换函数

```c
#include <arpa/inet.h>
unsigned short htons(unsigned short);

unsigned short ntohs(unsigned short);

unsigned long htonl(unsigned long);

unsigned short ntohl(unsigned long);
```

h代表 主机host字节序， n代表网络network字节序，
以s做后缀的函数中，s代表两个字节short，因此用于端口转换；以l作为后缀的函数中，l代表4个字节，因此用于IP地址转换。

除了向sockaddr_in结构体填充数据外，其他情况无需考虑字节序的问题(自动转换)

```c
memset(&serv_addr, 0, sizeof(serv_addr));
serv_addr.sin_family = AF_INET;
serv_addr.sin_addr.s_addr = inet_addr(argv[1]);
serv_addr.sin_port = htons(atoi(argv[2]));
```

### 网络地址的初始化及分配

```c
#include <arpa/inet.h>
extern in_addr_t inet_addr (const char *__cp) __THROW;
// i_addr_t 内部声明位32位整数型
```

这个函数会将字符串形式的IP地址转换成32位整数型数据。此函数转换的同时进行网络字节序转换。并且可以检测无效的IP(返回值为INADDR_NONE)

```c
#include <arpa/inet.h>
/* Convert Internet host address from numbers-and-dots notation in CP
   into binary data and store the result in the structure INP.  */
extern int inet_aton (const char *__cp, struct in_addr *__inp) __THROW;
```

更为常用的是inet_aton函数，需要传入in_addr结构体。

```c
#include <arpa/inet.h>
/* Convert Internet number in IN to ASCII representation.  The return value
   is a pointer to an internal array containing the string.  */
extern char *inet_ntoa (struct in_addr __in) __THROW;
```

此函数把网络字节序整数型IP地址转换成字符串形式。返回地址位char指针。返回字符串地址意味着字符串已保存到内部空间，但该函数未向程序员要求分配内存，而是在内部申请了内存并保存了字符串。也就是说调用完该函数后，应立即字符串信息复制到其他内存空间。因为，若再次调用inet_ntoa,则有可能覆盖之前保存的字符串信息。

```c
memset(&serv_addr, 0, sizeof(serv_addr));
```

serv_addr所有字节都初始化为0，这么做是为了把sockaddr_in结构体成员sin_zero初始化为0，强转成sockaddr。

`htonl(INADDR_ANY)` 自动获取服务器端IP地址。

### 向套接字分配网络地址（bind)

```c
#include <sys/socket.h>
int bind(int sockfd, struct sockaddr * myaddr, socklen_t addrlen);
```

sockfd 要分配地址信息(IP地址和端口号) 的套接字文件描述符。

myaddr 存有地址信息的结构体变量值

addrlen 第二个结构体长度

如果此函数调用成功，则将第二个参数指定的地址信息分配给第一个参数中的相应套接字。

## 基于TCP的服务器端

TCP套接字是面向连接的,因此又称为基于流(stream)的套接字

TCP(Transmission Control
Protocol)传输控制协议。意为“对数据传输过程的控制”，因此学习控制方法及范围有助于正确理解TCP套接字。

![tcp_ip_protocol](/images/Socket/tcp_ip_protocol.png)

TCP/IP协议共分4层，可以理解为数据收发分成了4个层次化过程。各层可能通过操作系统软件实现，也可能通过类似NIC的硬件设备实现。

### 链路层

链路层是物理链接领域标准化的结果，也是最基本的领域，专门LAN， WAN等网络标准。

### IP层

准备好物理链接后就要传输数据。为了在复杂的网络中传输数据，首先要考虑路径的选择。向目标传输数据需要经过那条路径。解决此问题就是IP层，该层使用的协议就是IP。

IP本身是面向消息的，不可靠的协议。每次传输数据时会帮我们选择路径，但并不一致。如果传输中发生路径错误，则选择其他路径;但如果发生数据丢失或错误，则无法解决。换言之，IP协议无法应对数据错误。

### TCP/UDP层

IP层解决数据传输中的路径选择问题，只需照此路径传输数据即可。TCP和UDP层以IP层提供的路径信息为基础完成实际的数据传输，故该层又称为传输层。TCP可以保证可靠的数据传输，但它发送的数据是以IP层为基础（这也是协议栈结构层次化的原因）。

IP层只关注1个数据包（数据传输的基本单位）的传输过程。因此，即使传输多个数据包，每个数据包也是由IP层实际传输的，也即使说传输顺序和传输本身是不可靠的。若只利用IP层传输数据，则有可能导致后传输的包B比先传输的包A提早到达。另外传输的数据包ABC中可能只收到AC，甚至收到的C可能已经损毁。如果数据交换过程中可以确认对方已收到数据，并重传丢失的数据，那么即使IP层不保证数据传输，这类通信也是可靠的。总之TCP和UDP存在IP层之上，决定主机之间的数据数据传输方式，TCP协议确认后向不可靠的IP协议赋予可靠性。

### 应用层

上述内容是套接字通信过程中自动处理的。选择数据传输路径，数据确认过程都被隐藏到套接字内部。编写软件过程中，需要根据程序特点决定服务器端和客户端之间的数据传输规则（规定），这便是应用层协议。网络编程大部分内容就是设计并实现应用层协议。

![involke](/images/Socket/involke.png)

### 进入等待连接请求状态 listen

```c
#inclue <sys/scoket.h>

int listen(int sock, int backlog);
```

sock
希望进入等待请求连接的套接字描述符，传递的描述符套接字参数称为服务器端套接字（监听套接字）。

backlog
连接请求等待队列(Queue)的长度，若为5，则队列为5，表示最多使5个连接请求进入队列。

已经调用了bind函数给套接字分配了地址，接下来调用listen进入**等待连接请求状态**。只有调用了listen函数，客户端才能进入**可发出连接请求的状态。**
换言之，这时候客户端才能调用connect()。

等待请求状态：客户端请求连接时，受理连接前一直使请求处于等待状态。

由此可知listen函数第一个参数传递的文件描述符套接字的用途。客户端连接请求本身也是从网络中接收到的一种数据，而想要接收就需要套接字。此任务就由服务端套接字完成。服务端套接字是接收请求的一名门卫或一扇门。准备好服务端套接字和请求连接等待队列后，这种可接收连接请求的状态称为等待连接请求状态。

### 受理客户端连接请求 accept

调用listen函数后，若有新的连接请求，则应按序受理。受理请求意味着可接受数据的状态。服务端套接字是做门卫的，因此需要另外一个套接字，但没必要亲自创建，accept函数将自动创建套接字，并连接到发起请求的客户端。

```c
#include <sys/socket.h>
int accept(int sock, struct sockaddr * addr, socklen_t * addrlen);
```

sock 服务端套接字的文件描述符

addr
保存发起请求的客户端地址信息的变量地址，调用函数后向传递来的地址变量参数填充客户端地址信息

addrlen
第二个参数addr结构体的长度，但是存有长度的变量地址。函数调用完成后，该变量即被填入客户端地址长度。

accept受理连接请求队列中待处理的客户端连接请求。函数调用成功时，accept函数内部将产生用于数据
I/O
的套接字，并返回其文件描述符。需要强调的是，套接字是自动创建的，并自动与发起请求的客户端建立连接。

## TCP客户端

与服务端相比，区别在于请求连接，他是创建客户端套接字后向服务端发起的连接请求。服务端调用listen函数后创建连接请求等待队列，之后客户端即可请求连接。

```c
#include <sys/socket.h>

int connect(int sock, struct sockaddr * servaddr, socklen_t addrlen);
```

sock 客户端套接字文件描述符

servaddr 保存目标服务器地址信息的变量地址值

addrlen 以字节为单位，传递已传递给第二个结构体参数servaddr的地址变量长度。

客户端调用connect函数后，发生下面情况才会返回。

服务端接收连接请求

发生断网等异常情况而中断连接请求。

需要注意，所谓“接收连接”并不意味着服务器端调用accept函数，其实是服务器端把连接信息记录到等待队列。因此connect函数返回后并不立即进行数据交换。

网络数据交换必须分配IP和端口，客户端套接字信息在
调用connect时自动分配，无需调用bind函数进行分配。

总结如下

服务端创建套接字后连续调用bind
listenh函数进入等待状态，客户端通过调用connect函数发起请求。需要注意的时客户端只能等到服务端调用listen后才能调connect函数。同时要清楚客户端connect之前，服务端可能率先调用accept函数，此时服务端在调用accept函数时进入阻塞状态，

直到客户端调用connect函数为止。

### TCP原理

TCP套接字中的IO缓冲

write函数调用瞬间，数据将移至输出缓冲，read函数调用瞬间，从输入缓冲读取数据。

i/o缓冲特性可整理如下:

- i/o缓冲在每个tcp套接字中单独存在。
- i/o缓冲在创建套接字时自动生成。
- 即使关闭套接字也会继续输出缓冲中遗留的数据。
- 关闭套接字将丢失输入缓冲中的数据。

### TCP内部工作原理1: 与对方套接字的连接

- 与对方套接字建立连接
- 与对方套接字进行数据交换
- 断开与对方套接字的连接

三次握手，套接字使以全双工方式工作的。也就是说他可以双向传递数据。

```
[SYN] SEQ: 1000, ACK:-
[SYN+ACK] SEQ:2000, ACK:1001
[ACK]SEQ:1001, ACK:2001
ack应答消息
seq数据包序号
```

### TCP内部工作原理2: 与对方主机的数据交换

### TCP内部工作原理3: 断开与套接字的连接 107页

四次挥手

## 基于UDP的服务端客户端

信件与udp特性完全相符。信件天填好收信人和收信人地址，信件的特性无法确认对方是否收到。

UDP在结构上比TCP简洁。不会发送类似ack的应答消息，也不会像seq那样给数据包分配序号。

流控制是UDP和TCP最重要的标志。TCP的生命在于流控制。

## 优雅地断开套接字连接

### 基于TCP的半关闭

断开一部分连接指的是，可以传输数据但无法接收，或可以接收数据但无法传输。顾名思义只关闭流的一半。

### 套接字和流

两台主机通过套接字建立连接后进入可交换数据的状态，又称“流形成状态”。也就是把建立套接字后可交换数据的状态看作一种流。

此处的流可以比作水流。水朝一个方向流动，同样在套接字的流中，数据也只能像向一个方向流动。因此为了进行双向通信，需要2个流。

一旦两台主机建立了套接字连接，每个主机就会拥有单独的输入流和输出流。其中一个主机的输入流与另一台主机的输出流相连，而输出流与另一主机的输入流相连。优雅的断开连接方式只断开其中1个流，而非同时断开两个流。

```c
#include <sys/socket.h>
int shutdown(int sock, int howto);
```

sock 需要断开的套接字文件描述符

howto 传递断开方式信息 `SHUT_RD` 断开输入流 `SHUT_WR` 断开输入流
`SHUT_RDWR`：同时断开IO流

断开输入流，套接字无法接收数据，即使输入缓冲收到数据也会抹去，而且无法调用输入相关函数。

中断输入流，无法传输数据，但如果输出缓冲还留有未传输的数据，则将传递至目标主机。

shutdown函数只关闭服务器输出流，这样既可以发送EOF同时保留了输入流，可以接收对方数据。

### 利用域名获取IP地址

```c
#include <netdb.h>
struct hostent * gethostbyname(const char * hostname);

struct hostent {
	char * h_name; // official name
	char ** h_aliases; // alias list
	int h_addrtype;    // host address type
	int h_length;      // address length
	char ** h_addr_lsit; // address list
}
```

### 利用IP地址获取域名

```c
#include <netdb.h>

struct hostent * gethostbyaddr(const char * addr, socklen_t len, int family);
// len ipv4 是 4 ipv6 是16
```

## 套接字可选项和IO缓冲大小

getsockopt & setsockopt 对可选项进行读取和设置

SO_RECVBUF
是输入缓冲大小相关的配置，SO_SNDBUF是输出缓冲大小相关的可选项。用这两个可选项既可以读取当前IO缓冲大小，也可以进行更改。

### SO_REUSEADDR

通常是客户端通知服务器端终止程序，客户端调用close函数，向服务器端发送FIN消息并经过四次握手过程。

服务器端向客户端发送FIN消息是，重新运行服务器将产生问题。

### Time-wait状态

套接字经过四次握手后并非立即消除，而是经过一段时间的timewait状态。只有先断开连接（先发送FIN消息的）主机才经过Time
wait状态。服务器先断开，则无法立即重新运行，套接字正处在time-wait过程中，相应端口正在使用。不管是服务端还是客户端都会有timewait过程。先断开连接的套接字必然会经过time-wait过程。但无需考虑客户端，因为客户端的端口是任意指定的。与服务器端不同的是，客户端每次运行程序时都会动态分配端口号。

如果没有timewait
主机A向B传输ACK消息后立即消除套接字，但最后的ACK消息在传递途中丢失，未能传给主机B.主机B会认为之前自己发送的FIN消息未能抵达主机A，继而试图重传。但此时主机A已经完全终止的状态，
因此主机B永远无法收到从主机A最后传来的ACK消息。相反，若主机A的套接字处于TIME-wait状态，则会向主机B重传最后的ACK消息，主机B也可以正常终止。基于这些考虑

先传输FIN消息的主机应经过timewait过程。

将套接字中SO_REUSEADDR
的值由默认0改成1可将timewait状态下套接字端口号重新分配给新的套接字。编程可随时运行的状态。

### TCP_NODELAY

Nagle算法，只有收到前一数据的ACK消息时，Nagle算法才发送下一数据。默认开启，不使用Nagle算法此时收发过程与ack接收与否无关，因此数据到达输出缓冲后将立即被发送出去，
将对网络流量产生负面影响。为了提高网络传输效率，必须使用Nagle算法。传输大文件数据时，即使不使用nagle算法，也会在装满输出缓冲时传输数据包。这不仅不会增加数据包的数量，反而在无需等待ACK的前提下连续传输，因此可以大大提高传输数据。

Nagle算法使用预付在网络流量下差别不大，使用Nagle算法的传输速度更慢。通过tcp_nodelay改成1禁用nagle算法。

# 多进程服务器端

## 并发服务器端的实现方法

- 多进程服务器：通过创建多个进程提供服务。
- 多路复用服务器： 通过捆绑并统一管理IO对象提供服务。
- 多线程服务器：通过生成与客户端等量的线程提供服务。

## 多进程服务器

理解进程，其定义如下：占用内存空间的正在运行的程序，从操作系统的角度看，进程是程序流的基本单位，若创建多个进程，则操作系统将同时运行。

```c
#include <unistd.h>
pid_t fork(void);
```

fork函数将创建调用的进程副本，也就是说，并非根据完全不同的程序创建进程，而是复制正在运行的，调用fork函数的进程。另外，两个进程都将执行fork函数调用后的语句（准确的说是在fork函数后返回）。但是因为通过同一个进程，复制相同的内存空间，之后的程序流要根据fork函数的返回值加以区分。即利用fork函数的如下特点区分程序执行流程。

- 父进程： fork函数返回子进程ID
- 子进程：fork函数返回0

此处父进程是指原进程，即调用fork函数的主题，而子进程是通过父进程调用fork函数复制出的进程。

### 产生僵尸进程的原因

- 传递参数并调用exit函数
- main函数中执行return语句并返回值

向exit函数传递参数值和main函数的return语句返回的值都会传递给操作系统。而操作系统不会销毁子进程，直到把这些值传递给产生该子进程的父进程。处于这种状态下的进程就是僵尸进程。也就是，将子进程编程僵尸进程的正是操作系统。既然如此，此僵尸进程何时被销毁呢？其实以

### 利用wait函数

```c
#include <sys/wait.h>
pid_t wait(int *statloc); //成功的返回终止的子进程ID， 失败时返回-1.
// WIFEXITED 子进程正常终止时返回真（true)
// WEXITSTATUS 返回子进程的返回值
```

### 使用waitpid函数

```c
#include <sys/wait.h>
pid_t waitpid(pid_t pid, int * statloc, int options); // 成功时返回终止的子进程ID(或0),失败时返回-1
```

pid 等待终止的目标子进程ID，若传递-1，则与wait函数相同，可以等待任意子进程终止。

statloc 与wait函数的statloc参数觉由相同含义

options 传递头文件sys/wait.h
中声明的常量WNOHANG，即使没有终止的子进程也不会进入阻塞状态，而是返回0并退出函数。

### 信号与signal函数

进程发现自己的子进程结束时，请求操作协同调用特性函数。该请求通过信号注册函数完成。

```c
#include <signal.h>
void (*signal(int signo, void (*func)(int)))(int); // 返回值为函数指针
void (*signal())(int); // signal是一个返回值为函数指针的函数
```

- 函数名： signal
- 参数 int signo. void(* func)(int)
- 返回类型： 参数为int型，返回void型函数指针

调用上述函数时，第一个参数为特殊情况信息，第二个参数为特殊情况下将要调用的函数的地址值（指针）。发生第一个参数代表的情况时，调用第二个
参数所指的函数。signal函数中注册的部分特殊情况和对应的常数。

**发生信号将唤醒由于调用sleep函数而进入阻塞状态的进程**

调用函数的主体的确是操作系统，但进程处于睡眠状态时无法调用函数。因此，产生信号时，为了调用信号处理器，将唤醒由于调用sleep函数而进入阻塞状态的进程。而且，进程一旦被唤醒就不会在进入睡眠状态。即使未到sleep函数中规定的时间也是如此。

### 利用sigaction函数进行信号处理

sigaction函数类似于signal函数，而且完全可以替代后者，也更稳定。之所以稳定是因为，signal函数在unix系列不同的操作系统中可能存在区别，但sigaction函数完全相同。

```c
#include <signal.h>
int sigaction(int signo, const struct sigaction *act, struct sigaction * oldact); 
// succee 0, faild -1

struct sigaction{
	void (*sa_handler)(int); // 保存信号处理函数的指针值
	sigset_t sa_mask; // 初始化为0即可
	int sa_flags; // 0
}
```

### 基于多任务的并发服务器

- 第一阶段： 回声服务器端（父进程）通过调用accept函数受理连接请求
- 第二阶段： 此时获取的套接字文件描述符创建并传递给子进程
- 第三阶段： 子进程利用传递过来的文件描述符提供服务。

因为子进程会复制父进程拥有的所有资源，实际上根本不用另外经过传递文件描述符的过程。

### 通过fork函数复制文件描述符

fork函数将2个套接字文件描述符复制给了子进程。不会复制套接字，严格意义上来说，套接字属于操作系统。进程拥有代表套接字的文件描述符。1个套接字存在2个文件描述符，只有2个文件描述符都终止（销毁）后，才能销毁套接字。

### 分割TCP的IO程序

分割IO程序的优点

客户端父进程负责接收数据，额外创建的子进程负责发送数据。分割后，不同进程分别负责输入和输出，这样，无论客户端是否从服务器端接收完数据都可以进行传输。分割IO程序的另一个好处是，提议提高频繁交换数据的程序性能。分割IO后的客户端发送数据时不必考虑接收数据的情况，因此可以连续发送数据，由此提高同一时间内传输的数据量。

## 进程间通信

进程间通信意味着两个不同进程间可以交换数据，为了完成这一点，操作系统应提供两个进程可以同时访问的内存空间。

### 通过管道实现进程间通信

管道并非属于进程的资源，而是和套接字一样，属于操作系统(也就不是fork函数的复制对象)。所以操作系统提供的内存空间进行通信。

```c
#include <unistd.h>
int pipe(int filedes[2]); // 成功返回0， 失败返回-1
// filedes[0] 通过管道接收数据时使用的文件描述符，即管道出口
// filedes[1] 通过管道传输数据时使用的文件描述符，即管道入口
```

父进程调用该函数时将创建管道，同时获取对应于出入口的文件描述符，通过fork将这两个文件描述符传递给子进程。

向管道传递数据时，先读的进程会把数据取走。数据进入管道后成为无主数据。也就是通过read函数先读取数据的进程将得到数据，即使该进程将数据传到了管道。

# 基于IO复用的服务器端

IO复用服务端的进程需要确认举手（收到数据）的套接字，病故过举手的套接字接收数据。

### select函数的功能和调用顺序

使用select函数时可以将多个文件描述符集中在一起统一监视，项目如下。

- 是否存在套接字接收数据？
- 无需阻塞传输数据的套接字有那些？
- 那些套接字发生了异常

select函数时IO复用的全部内容。

select函数的调用方法和顺序如下

**步骤一：**

设定文件描述符

指定监视范围

设置超时

**步骤二**

调用select函数

**步骤三**

查看调用结果

### 设置文件描述符

利用select函数可以同时监视多个文件描述符。当然，监视文件描述符可以视为监视套接字。此时首先需要将要监视的文件描述符集中在一起。集中时也要按照监视项（接收，传输，异常）进行区分，即按照上述三种监视项分成3类。使用fd_set数组变量执行此项操作。该数组是存有0和1的位数组。

![file_description](/images/Socket/file_description.png)

最左端的位表示文件描述符0（所在位置）。如果该位设置为1，则表示该文件描述符时监视对戏那个。（1，3是监视对象）。

在fd_set变量中注册和改变值的操作都由下列宏完成

- `FD_ZERO(fd_set *fdset)` :将fd_set变量的所有位初始化位0.
- `FD_SET(int fd, fd_set * fdset)` :
  在参数fdset指向的变量中注册文件描述符fd的信息。
- `FD_CLR(int fd, fd_set *fdset)` ：
  从参数fdset指向的变量中清除文件描述符fd的信息。
- `FD_ISSET(int fd, fd_set *fdest)` :
  从参数fdset指向的变量中包含文件描述符fd的信息，则返回“真”。

```c
#include <sys/select.h>
#include <sys/time.h>

int select(int maxfd, fd_set * readset, fd_set *writeset, fd_set *expectset,
					const struct timeval * timeout);// 成功时返回大于0的值，失败时返回-1.
```

- maxfd 监视对象文件描述符数量。
- readset
  将所有关注“是否存在待读取数据”的文件描述符注册到fd_set型变量，并传递其地址值。
- writeset
  将所有关注“是否可传输无阻塞数据”的文件描述符注册到fd_set型变量，并传递其地址值。
- exceptset 将关注“是否发生异常”的文件描述符注册到fd_set型变量，并传递其地址值。
- timeout 调用select函数后，为防止陷入无线阻塞的状态，传递超时(time-out)信息。
- 返回值
  发生错误时返回-1，超时返回时返回0。因发生关注的事件返回时，返回大于0的值，该值时发生事件的文件描述符。

如上所述，select函数用来验证3种监视项的变化情况。根据监视项声明3个fd_set型变量。分别向其注册文件描述符信息。但在此之前（调用select函数前）需要决定下面2件事。

“文件描述符的监视（检查）范围是？”

“如何设定select函数的超时时间？”

第一，只需将最大的文件描述符加1在传递到select函数即可（文件描述符从0开始）

本来select函数只在监视的文件描述符发生变化时才返回。如果未发生变化，机会进入阻塞状态。制定超时时间就是为了防止这种情况发生。将秒数填入tv_sec成员，将毫秒数填入tv_usec成员。即使文件描述符未发生变化，只要过了指定时间，也可以从函数返回。这种情况下返回0.不想设置超时则传递NULL。

![select](/images/Socket/select.png)

select函数调用完成后，向其传递的fd_set变量中将发生变化。原来为1的所有位均变成0，但发生变化的文件描述符位除外。因此，可以认为值仍为1的位置上的文件描述符发生了变化。

```c
#include <stdio.h>
#include <sys/select.h>
#include <sys/time.h>
#include <unistd.h>
#define BUF_SIZE 30

int main(void) {
  fd_set reads, temps;
  int result, str_len;
  char buf[BUF_SIZE];
  struct timeval timeout;

  FD_ZERO(&reads);  // 初始化fd_set变量

  //将文件描述符0对应的为设置位1.换言之，需要监视标准输入的变化
  FD_SET(0, &reads);  // 0 is standard input(console)

  //不能在此时设置超时。因为调用select函数后，
  //结构体timeval的成员tv_sec和tv_usec的值将被替换为超时前剩余时间。
  //因此，调用select函数前每次都要初始化timeval结构体变量
  /*
  timeout.tv_sec = 5;
  timeout.tv_usec = 5000;
  */

  while (1) {
    //将准备好的fd_set变量reads的内容复制到temps变量，
    //调用select函数后，除发生变化的文件描述符对应位外，剩下的所有位将初始化位0.
    //因此为了记住初始值，必须经过这种复制。这是使用select函数的通用方法。
    temps = reads;
    timeout.tv_sec = 5;
    timeout.tv_usec = 0;
    //调用select函数。如果控制台有输入数据，则返回大于0的整数；
    //如果没有输入数据而引发超时，则返回0
    result = select(1, &temps, 0, 0, &timeout);
    if (result == -1) {
      puts("select() error!");
      break;
    } else if (result == 0) {
      puts("Time-out!");
    } else {
      //验证发生变化的文件描述符是否为标准输入。
      //若是，则从标准输入读取数据并向控制台输出。
      if (FD_ISSET(0, &temps)) {
        str_len = read(0, buf, BUF_SIZE);
        buf[str_len] = 0;
        printf("message from console: %s", buf);
      }
    }
  }
  return 0;
}
```
