---
title: "「TCP IP网络编程」读书笔记2-基于Linux的编程"
date: 2023-02-05T16:03:34+08:00
tags: ["c", "socket"]
categories: ["阅读"]
draft: false
---

## 回声服务器代码

```cpp
#include <arpa/inet.h>
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/select.h>
#include <sys/socket.h>
#include <sys/time.h>
#include <unistd.h>

#define BUF_SIZE 100

void error_handling(char *message);

int main(int argc, char *argv[]) {
  int serv_sock, clnt_sock;
  struct sockaddr_in serv_adr, clnt_adr;
  struct timeval timeout;
  fd_set reads, cpy_reads;

  socklen_t adr_sz;
  int fd_max, str_len, fd_num, i;

  char buf[BUF_SIZE];

  if (argc != 2) {
    printf("Usage :%s <port>\n", argv[0]);
    exit(1);
  }

  serv_sock = socket(PF_INET, SOCK_STREAM, 0);

  memset(&serv_adr, 0, sizeof(serv_adr));
  serv_adr.sin_family = AF_INET;
  serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
  serv_adr.sin_port = htons(atoi(argv[1]));

  if (bind(serv_sock, (struct sockaddr *)&serv_adr, sizeof(serv_adr)) == -1) error_handling("bind() error");

  if (listen(serv_sock, 5) == -1) error_handling("listen() error");

  // 向要传到select函数第二个参数的fd_set变量reads注册服务器套接字。
  // 这样，接收数据情况的监视对象就包含了服务器端套接字。
  // 客户端的连接同样通过传输数据完成。
  // 因此服务器端套接字中有接收的数据，就意味着有新的连接请求。
  FD_ZERO(&reads);
  FD_SET(serv_sock, &reads);
  fd_max = serv_sock;

  while (1) {
    cpy_reads = reads;
    timeout.tv_sec = 5;
    timeout.tv_usec = 5000;

    // 第三，四参数为空，只需要根据目的传递必要的参数
    if ((fd_num = select(fd_max + 1, &cpy_reads, 0, 0, &timeout)) == -1) break;

    if (fd_num == 0) continue;

    for (i = 0; i < fd_max + 1; i++) {
      // 查找发生状态变化的（有接收数据的套接字的）文件描述符
      if (FD_ISSET(i, &cpy_reads)) {
        //发生状态变化时，首先验证服务器端套接字中是否有变化。
        //如果服务器端套接字的变化，将受理连接请求。
        if (i == serv_sock) {  // connect request!
          adr_sz = sizeof(clnt_adr);
          clnt_sock = accept(serv_sock, (struct sockaddr *)&clnt_adr, &adr_sz);

          // fd_set变量reads中注册了与客户端连接的套接字文件描述符
          FD_SET(clnt_sock, &reads);
          if (fd_max < clnt_sock) fd_max = clnt_sock;
          printf("connected client : %d \n", clnt_sock);

          // 发生变化的套接字并非服务器端套接字时，即有要接收的数据时执行else语句
        } else {  // read message !
          str_len = read(i, buf, BUF_SIZE);
          // 接收的数据为EOF时需要关闭套接字，并从reads中删除相应信息。
          if (str_len == 0) {  // close request !
            FD_CLR(i, &reads);
            close(i);
            printf("close client : %d \n", i);
          } else {
            // 接收的数据为字符串时，执行回声服务。
            write(i, buf, str_len);  // echo!
          }
        }
      }
    }
  }
  close(serv_sock);
  return 0;
}

void error_handling(char *message) {
  fputs(message, stderr);
  fputc('\n', stderr);
  exit(1);
}
```

## 第十三章 多种IO函数

### Linux中的send & recv

```c
#include <sys/socket.h>
ssize_t send(int sockfd, const void * buf, size_t nbytes, int flags); 
// 成功时返回发送的字节数，失败时返回 -1
```

- sockfd 表示与数据传输对象的连接的套接字文件描述符。
- buf 表示待传输数据的缓冲地址值
- nbtyes 待传输的字节数
- flags 传输数据时指定的可选项信息

#### recv 函数

```c
#include <sys/socket.h>
ssize_t recv(int sockfd, void * buf, size_t nbytes, int flags)
// 成功时返回接收的字节数（收到EOF时返回0）， 失败时返回-1？
```

- sockfd 表示数据接收对象的连接的套接字文件描述符。
- buf 保存接收数据的缓冲地址值
- nbtyes 可接收的最大字节数
- flags 接收数据时指定的可选项信息

| 可选项        | 含义                                                      | send | recv |
| ------------- | --------------------------------------------------------- | ---- | ---- |
| MSG_OOB       | 用于传输带外数据(out-of-band data)                        | 是   | 是   |
| MSG_PEEK      | 验证输入缓冲是否存在接收数据                              | 否   | 是   |
| MSG_DONTROUTE | 数据传输过程中不参照路由(Routing)表，在本地网络寻找目的地 | 是   | 否   |
| MSG_DONTWAIT  | 调用IO函数时不阻塞，用于使用非阻塞IO                      | 是   | 是   |
| MSG_WAITALL   | 防止函数返回，直到接收全部请求的字节数                    | 否   | 是   |

MSG_OOB 发送紧急消息

MSG_OOB 可选项用于创建特殊发送方法和通道以发送紧急消息。收到 MSG_OOB
紧急消息时操作系统将产生SIGURG信号，并调用注册的信号处理函数。

`fcntl(recv_sock, F_SETOWN, getpid());`

fcntl函数用于控制文件描述符，上述调用语句含义如下：

**将文件描述符recv_sock指向的套接字拥有者(F_SETOWN)改为把getpid函数返回值用作ID的进程**。

操作系统实际创建并管理套接字，所以从严格意义上说，套接字拥有者是操作系统。只是此处所谓的拥有者是指负责套接字所有事物的主体。上述描述简要概括如下：

**文件描述符recv_sock指向的套接字引发的SIGURG信号处理进程变为将getpid函数返回值用作ID的进程。**

通过MSG_OOB可选项不会加快数据传输速度，而且通过信号处理函数
urg_handler读取数据时也只能读1个字节。剩余数据只能通过未设置MSG_OOB可选项的普通输入函数读取。这是因为TCP不存在真正意义上的带外数据。MSG_OOB中的OOB指定是
out-of-band ,而带外数据的含义是：

**通过完全不同的通信路径传输的数据。**

即真正意义上的out-of-band需要通过单独的通信路径高速传输数据，但TCP不另外提供，只利用TCP的紧急模式（Urget
mode)进行传输。

MSG_OOB的真正意义在于督促接收数据接收对象尽快处理数据。这是紧急模式的全部内容，而且TCP保持传输顺序的传输特性依然成立。

#### 检查输入缓冲

同时设置MSG_PEEK选项和MSG_DONTWAIT选项，以验证输入缓冲中是否存在接收的数据。设置MSG_PEEK选项并调用recv函数时，即使读取了输入缓冲的数据也不会删除。因此，该选项通常与MSG_DONTWAIT合作，用于调用非阻塞方式验证待读数据存在与否的函数。

```c
#include <arpa/inet.h>
#include <fcntl.h>
#include <netinet/in.h>
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>

#define BUF_SIZE 30
void error_handling(char *message);

int main(int argc, char *argv[]) {
  int acpt_sock, recv_sock;
  struct sockaddr_in acpt_adr, recv_adr;
  int str_len;
  socklen_t recv_adr_sz;
  char buf[BUF_SIZE];
  if (argc != 2) {
    printf("Usage : %s <port> \n", argv[0]);
    exit(1);
  }

  acpt_sock = socket(PF_INET, SOCK_STREAM, 0);
  memset(&acpt_adr, 0, sizeof(acpt_adr));
  acpt_adr.sin_family = AF_INET;
  acpt_adr.sin_addr.s_addr = htonl(INADDR_ANY);
  acpt_adr.sin_port = htons(atoi(argv[1]));

  if (bind(acpt_sock, (struct sockaddr *)&acpt_adr, sizeof(acpt_adr)) == -1) error_handling("bind() error!");
  if (listen(acpt_sock, 5) == -1) error_handling("listen() error!");

  recv_adr_sz = sizeof(recv_adr);
  recv_sock = accept(acpt_sock, (struct sockaddr *)&recv_adr, &recv_adr_sz);

  while (1) {
    //调用recv函数的同时传递MSG_PEEK可选项，这是为了保证即使不存在待读取数据也不会进入阻塞状态
    str_len = recv(recv_sock, buf, sizeof(buf) - 1, MSG_PEEK | MSG_DONTWAIT);
    if (str_len > 0) break;
  }

  buf[str_len] = 0;
  printf("Buffering %d bytes : %s \n", str_len, buf);

  // 再次调用recv函数。这次并非设置任何可选项，因此，本次读取的数据将从缓冲中删除。
  str_len = recv(recv_sock, buf, sizeof(buf) - 1, MSG_PEEK | MSG_DONTWAIT);
  buf[str_len] = 0;
  printf("Read again : %s \n", buf);

  close(recv_sock);
  close(acpt_sock);
  return 0;
}

void error_handling(char *message) {
  fputs(message, stderr);
  fputc('\n', stderr);
  exit(1);
}
```

#### readv & writev 函数

readv & writev 函数的功能可概括为：对数据进行整合传输及发送的函数。

也就是说，通过writev函数可以将分散保存在多个缓冲中的数据一并发送，通过readv函数可以由多个缓冲分别接收。因此，适当使用这两个函数可以减少IO函数的调用次数。

```c
#include <sys/uio.h>
ssize_t writev(int filedes, const strut iovec * iov, int iovcnt);
//成功时返回发送的字节数，失败时返回-1

struct iovec {
	void * iov_base; //缓冲地址
	size_t iov_len; // 缓冲大小
}
```

- filedes
  表示传输对象的套接字文件描述符。但该函数并不只限于套接字，因此，可以向read函数一样向其传递文件或标准输出描述符。
- iov iovec结构体数组的地址值，结构体iovec中包含待发送数据的位置和大小信息。
- iovcnt 向第二个参数传递的数据长度

结构体iovec
由保存待发送数据的缓冲（char型数组）地址值和实际发送的数据长度信息构成。

`writev(1, ptr, 2);`

writev第一个参数1是文件描述符，因此向控制台输出数据，ptr是存有待发送数据信息的iovec数组指针。第三个参数是2，因此，从ptr指向的地址开始，共浏览2个iovec结构体变量，发送这些指针指向的缓冲地址。

```c
#include <sys/uio.h>
ssize_t readv(int filedes, const struct iovec * iov, int iovcnt);
// 成功时返回接收的字节数，失败时返回-1。
```

- filedes 传递接收数据的文件（或套接字）描述符
- iov 包含数据保存位置和大小信息的iovec结构体数组的地址值。
- iovcnt 第二个参数中数组的长度。

## 第十五章 套接字和标准IO

标准IO函数的两个优点

- 良好的移植性
- 利用缓冲提高性能

使用标准IO函数传输数据是，经过两个缓冲。首先将数据传递到标准IO函数的缓冲，然后数据移动到套接字输出缓冲，最后将字符串发送到对方主机。

设置缓冲的主要目的是为了提高性能，但套接字中的缓冲主要是为了TCP协议而设立的。例如，TCP传输中丢失数据时将再次传递，而再次发送数据则意味着在某地保存了数据。存在什么地方呢？套接字的输出缓冲。与之相反，使用标准IO函数缓冲的主要目的是为了提高性能。

### 利用fdopen函数转换为FILE结构体指针

可以通过fdopen函数将创建套接字时返回的文件描述符转换成标准IO函数中使用的FILE结构体指针。

```c
#include <stdio.h>
FILE * fdopen(int fildes, const char * mode);
```

- fildes 需要转换的文件描述符
- mode 将要创建的FILE结构体指针的模式信息

### 利用fileno函数转换成文件描述符

```c
#include <stdio.h>
int fileno(FILE * stream);
```

## 第十六章 关于IO流分离的其内容

调用fopen函数打开文件后可以与文件交换数据，因此说调用fopen函数创建了流。此处流是指数据流动，但通常可以比喻为“以数据收发为目的的一种桥梁”。将流理解为数据收发路径。

### 2次IO流分离

第一种是调用fork函数复制出1个文件描述符，以区分输入和输出中使用的文件描述符。虽然文件描述符本身不会根据输入和输出流进行区分，但我们分开了2个文件描述符的用途，因此这也属于流的分离。

第二种，通过2次调用fdopen函数的调用。创建读模式FILE指针和写模式FILE指针。换言之，分离了输入工具和输出工具。因此也可视为流的分离。

### 分离流的好处

第一种里

- 通过分开输入过程（代码）和输出过程降低实现难度。
- 与输入无关的输出操作可以提高速度。

第二种里面

- 为了将FILE指针按读模式和写模式加以区分。
- 可以通过区分读写模式降低实现难度。
- 通过区分IO缓冲提高缓冲性能。

### 文件描述符的复制和半关闭

![dup](/images/Socket/dup.png)

读模式FILE指针和写模式FILE指针都是基于同一文件描述符创建的。因此，针对任意一个FILE指针调用fclose函数都会关闭文件描述符，也就终止套接字。销毁套接字时再也无法进行数据交换。那如何进入可以输入但无法输出的半关闭状态呢。很简单，**创建FILE指针前先复制文件描述符即可**。

![dup2](/images/Socket/dup2.png)

复制后另外创建1个文件描述符，然后利用各自的文件描述符生成读模式FILE指针和写模式FILE指针。**这就为半关闭准备好了环境**，因此套接字和文件描述符之间具有如下关系：

**“销毁所有文件描述符后才能销毁套接字。”**

也就是说，针对写模式FILE指针调用fclose函数时，只能销毁与该FILE指针相关的文件描述符，无法销毁套接字。调用fclose函数后还剩1个文件描述符，因此没有销毁套接字。那次时是否为半关闭状态？不是，只是准备好了半关闭环境。要进入真正的半关闭状态需要特殊处理。因为还剩1个文件描述符，而且该文件描述符可以同时进行IO。因此，不但没有发送EOF，而且仍然可以利用文件描述符进行输出。

### 复制文件描述符

调用fork函数时将复制整个进程，因此同一进程内不能同时有原件和副本。这里讨论的的复制并非针对整个进程，而是在同一进程内完成描述符的复制。

![fd_dup](/images/Socket/fd_dup.png)

上图给出的是同一进程内存在2个文件描述符可以同时访问文件的情况。当然，文件描述符的值不能重复，因此各使用5和7的整数值。为了形成这种结构，需要复制文件描述符。此处所谓的“复制”具有如下含义：

“为了访问同一文件和套接字，创建另一个文件描述符。”

通常的复制很容易让人理解为将包含文件描述符整数值在内的所有内容进行复制，而次数的复制方式却不同。

### dup & dup2

```c
#include <unistd.h>
int dup(int fildes);
int dup2(int fildes, int fildes2);
//成功时返回复制的文件描述符，失败时返回-1
```

- fildes 需要复制的文件描述符。
- fildes2 明确指定的文件描述符整数值。

dup2函数明确指定复制的文件描述符整数值。向其传递大于0且小于进程能生成最大文件描述符值时，该值将称为复制出的文件描述符。

通过调用shutdown函数。服务器端进入半关闭状态，并向客户端发送EOF。**调用shutdown时，无论复制出多少文件描述符都进入半关闭状态，同时传递EOF。**

## 第十七章优于select的epoll

基于select的IO复用速度慢的原因

- 调用select函数后常见的针对所有文件描述符的循环语句
- 每次调用select函数时都需要向该函数传递监视对象信息

调用select函数后，并不是把发生变化的文件描述符单独集中在一起，而是通过观察作为监视对象的fd_set变量的变化，找出发生变化的文件描述符，因此无法避免针对所有监控对象的循环语句。而且，作为监视对象的fd_set变量会发生变化，所以调用select函数前应复制并保存原有信息，并且每次调用select函数时传递新的监视对象信息。

相比于循环语句，提高性能更大的障碍时每次传递监视对象信息。因为传递监视对象信息具有如下含义：

“**每次调用select函数时向操作系统传递监视对象信息。”**

应用程序向操作系统传递数据将对程序造成很大负担，而且无法通过优化代码解决，因此将成为性能上的致命弱点。select函数与文件描述符有关，更准确地说，是监视套接字变化的函数。而套接字是由操作系统管理的，所以select函数绝对需要借助于操作系统才能完成功能。select函数的这一缺点可以通过如下方式弥补：

“仅向操作系统传递1次监视对象，监视范围或内容发生变化时只通知发生变化的事项。”linux支持的方式是epoll,windows支持方式时IOCP。

### 实现epoll时必要的函数和结构体

能够克服select函数缺点的epoll函数具有如下优点，这些优点与之前的select函数缺点相反。

- 无需编写以监视状态变化为目的的针对所有描述符的循环语句。
- 调用对应于select函数的epoll_wait函数时无需每次传递监视对象信息。

epoll服务器端实现中需要的3个函数

- epoll_create: 创建保存epoll文件描述符的空间。
- epoll_ctl: 向空间注册并注销文件描述符。
- epoll_wait: 与select函数类似，等待文件描述符发生变化。

select方式中为了保存监视对象描述符，直接声明了fd_set变量。但epoll方式下由操作系统负责保存监视对象文件描述符，因此需要向操作系统请求创建保存文件描述符的空间，此时使用的函数就是epoll_create。

此外，为了添加和删除监视对象文件描述符，select方式中需要FD_SET， FD_CLR
函数。但在epoll方式中，通过epoll_ctl
函数请求操作系统完成。最后，select方式下调用select函数等待文件描述符的变化，而epoll中调用epoll_wait函数。还有，select方式中通过fd_set
变量查看对象的状态变化（事件发生与否），而epoll方式中通过如下结构体epoll_event将发生变化的（发生事件的）文件描述符单独集中到一起。

```c
typedef union epoll_data
{
  void *ptr;
  int fd;
  uint32_t u32;
  uint64_t u64;
} epoll_data_t;

struct epoll_event
{
  uint32_t events;	/* Epoll events */
  epoll_data_t data;	/* User data variable */
} __EPOLL_PACKED;
```

声明足够大的epoll_event结构体数组后，传递给epoll_wait函数时，发生变化的文件描述符信息被填入该数组。因此，无需像select函数那样针对所有文件描述符进行循环。

### epoll_create

调用epoll_create函数时创建的文件描述符保存空间成为“epoll例程”。通过参数size传递的值决定epoll例程的大小，但该值只是向操作系统提的建议。换言之，size并非用来决定epoll例程的大小，而仅供操作系统参考。

epoll_create函数创建的资源与套接字相同，也由操作系统管理。因此，该函数和创建套接字的情况相同，也会返回文件描述符。也就是说，该函数返回的文件描述符主要用与于区分epoll例程。需要终止时，与其他文件描述符相同，也要调用close函数。

### epoll_ctl

生成epoll例程后，应在其内部注册监视对象文件描述符，此时使用epoll_ctl函数。

```c
#include <sys/epoll.h>
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
//成功时返回0，失败时返回-1.
```

- epfd 用于注册监视对象的epoll例程的文件描述符。
- op 用于指定监视对象的添加，删除或更改等操作。
- fd 需要注册的监视对象描述符。
- event 监视的事件类型。

**epoll_event结构体用于保存发生事件的文件描述符集合。但也可以在epoll例程中注册文件描述符时，用于注册关注的事件。**

epoll_ctl 的常量及含义。

- EPOLL_CTL_ADD：将文件描述符注册到epoll例程。
- EPOLL_CTL_DEL: 从epoll例程中删除文件描述符， 第四个参数传递NULL。
- EPOLL_CTL_MOD: 更改注册的文件描述符的关注事件发生情况。

epoll_event的成员events中可以保存的常量及所指的事件类型。

- EPOLLIN：需要读取数据的情况。
- EPOLLOUT: 输出缓冲为空，可以立即发送的情况。
- EPOLLPRI: 收到OOB数据的情况。
- EPOLLRDHUP: 断开连接或半关闭的情况，这在边缘触发方式下非常有用。
- EPOLLERR: 发生错误的情况。
- EPOLLET: 以边缘触发的方式得到事件通知。
- EPOLLONESHOT:
  发生一次事件后，相应文件描述符不再收到事件通知。因此需要向epoll_ctl函数的第二个参数传递EPOLL_CTL_MOD,
  再次设置事件。

可以通过位或运算同时传递多个上述参数。

### epoll_wait

与select函数对应的epoll_wait函数，epoll相关函数中默认最后调用该函数。

```c
#include <sys/epoll.h>
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
// 成功时返回事件的文件描述符，失败时返回-1。
```

epfd 表示事件发生监视范围的epoll例程的文件描述符。

events 保存发生事件的文件描述符集合的结构体地址值。

maxevents 第二个参数中可以保存的最大事件数。

timtout 以1/1000秒为单位的等待时间，传递-1时，一直等待直到发生事件。

第二个参数所指缓冲需要动态分配。

```c
#include <sys/epoll.h>
int event_cnt;
struct epoll_event * ep_events;
ep_events = malloc(sizeof(struct epoll_event)*EPOLL_SIZE); // EPOLL_SIZE是宏常量
event_cnt = epoll_wait(epfd, ep_events, EPOLL_SZIE, -1);
```

调用函数后，返回发生事件的文件描述符数，同时在第二个参数所指向的缓冲中保存发生事件的文件描述符集合。因此，无需像select那样插入针对所有文件描述符的循环。

### 条件触发和边缘触发

只有正确区分条件触发(Level Trigger)和边缘触发(Edge Trigger)，才算完整掌握epoll.

**条件触发和边缘触发的区别在于发生事件的时间点**

条件触发方式中，只要输入缓冲有数据就会一直通知该事件。

例如，服务器端输入缓冲收到50字节的数据时，服务器端操作系统将通知该事件（注册到发生变化的文件描述符）。但服务器端读取20字节后还剩30字节的情况下，仍会注册事件。也就是说，条件触发方式中，只要输入缓冲还剩有数据，就会以事件方式再次注册。

边缘触发中输入缓冲收到数据时仅注册1次该事件。即使输入缓冲中还留有数据，也不会再进行注册。

select模式是以条件触发的方式工作的，输入缓冲中如果还剩有数据，肯定会注册事件。

### 边缘触发的服务器实现中必知的两点

- 通过errno变量验证错误原因。
- 为了完成非阻塞(Non-blocking)I/O, 更改套接字特性。

为了访问该变量，需要引入error.h头文件，因为此头文件中有上述变量的extern声明。另外，每种函数发生错误时，保存到errno变量中的值都不同，没必要记住所有可能的值。

“read函数发现输入缓冲中没有数据可读时返回-1，同时在errno中保存EAGAIN常量。”

linux 提供更改或读取文件属性的方法。

```c
#include <fcntl.h>
int fcntl(int filedes, int cmd, ...);
```

- filedes 属性更改目标的文件描述符。
- cmd 表示函数调用的目的。

如果向第二个参数传递F_GETFL,
可以获得第一个参数所指的文件描述符属性(int型).反之，如果传递F_SETFL,可以更改文件描述符属性。若希望将文件（套接字）改成非阻塞模式。需要如下2条语句。

```c
int flag = fcntl(fd, F_GETFL,0);
fcntl(fd, F_SETFL, flag|O_NONBLOCK);
```

通过第一条语句获取之前设置的属性信息。通过第二条语句在此基础上添加非阻塞
O_NONBLOCK标志。调用 read & write
函数时，无论是否存在数据，都会形成非阻塞文件（套接字）。

之所以介绍读取错误原因的方法和非阻塞模式的套接字创建方法，原因在于二者都与边缘触发的服务端有密切联系。首先说明为何需要通过errno确认错误原因。

“边缘触发方式中，接收数据时仅注册1次该事件。”

就是因为这种特点，一旦发生输入相关事件，就应该读取输入缓冲中的全部数据。因此需要验证输入缓冲是否为空。

“read函数返回-1，变量errno中的值为EAGAIN时，说明没有数据可读。”

边缘触发方式下，以阻塞方式工作的read &
write函数有可能引起服务器端的长时间停顿。因此，边缘触发方式中一定要采用非阻塞
read & write 函数。

### 边缘触发和条件触发孰优孰劣

边缘触发方式可以做到如下这点：

“可以分离接收数据和处理数据的时间点！”

即使输入缓冲收到数据（注册相应事件），服务器端也能决定读取和处理这些数据的时间点，这样就给服务端实现带来了巨大的灵活性。

## 第十八章 多线程服务器端的实现

多进程模型的缺点可概括如下

- 创建进程的过程会带来一定的开销
- 为了完成进程中的数据交换，需要特殊的IPC技术。

相对于下面的缺点，上述2个缺点不算什么。

“每秒少则数十次，多则数千次的‘上下文切换’是创建进程时最大的开销。”

只有1个cpu的系统也可以同时运行多个进程。这是因为系统将cpu时间分为多个微小的块后分配给了多个进程。为了分时使用cpu，需要上下文切换的概念。

上下文切换：运行程序前需要将相应程序信息读入内存，如果运行进程A后需要紧接着运行进程B，就应该将进程A相关信息移出内存，并读入进程B相关信息。这就是上下文切换。但此时进程A的数据将被移动到硬盘，所以上下文切换需要很长时间。

线程优点：

- 线程创建和上下文切换比进程的创建和上下文切换更快。
- 线程间交换数据时无需特殊技术。

### 线程和进程的差异

每个进程的内存空间都由保存全局变量的数据区，向malloc等函数的动态分配提供空间的堆，函数运行时使用的栈构成。

线程：

- 上下文切换时不需要切换数据区和堆
- 可以利用数据区和堆交换数据
