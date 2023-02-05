---
title: "gdb的使用"
date: 2023-02-05T19:41:00+08:00
tags: ["gdb"]
categories: ["Linux"]
draft: false
---

gcc -g 选项生成带有调试符号信息

strip命令用于移除某个程序调试信息。

优化等级 o0~o4

使用：

gcc -O0 -g hello.c -o hello

## 启动gdb调试的方法

使用gdb调试一个程序一般有三中情况：

1. gdb _filename_
2. gdb attach _pid_
3. gdb _filename_ corename

### 方法一 直接调试目标程序

这种方式是直接使用gdb启动一个程序进行调试，也就是说这个程序还没有启动。

### 方法二 附加进程

使用**gdb attach 程序进程ID**附加到运行中的程序

通过 `ps -ef |grep  running_program` 查询运行程序PID。

当用gdb attach
上目标进程后，调试器会暂停下来，此时可以通过continue命令让程序继续运行或加上相应的断点后在继续运行。

调试完成后，想让程序继续运行使用detach让程序和gdb调试器分离。再退出gdb。

### 方法三 进程crash之后如何定位问题—调试core文件

在程序崩溃的时候有core文件生成，可以使用core文件来定位崩溃的原因。

使用 `ulimit -c` 查看是否开启了这一机制。 `ulimit -a` 查看其他变量情况。

`ulimit -c unlimited` 设置不限制core大小。当前linux会话中有效。

设置永久有效的方式有两种

1. 在 `/etc/security/limits.conf` 中增加一行

```
#<domain>      <type>  <item>         <value>
*               soft    core          unlimited
```

1. 把 `ulimit -c unlimited` 加到 `/etc/profile` (root用户）或对应用户的baserc
   bash_profle文件中。

core文件默认命名方式是：core.pid。其位置是崩溃程序所在目录。调试core文件的命令是

`gdb filename corename  filename 是程序名`

多个程序同时崩溃区分其core文件的方法

1. 程序启动时记录下自己的PID；

```c
#include <assert.h>
#include <stdint.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>

void writePid() {
  uint32_t curPid = (uint32_t)getpid();
  FILE *f = fopen("xxserver.pid", "w");
  assert(f);
  char szPid[32];
  snprintf(szPid, sizeof(szPid), "%d", curPid);
  fwrite(szPid, strlen(szPid), 1, f);
  fclose(f);
}
```

1. 自定义core文件的名称和目录

`/proc/sys/kernel/core_uses_pid`
可以控制产生的core文件的文件名中是否添加PID作为拓展，如果添加则文件内容为1，否则为0；
`/proc/sys/kernel/core_pattern`
可以设置格式化的core文件保存位置或文件名。修改方式如下：

`echo "/corefile/core-%e-%p-%t" > /proc/sys/kernel/core_pattern`

| 参数名称 | 参数含义                           |
| -------- | ---------------------------------- |
| %p       | 添加pid到core文件名中              |
| %u       | 添加当前uid到core文件名中          |
| %g       | 添加当前gid到core文件名中          |
| %s       | 添加导致产生core的信号到core文件中 |
| %t       | 添加core文件生成时间到core文件名中 |
| %h       | 添加主机名到core文件中             |
| %e       | 添加文件名到core文件中             |

假设程序叫**test** , 设置 core文件名如下:

`echo "/root/testcore/core-%e-%p-%t" > /proc/sys/kernel/core_pattern`

那么最终会在 `/root/testcore/` 目录下生成core文件名格式如下：

`core-test-13154-1547445291`

需要注意必须对指定core文件目录具有写权限。

## gdb 常用调试命令概览和说明

| 命令名称     | 命令缩写 | 命令说明                                                         |
| ------------ | -------- | ---------------------------------------------------------------- |
| run          | r        | 运行一个程序                                                     |
| continue     | c        | 让暂停的程序继续运行                                             |
| break        | b        | 添加断点                                                         |
| tbreak       | tb       | 添加临时断点                                                     |
| backtrace    | bt       | 查看当前线程的调用堆栈                                           |
| frame        | f        | 切换到当前调用线程的指定堆栈                                     |
| info         |          | 查看断点/线程信息                                                |
| enable       |          | 启用某个断点                                                     |
| disable      |          | 禁用某个断点                                                     |
| delete       | del      | 删除断点                                                         |
| list         | l        | 显示源码                                                         |
| print        | p        | 打印或修改变量或寄存器值                                         |
| ptype        |          | 查看变量类型                                                     |
| thread       |          | 切换到指定线程                                                   |
| next         | n        | 运行到下一行                                                     |
| step         | s        | 如果有调用函数，进入调用的函数内部，相当于step into              |
| until        | u        | 运行到指定行停下来                                               |
| finish       | fi       | 结束当前调用函数，到上一层函数调用处                             |
| return       |          | 结束当前调用函数并返回指定值，到上一层函数调用处                 |
| jump         | j        | 将当前程序执行流跳转到指定行或地址                               |
| disaassemble | dis      | 查看汇编代码                                                     |
| set args     |          | 设置程序启动命令行参数                                           |
| show args    |          | 查看设置的命令行参数                                             |
| watch        |          | 监视某一个变量或内存地址的值是否发生变化                         |
| display      |          | 监视的变量或者内存地址，当程序中断后自动输出监控的变量或内存地址 |
| dir          |          | 重定向源码的位置                                                 |

### break命令

```
## 在函数名为functionname的入口处添加一个断点 
break functionname                 
## 在当前文件行号为LineNo处添加一个断点
break LineNo                            
## 在filename文件行号为LineNo处添加一个断点
break filename:LineNo
```

### backtrace命令和frame命令

backtrace可简写成bt,
用来查看当前所在线程的调用堆栈。（可以看作是当前函数的调用链）。通过
`frame 堆栈编号 （编号不不用加 #）` 切换堆栈。

### info break, enable, disable, delete命令

enable disable 禁用启用断点， 不加参数则表示全部

delete 删除断点

### list命令

list命令可以查看当前断点附近的代码。list会显示前后10行代码。继续输入list指令会以递增行号的形式继续显示剩余的代码行，一直到文件结束为止。list指令可以往前和往后显示代码，命令分别是
list + 和 list -

### print 命令和 ptype 命令

print 命令可以方便地查看变量的值，也可以修改当前内存中的变量值。

`{0 <repeats 16 times>}` 这是gdb
显示字符串，字符数组特有的方式，当一个字符串变量，字符数组或者连续的内存值重复若干次，gdb就会以这种模式来显示，以节约显示空间。

print命令不仅可以输出变量值，也可以输出特定表达式的计算结果值，甚至可以输出一些函数的执行结果。

举个例子，我们可以输入 p &server.port 来输出 server.port 的地址，如果在 C++
对象中，我们可以 通过 p this 来显示当前对象的地址，也可以通过 p *this
来列出当前对象的各个成员变量值，如果有三个变量可以相加（假设变量名分别叫
a、b、c），我们可以使用 p a+b+c 来打印这三个变量的结果值

假设 func() 是一个可以执行的函数，p func()
命令可以输出该变量的执行结果。举一个最常用的例
子，某个时刻，某个系统函数执行失败了，通过系统变量 errno
得到一个错误码，我们可以使用 p strerror(errno)
将这个错误码对应的文字信息打印出来，这样我们就不用费劲地去 man
手册上查找这个错误码对应的错误含义了。

print 命令不仅可以输出表达式结果，同时也可以修改变量的值。

`p server.port=6400`

print 输出变量值时可以指定输出格式，命令使用格式如下：

`print /format variable`

format 常见的取值有：

```
o octal 八进制显示
x hex 十六进制显示
d decimal 十进制显示
u unsigned decimal 无符号十进制显示
t binary 二进制显示
f float 浮点值显示
a address 内存地址格式显示(与十六进制相似)
i instruction 指令格式显示
s string 字符串形式显示
z hex, zero padded on the left 十六进制左侧补0显示
```

完整的格式和用法可以在 gdb 中输入 help x 来查看。

总结起来，利用 print
命令，我们不仅可以查看程序运行过程中的各个变量的状态值，也可以通过临时修改变量的值来控制程序的行为。

gdb 还有另外一个命令叫 ptype，顾名思义，其含义是 “print
type”，就是输出一个变量的类型。

对于一个复合数据类型的变量，ptype
不仅列出了这个变量的类型而且详细地列出了每个成员变量的字段名。

### info命令和thread命令

info threads LWP 后的数值是线程ID 线程编号前面的星号表示当前gdb作用于哪个线程。

**thread 线程编号** 切换到具体线程。

info 命令还可以用来查看当前函数的参数值，组合命令是： **info args**

### next、step、until、finish、return、jump 命令

之所以把这几个命令放在一起是因为在我们用 gdb
调试程序时，最常用的几个控制流命令。next 命令简写成 n，意思是让 gdb
调到下一条命令去执行，这里跳转到下一条命令执行不是说一定跳到代码的临近下一行，而是根据程序逻辑，跳转到相应的位置。

在 gdb
命令行界面如果直接按下回车键，默认是将最近一条命令重新执行一遍，所以，当我们使用
next 命令单步调试时，不必反复输入 n 命令，直接回车就可以。

next 命令用调试的术语叫“单步步过”（step
over），即遇到函数调用不进入函数体内部而直接跳过； step 命令就是”单步步入“（step
into），顾名思义，就是遇到函数调用，进入函数内部。step 可简写成 s。

说到 step
命令，我们还有一个需要注意的地方，就是当函数的参数也是函数调用时，这个时候，使用step命令会依次进入各个函数，那么顺序是什么呢。

```c
#include <stdio.h>

int func1(int a, int b) {
  int c = a + b;
  c += 2;
  return c;
}

int func2(int p, int q) {
  int t = p * q;
  return t * t;
}

int func3(int m, int n) { return m + n; }

int main(void) {
  int c;
  c = func3(func1(1, 2), func2(8, 9));
  printf("c=%d.\n", c);
  return 0;
}
```

### 函数的调用方式

常用的函数调用方式有`__cdecl`,`__stdcall`
,C++的非静态成员函数的调用方式是`__thiscall`
,这些调用方式，函数参数的传递本质上是函数参数的入栈的过程，而这三种调用方式参数的入栈顺序都是从右往左的，所以，这段代码中并没有显示表明函数的调用方式，所以采用默认`_cdecl`
。

当我们在 `c = func3(func1(1, 2), func2(8, 9));`代码处，输入 step 先进入的是
func2()，当从 func2() 返回时再次输入step 命令会接着进入 func1()，当从 func1
返回时，此时两个参数已经计算出来了，这时候会最终进入 func3()。

在实际调试的时候，我们在某个函数中调试一会儿后，我们不需要再一步步执行到函数返回处，我们希望直接执行完当前函数并回到上一层调用处，我们可以使用
finish 命令。与 finish 命令类似的还有return 命令，return
命令作用是结束执行当前函数，还可以指定该函数的返回值。这里需要注意一下二者的区别：**finish
命令会执行函数到正常退出该函数；而 return
命令是立即结束执行当前函数并返回，也就是说，如果当前函数还有剩余的代码未执行完毕，也不会执行了**。

### until命令

实际调试时，还有一个叫 until 的命令，简写成
u，我们使用使用这个命令指定程序运行到某一行停下来。

jump 命令基本用法是：`jump <location>`

location 可以是程序的行号或者函数的地址，jump
会让程序执行流跳转到指定位置执行，当然其行为也是不可控制的，例如您跳过了某个对象的初始化代码，直接执行操作该对象的代码，那么可能会导致程序崩溃或其他意外行为。jump
命令可以简写成 j，但是不可以简写成 jmp，其使用有一个注意事项，即如果 jump
跳转到的位置后续没有断点，那么 gdb 会执行完跳转处的代码会继续执行。

jump
命令除了跳过一些代码的执行外，还有一个妙用就是可以执行一些我们想要执行的代码，而这些代码在正常的逻辑下可能并不会执行(比如进入到if不成立的条件分支下，当然也因此会产生一些非预期结果，这需要自行斟酌使用。

### disassemble 命令

在一些高级调试时，我们可能要查看某段代码的汇编指令去排查问题，或者是在调试一些没有调试信息的发布版程序时，也只能通过反汇编代码去定位问题，那么
disassemble 命令就派上用场了。 disassemble 会输出**当前所在函数**的汇编指令。

gdb 默认反汇编为 AT&T 格式的指令，可以通过 show disassembly-flavor
查看。如果习惯 intel 汇编格式的，用命令 **set disassembly-flavor intel**
来设置。

这个命令在我们只有程序崩溃后产生 core
文件，且无对应的调试符号时非常有用，我们可以通过分析汇编代码定位一些错误。

**set args 与 show args 命令**

很多程序，需要我们传递命令行参数。在 gdb 调试中，很多人会觉得可以使用 gdb
filename args 这 种形式来给 gdb
调试的程序传递命令行参数，这样是不行的。正确的做法是在用 gdb 附加程序后，在使用
run 命令之前，使用 set args 您要的命令行参数来指定。

如果您单个命令行参数之间含有空格，可以使用引号将参数包裹起来。

`(gdb) set args "999 xx" "hu jj"` `(gdb) show args`

如果想清除掉已经设置好的命令行参数，使用 set args 不加任何参数即可。

### watch 命令

watch
是一个强大的命令，它可以用来监视一个变量或者一段内存，当这个变量或者该内存处的值发送变化时，gdb
就会中断下来。监视某个变量或者某个内存地址会产生一个“watch point”（观察点）。

watch 命令就实际上可能会通过添加硬件断点来达到监视数据变化的目的。watch
命令的使用方式是 watch 变量名或内存地址，一个 watch pointwatch
一般有以下几种格式：

1. 整形变量

```c
int i;
watch i;
```

1. 指针类型

```c
char *p
watch p 与 watch *p
```

注意： watch p 与 watch *p 是有区别的，前者是查看 *(&p)， 是 p 变量本身；后者是
p 所指的
内存的内容，一般是我们所需要的，我们大多数情况就是要看某内存地址上的数据是怎样变化的。

1. watch 一个数组或内存空间

```c
char buf[128];
wathc buf
```

这里是对 buf 的 128 个数据进行了监视

需要注意的是：当设置的观察点是一个局部变量时。局部变量无效后，观察点也会失效。

### display 命令

display
命令监视的变量或者内存地址，每次程序中断下来都会自动输出这些变量或内存的值。

可以使用 info display 查看当前已经自动添加了哪些值，使用 delete display
清除全部需要自动输出的变量，使用delete display 编号 删除某个自动输出的变量。

### dir 命令 —— 让被调试的可执行程序匹配源代码

我们在使用 gdb
调试时由于生成可执行文件的机器和实际执行该可执行程序的机器不是同一台机器，例如大多数企业产生目标服务程序的机器是编译机器，即发版机，然后把发版机产生的可执行程序拿到生产机器上去执行。这个时候，如果可执行程序产生了崩溃，我们用
gdb 调试 core 文件时，gdb
会提示或者由于一些原因，编译时的源码文件被挪动了位置，使用 gdb
调试时也会出现上述情况。 gcc/g++ 编译出来的可执行程序并不包含完整源码，-g
只是加了一个可执行程序与源码之间的位置映射关系，我们可以通过 dir
命令重新定位这种关系。

dir 命令使用格式：

```c
# 加一个源文件路径到当前路径的前面,指定多个路径，可以使用”:”
dir SourcePath1:SourcePath2:SourcePath3
```

SourcePath1、SourcePath2、SourcePath3 指的就是需要设置的源码目录，gdb
会依次去这些目录 搜索相应的原文件。

如果要查看当前设置了哪些源码搜索路径，可以使用 show dir 命令

dir 命令不加参数表示清空当前已设置的源码搜索目录

## 使用 gdb 调试多线程程序

**调试时控制线程切换**

在调试多线程程序时，有时候我们希望执行流一直在某个线程执行，而不是切换到其他线程，有办法做到这样吗？

针对调试多线程存在的上述状况，gdb
提供了一个在调试时将程序执行流锁定在当前调试线程的命令选项——scheduler-locking
选项，这个选项有三个值，分别是 on、step 和 off，使用方法如下：

`set scheduler-locking on/step/off`

set scheduler-locking on 可以用来锁定当前线程，只观察这个线程的运行情况，
当锁定这个线程 时， 其他线程就处于了暂停状态，也就是说你在当前线程执行
next、step、until、finish、return 命令时，其他线程是不会运行的。

set scheduler-locking step 也是用来锁定当前线程，当且仅当使用 next 或 step
命令做单步调试时 会锁定当前线程，如果你使用 until、finish、return
等线程内调试命令，但是它们不是单步命令，所以其他线程还是有机会运行的。相比较 on
选项值，step
选项值给为单步调试提供了更加精细化的控制，因为通常我们只希望在单步调试时，不希望其他线程对当前调试的各个变量值造成影响

set scheduler-locking off 用于关闭锁定当前线程。

## gdb 实用调试技巧

### 将 print 显示的字符串或字符数组显示完整

当我们使用 print 命令打印一个字符串或者字符数组时，如果该字符串太长，print
命令默认显示不全的，我们可以通过在 gdb 中输入 **set print element 0**
设置一下，这样再次使用 print 命令就能完整地显示该变量所有字符串了。

### 让被gdb调试的程序接收信号

这个程序中，我们让程序在接收到 Ctrl + C 信号（对应信号值是
SIGINT）时简单打印一行信息，当我们用 gdb 调试这个程序时，由于 Ctrl + C 默认会被
gdb
接收到（让调试器中断下来），导致我们无法模拟程序接收这一信号。解决这个问题有两种方式：

1. 在 gdb 中使用 signal 函数手动给我们的程序发送信号，这里就是 signal SIGINT 。
2. 改变 gdb 信号处理的设置，通过 **handle SIGINT nostop print** 告诉 gdb
   在接收到 SIGINT 时不 要停止、并把该信号传递给调试目标程序 。

### 明明函数存在，添加断点时却无效的解决方案

有时候，一个函数明明存在，并且我们的程序也存在调试符号，我们使用 break
functionName 添加

断点时，gdb 却提示：
`Make breakpoint pending on future shared library load? y/n`

即使我们输入
y，添加的断点可能也不会被正确的触发。此时我们就需要改变添加断点的策略，使用该函数所在的代码文件和行号这种方式添加断点就能添加同样效果的断点。

### 调试中的断点

实际调试中，我们一般会用到三种断点：普通断点、条件断点和硬件断点。
硬件断点又叫数据断点，这样的断点其实就是我们上文中介绍的用 watch
命令添加的部分断点（为什么是部分，而不是全部，上文中也介绍了原因了，watch
添加的断点，有部分是软中断实现的，不属于硬件断点）。**硬件断点触发时机是当监视的内存地址或者变量值发送变化时，就会触发**。
普通断点就是我们添加的断点除去条件断点和硬件断点以外的断点。
所谓条件断点，就是满足某个条件才会触发的断点。

添加条件断点的命令是：`break [lineNo] if [condition]`，其中lineNo
是程序触发断点后需要停的位置，condition 是断点触发的条件。

添加条件断点，还有一个方法就是先添加一个普通断点，然后使用
`condition 断点编号 断点触发条件`

这样的格式来添加。

## 使用 gdb 调试多进程程序 —— 以调试 nginx 为例

方法一

用 gdb 先调试父进程，等子进程被 fork 出来后，使用 gdb attach
到子进程上去。当然，您需要重新

开启一个 Shell 窗口用于调试，gdb attach 的用法在前面已经介绍过了。

然而，方法一存在一个缺点，即程序已经启动了，我们只能使用 gdb
观察程序在这之后的行为，如果我们想调试程序从启动到运行起来之间的执行流程，方法一可能不太适用。有些读者可能会说，我用
gdb附加到进程后，我加好断点然后使用 run
命令重启进程这样不就可以调试程序从启动到运行起来之间的执行流程了。问题是这种方法不是通用的，因为对于多进程服务模型，有些父子进程有一定的依赖关系，是不方便在运行过程中重启的。这个时候就可以使用方法二来调试了。

方法二

gdb 调试器提供一个选项叫 follow-fork ，通过 `set follow-fork mode`
来设置是当一个进程 fork 出新 的子进程时，gdb 是继续调试父进程（取值是
parent）还是子进程（取值是 child），默认是父进程

（取值是 parent）。

我们可以使用 show follow-fork mode 查看当前值。

我们可以利用方法二调试程序 fork
之前和之后的任何逻辑，是一种较为通用的多进程调试方法，建议读者掌握。
