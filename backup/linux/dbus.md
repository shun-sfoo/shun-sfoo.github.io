# dbus

D-Bus 是针对桌面环境优化的 IPC（interprocess communication ）机制，
用于进程间的通信或进程与内核的通信。
最基本的 D-Bus 协议是一对一的通信协议。

dbus 提供了两个小工具：dbus-send 和 dbus-monitor。

用法

`dbus-send --session --type=method_call --print-reply --dest=org.company.Test /TestObj org.company.Test.CCalc int32:100 int32:999`

## 消息总线

在很多情况下，通信的一方是消息总线。消息总线是一个特殊的应用，它同时与多个应用通信，并在应用之间传递消息。 消息总线的角色有点类似与 X 系统中的窗口管理器，窗口管理器既是 X 客户，又负责管理窗口。

### 消息总线的流程

应用程序 A 和消息总线连接，这个连接获取了一个众所周知的公共名（记作连接 A）。
应用程序 A 中有对象 A1 提供了接口 I1，接口 I1 有方法 M1。
应用程序 B 和消息总线连接，要求调用连接 A 上对象 A1 的接口 I1 的方法 M1。
应用程序 B 调用应用程序 A 的方法，其实就是应用程序 B 向应用程序 A 发送了一个类型为`method_call` 的消息。
应用程序 A 通过一个类型为 `method_retutn` 的消息将返回值发给应用程序 B。

#### 消息类型

D-Bus 有四种类型的消息：

- `method_call` 方法调用
- `method_return` 方法返回
- error 错误
- signal 信号

### 消息总线的方法和信号

消息总线是一个特殊的应用，它可以在与它连接的应用之间传递消息。 可以把消息总线看作一台路由器。
正是通过消息总线，D-Bus 才在一对一的通信协议基础上实现了多对一和一对多的通信。
消息总线虽然有特殊的转发功能，但消息总线也还是一个应用。 其它应用与消息总线的通信也是通过基本消息类型完成的。
作为一个应用，消息总线也提供了自己的接口，包括方法和信号。

我们可以通过向连接“org.freedesktop.DBus ”上对象“/”发送消息来调用消息总线提供的方法。
事实上，应用程序正是通过这些方法连接到消息总线上的其它应用，完成请求公共名等工作的。

## 系统总线， 会话总线

支持 dbus 的系统都有两个标准的消息总线：(system bus) 接口系统总线和 (session bus) 会话总线。
系统总线用于系统与应用的通信。会话总线用于应用之间的通信。 使用 d-feet 的 python 程序，可以用它来观察系统中的 dbus 世界。

## dbus 世界中的名词解释

### Bus name

可以把 Bus Name 理解为连接的名称，一个 Bus Name 总是代表一个应用和消息总线的连接。
有两种作用不同的 Bus Name，一个叫公共名（well-known names），还有一个叫唯一名（Unique Connection Name）。

#### 可能有多个备选连接的公共名

公共名提供众所周知的服务。其他应用通过这个名称来使用名称对应的服务。
可能有多个连接要求提供同个公共名的服务，即多个应用连接到消息总线，要求提供同个公共名的服务。
消息总线会把这些连接排在链表中，并选择一个连接提供公共名代表的服务。可以说这个提供服务的连接拥有了这个公共名。
如果这个连接退出了，消息总线会从链表中选择下一个连接提供服务。
公共名是由一些圆点分隔的多个小写标志符组成的，例如“org.company.Test”、“org.bluez”。

#### 每个连接都有一个唯一名

当应用连接到消息总线时，消息总线会给每个应用分配一个唯一名。唯一名以“:”开头，“:”后面通常是圆点分隔的两个数字，例如“:1.0”。
每个连接都有一个唯一名。在一个消息总线的生命期内，不会有两个连接有相同的唯一名。
拥有公众名的连接同样有唯一名。例如“org.company.Test”的唯一名是“:1.17”。

有的连接只有唯一名，没有公众名。可以把这些名称称为私有连接，因为它们没有提供可以通过公共名访问的服务。

#### Object Paths

Bus Name 确定了一个应用到消息总线的连接。在一个应用中可以有多个提供服务的对象。
这些对象按照树状结构组织起来。 每个对象都有一个唯一的路径（Object Paths）。
或者说，在一个应用中，一个对象路径标志着一个唯一的对象。

#### interface

通过对象路径，我们找到应用中的一个对象。每个对象可以实现多个接口。

- org.freedesktop.DBus.Introspectable
- org.freedesktop.DBus.Properties

接口“org.freedesktop.DBus.Introspectable”和“org.freedesktop.DBus.Properties”是消息总线提供的标准接口。不用手动实现。

类似于 Java 的反射接口，调用 Introspect 方法可以返回接口的 xml 描述。

#### Methods 和 Signals

### DBUS 数据类型

| 标记 | 含义                                                                          |
| ---- | ----------------------------------------------------------------------------- |
| a    | ARRAY 数组                                                                    |
| b    | BOOLEAN 布尔值                                                                |
| d    | DOUBLE IEEE 754 双精度浮点数                                                  |
| g    | SIGNATURE 类型签名                                                            |
| i    | INT32 32 位有符号整数                                                         |
| n    | INT16 16 位有符号整数                                                         |
| o    | `OBJECT_PATH` 对象路径                                                        |
| q    | UINT16 16 位无符号整数                                                        |
| s    | STRING 零结尾的 UTF-8 字符串                                                  |
| t    | UINT64 64 位无符号整数                                                        |
| u    | UINT32 32 位无符号整数                                                        |
| v    | VARIANT 可以放任意数据类型的容器，数据中包含类型信息。例如 glib 中的 GValue。 |
| x    | INT64 64 位有符号整数                                                         |
| y    | BYTE 8 位无符号整数                                                           |
| ()   | 定义结构时使用。例如"(i(ii))"                                                 |
| {}   | 定义键－值对时使用。例如"a{us}"                                               |

a 表示数组，数组元素的类型由 a 后面的标记决定。例如：
"as"是字符串数组。
数组"a(i(ii))"的元素是一个结构。用括号将成员的类型括起来就表示结构了，结构可以嵌套。
数组"a{sv}"的元素是一个键－值对。"{sv}"表示键类型是字符串，值类型是 VARIANT。

## developer

使用 rust 开发
dbus 开发一般可分为客户端(client)和服务端(server)
server 端提供可供客户端调用的 api
rust 中需要用到的两个 crate
`dbus dbus-crossroads`

### 一个计算器 api

```rust
// dbus server
use dbus::blocking::Connection;
use dbus_crossroads::{Crossroads, Context};
use std::error::Error;

// This is our "Hello" object that we are going to store inside the crossroads instance.
// 这是我们“加法” 对象 存储从接口crossroadss实例
// 即返回结果
struct Counter {
    result: i32,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // 首先开始连接到会话总线的链接，并设定一个公共名
    let conn = Connection::new_session()?;
    conn.request_name("com.dmp.adder", false, true, false)?;

    // Create a new crossroads instance.
    // The instance is configured so that introspection and properties interfaces
    // are added by default on object path additions.
    // 创建一个crossroads实例
    // 这个实例的introspecable 和properties 接自动被添加了
    let mut cr = Crossroads::new();



    // Let's build a new interface, which can be used for "Hello" objects.
    // 开始创建dbus应用的接口 用于 "adder" 对象
    let iface_token = cr.register("com.dmp.adder", |b| {
        // todo signal 信号
        // Let's add a method to the interface. We have the method name, followed by
        // names of input and output arguments (used for introspection). The closure then controls
        // the types of these arguments. The last argument to the closure is a tuple of the input arguments.
        // 为此接口增加方法，
        // 定义方法名称，方法参数，和输出参数
        // 闭包方法用来控制这些参数,
        // 闭包方法最后的参数是输入参数的元组
        b.method(
            "add",
            ("arg1", "arg2"),
            ("result",),
            move |ctx: &mut Context, res: &mut Counter, (arg1, arg2): (i32, i32)| {
                println!("{} +  {}", arg1, arg2);

                res.res = arg1 + arg2;
                let result = format!("adder {}", res.res);
                Ok((result,))
            },
        );
    });



    // Let's add the "/hello" path, which implements the com.example.dbustest interface,
    // to the crossroads instance.
    // 把实现了 com.dmp.adder 接口方法加入到path
    cr.insert("/adder", &[iface_token], Counter { res: 0 });

   // Serve clients forever.
    cr.serve(&conn)?;

    unreachable!()
}

```
