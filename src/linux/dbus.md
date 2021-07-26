# dbus

## dbus 是什么东西

DBus 的出现，使得 Linux 进程间通信更加便捷，不仅可以和用户空间应用程序进行通信，而且还可以和内核的程序进行通信，
DBus 使得 Linux 变得更加智能，更加具有交互性

DBus 分为两种类型：system bus(系统总线)，用于系统(Linux)和用户程序之间进行通信和消息的传递；
session bus(会话总线)，用于桌面(GNOME, KDE 等)用户程序之间进行通信。

D-Bus 是针对桌面环境优化的 IPC（interprocess communication ）机制，用于进程间的通信或进程与内核的通信。
最基本的 D-Bus 协议是一对一的通信协议。 但在很多情况下，通信的一方是消息总线。
消息总线是一个特殊的应用，它同时与多个应用通信，并在应用之间传递消息。
下面我们会在实例中观察消息总线的作用。 消息总线的角色有点类似与 X 系统中的窗口管理器，窗口管理器既是 X 客户，又负责管理窗口。

支持 dbus 的系统都有两个标准的消息总线：系统总线和会话总线。系统总线用于系统与应用的通信。会话总线用于应用之间的通信

## 名词

### Bus name

可以把 Bus Name 理解为连接的名称，一个 Bus Name 总是代表一个应用和消息总线的连接。
有两种作用不同的 Bus Name，一个叫公共名（well-known names），还有一个叫唯一名（Unique Connection Name）。

#### 可能有多个备选连接的公共名

公共名提供众所周知的服务。其他应用通过这个名称来使用名称对应的服务。
可能有多个连接要求提供同个公共名的服务，即多个应用连接到消息总线，要求提供同个公共名的服务。
消息总线会把这些连接排在链表中，并选择一个连接提供公共名代表的服务。可以说这个提供服务的连接拥有了这个公共名。
如果这个连接退出了，消息总线会从链表中选择下一个连接提供服务。
公共名是由一些圆点分隔的多个小写标志符组成的，例如“org.fmddlmyy.Test”、“org.bluez”。

#### 每个连接都有一个唯一名

当应用连接到消息总线时，消息总线会给每个应用分配一个唯一名。唯一名以“:”开头，“:”后面通常是圆点分隔的两个数字，例如“:1.0”。
每个连接都有一个唯一名。在一个消息总线的生命期内，不会有两个连接有相同的唯一名。
拥有公众名的连接同样有唯一名，例如在前面的图中，“org.fmddlmyy.Test”的唯一名是“:1.17”。

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

#### Methods 和 Signals

接口包括方法和信号。
标准接口“org.freedesktop.DBus.Introspectable”的 Introspect 方法是个很有用的方法。
类似于 Java 的反射接口，调用 Introspect 方法可以返回接口的 xml 描述。
还有另外一种调用 Introspectable

### 消息总线

应用程序 A 和消息总线连接，这个连接获取了一个众所周知的公共名（记作连接 A）。
应用程序 A 中有对象 A1 提供了接口 I1，接口 I1 有方法 M1。
应用程序 B 和消息总线连接，要求调用连接 A 上对象 A1 的接口 I1 的方法 M1。
应用程序 B 调用应用程序 A 的方法，其实就是应用程序 B 向应用程序 A 发送了一个类型为“method_call”的消息。
应用程序 A 通过一个类型为 `method_retutn` 的消息将返回值发给应用程序 B。

### D-Bus 的消息

上一讲说过最基本的 D-Bus 协议是一对一的通信协议。与直接使用 socket 不同，D-Bus 是面向消息的协议。
D-Bus 的所有功能都是通过在连接上流动的消息完成的。

#### 消息类型

D-Bus 有四种类型的消息：

- `method_call` 方法调用
- `method_return` 方法返回
- error 错误
- signal 信号

### 消息总线的方法和信号

消息总线是一个特殊的应用，它可以在与它连接的应用之间传递消息。 可以把消息总线看作一台路由器。
正是通过消息总线，D-Bus 才在一对一的通信协议基础上实现了多对一和一对多的通信。
消息总线虽然有特殊的转发功能，但消息总线也还是一个应用。 其它应用与消息总线的通信也是通过 1.1 节的基本消息类型完成的。
作为一个应用，消息总线也提供了自己的接口，包括方法和信号。

我们可以通过向连接“org.freedesktop.DBus ”上对象“/”发送消息来调用消息总线提供的方法。
事实上，应用程序正是通过这些方法连接到消息总线上的其它应用，完成请求公共名等工作的。

#### 清单

可以调用 org.freedesktop.DBus.Introspectable.Introspect 方法查看消息总线对象支持的接口。例如：
`dbus-send --session --type=method_call --print-reply --dest=org.freedesktop.DBus / org.freedesktop.DBus.Introspectable.Introspect`

从输出可以看到会话总线对象支持标准接口“org.freedesktop.DBus.Introspectable”和接口“org.freedesktop.DBus”。
接口“org.freedesktop.DBus”有 16 个方法和 3 个信号。下表列出了“org.freedesktop.DBus”的 12 个方法的简要说明：

```bash
org.freedesktop.DBus.RequestName (in STRING name, in UINT32 flags, out UINT32 reply)
请求公众名。其中flag定义如下：
DBUS_NAME_FLAG_ALLOW_REPLACEMENT 1
DBUS_NAME_FLAG_REPLACE_EXISTING 2
DBUS_NAME_FLAG_DO_NOT_QUEUE 4

返回值reply定义如下：
DBUS_REQUEST_NAME_REPLY_PRIMARY_OWNER 1
DBUS_REQUEST_NAME_REPLY_IN_QUEUE 2
DBUS_REQUEST_NAME_REPLY_EXISTS 3
DBUS_REQUEST_NAME_REPLY_ALREADY_OWNER 4
```

```bash
org.freedesktop.DBus.ReleaseName (in STRING name, out UINT32 reply)
释放公众名。返回值reply定义如下：
DBUS_RELEASE_NAME_REPLY_RELEASED 1
DBUS_RELEASE_NAME_REPLY_NON_EXISTENT 2
DBUS_RELEASE_NAME_REPLY_NOT_OWNER 3
```

```bash
org.freedesktop.DBus.Hello (out STRING unique_name)
一个应用在通过消息总线向其它应用发消息前必须先调用Hello获取自己这个连接的唯一名。返回值就是连接的唯一名。dbus没有定义专门的切断连接命令，关闭socket就是切断连接。
在1.2节的dbus-monitor输出中可以看到dbus-send调用消息总线的Hello方法。
```

```bash
org.freedesktop.DBus.ListNames (out ARRAY of STRING bus_names)
返回消息总线上已连接的所有连接名，包括所有公共名和唯一名。例如连接“org.fmddlmyy.Test”同时有公共名“org.fmddlmyy.Test”和唯一名“:1.21”， 这两个名称都会被返回。
```

```bash
org.freedesktop.DBus.ListActivatableNames (out ARRAY of STRING bus_names)
返回所有可以启动的服务名。dbus支持按需启动服务，即根据应用程序的请求启动服务。
```

```bash
org.freedesktop.DBus.NameHasOwner (in STRING name, out BOOLEAN has_owner)
检查是否有连接拥有指定名称。
```

```bash
org.freedesktop.DBus.StartServiceByName (in STRING name, in UINT32 flags, out UINT32 ret_val)
按名称启动服务。参数flags暂未使用。返回值ret_val定义如下：
1.服务被成功启动
2.已经有连接拥有要启动的服务名
```

```bash
org.freedesktop.DBus.GetNameOwner (in STRING name, out STRING unique_connection_name)
返回拥有指定公众名的连接的唯一名。
```

```bash
org.freedesktop.DBus.GetConnectionUnixUser (in STRING connection_name, out UINT32 unix_user_id)
返回指定连接对应的服务器进程的Unix用户id。
```

```bash
org.freedesktop.DBus.AddMatch (in STRING rule)
为当前连接增加匹配规则。
```

```bash
org.freedesktop.DBus.RemoveMatch (in STRING rule)
为当前连接去掉指定匹配规则。
```

```bash
org.freedesktop.DBus.GetId (out STRING id)
返回消息总线的ID。这个ID在消息总线的生命期内是唯一的。

接口“org.freedesktop.DBus”的3个信号是：

org.freedesktop.DBus.NameOwnerChanged (STRING name, STRING old_owner, STRING new_owner)
指定名称的拥有者发生了变化。

org.freedesktop.DBus.NameLost (STRING name)
通知应用失去了指定名称的拥有权。

org.freedesktop.DBus.NameAcquired (STRING name)
通知应用获得了指定名称的拥有权。
```

## dbus 是什么东西

DBus 的出现，使得 Linux 进程间通信更加便捷，不仅可以和用户空间应用程序进行通信，而且还可以和内核的程序进行通信，
DBus 使得 Linux 变得更加智能，更加具有交互性

DBus 分为两种类型：system bus(系统总线)，用于系统(Linux)和用户程序之间进行通信和消息的传递；
session bus(会话总线)，用于桌面(GNOME, KDE 等)用户程序之间进行通信。

D-Bus 是针对桌面环境优化的 IPC（interprocess communication ）机制，用于进程间的通信或进程与内核的通信。
最基本的 D-Bus 协议是一对一的通信协议。 但在很多情况下，通信的一方是消息总线。
消息总线是一个特殊的应用，它同时与多个应用通信，并在应用之间传递消息。
下面我们会在实例中观察消息总线的作用。 消息总线的角色有点类似与 X 系统中的窗口管理器，窗口管理器既是 X 客户，又负责管理窗口。

支持 dbus 的系统都有两个标准的消息总线：系统总线和会话总线。系统总线用于系统与应用的通信。会话总线用于应用之间的通信

## 名词

### Bus name

可以把 Bus Name 理解为连接的名称，一个 Bus Name 总是代表一个应用和消息总线的连接。
有两种作用不同的 Bus Name，一个叫公共名（well-known names），还有一个叫唯一名（Unique Connection Name）。

#### 可能有多个备选连接的公共名

公共名提供众所周知的服务。其他应用通过这个名称来使用名称对应的服务。
可能有多个连接要求提供同个公共名的服务，即多个应用连接到消息总线，要求提供同个公共名的服务。
消息总线会把这些连接排在链表中，并选择一个连接提供公共名代表的服务。可以说这个提供服务的连接拥有了这个公共名。
如果这个连接退出了，消息总线会从链表中选择下一个连接提供服务。
公共名是由一些圆点分隔的多个小写标志符组成的，例如“org.fmddlmyy.Test”、“org.bluez”。

#### 每个连接都有一个唯一名

当应用连接到消息总线时，消息总线会给每个应用分配一个唯一名。唯一名以“:”开头，“:”后面通常是圆点分隔的两个数字，例如“:1.0”。
每个连接都有一个唯一名。在一个消息总线的生命期内，不会有两个连接有相同的唯一名。
拥有公众名的连接同样有唯一名，例如在前面的图中，“org.fmddlmyy.Test”的唯一名是“:1.17”。

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

#### Methods 和 Signals

接口包括方法和信号。
标准接口“org.freedesktop.DBus.Introspectable”的 Introspect 方法是个很有用的方法。
类似于 Java 的反射接口，调用 Introspect 方法可以返回接口的 xml 描述。
还有另外一种调用 Introspectable

### 消息总线

应用程序 A 和消息总线连接，这个连接获取了一个众所周知的公共名（记作连接 A）。
应用程序 A 中有对象 A1 提供了接口 I1，接口 I1 有方法 M1。
应用程序 B 和消息总线连接，要求调用连接 A 上对象 A1 的接口 I1 的方法 M1。
应用程序 B 调用应用程序 A 的方法，其实就是应用程序 B 向应用程序 A 发送了一个类型为“method_call”的消息。
应用程序 A 通过一个类型为 `method_retutn` 的消息将返回值发给应用程序 B。

### D-Bus 的消息

上一讲说过最基本的 D-Bus 协议是一对一的通信协议。与直接使用 socket 不同，D-Bus 是面向消息的协议。
D-Bus 的所有功能都是通过在连接上流动的消息完成的。

#### 消息类型

D-Bus 有四种类型的消息：

- `method_call` 方法调用
- `method_return` 方法返回
- error 错误
- signal 信号

### 消息总线的方法和信号

消息总线是一个特殊的应用，它可以在与它连接的应用之间传递消息。 可以把消息总线看作一台路由器。
正是通过消息总线，D-Bus 才在一对一的通信协议基础上实现了多对一和一对多的通信。
消息总线虽然有特殊的转发功能，但消息总线也还是一个应用。 其它应用与消息总线的通信也是通过 1.1 节的基本消息类型完成的。
作为一个应用，消息总线也提供了自己的接口，包括方法和信号。

我们可以通过向连接“org.freedesktop.DBus ”上对象“/”发送消息来调用消息总线提供的方法。
事实上，应用程序正是通过这些方法连接到消息总线上的其它应用，完成请求公共名等工作的。

#### 清单

可以调用 org.freedesktop.DBus.Introspectable.Introspect 方法查看消息总线对象支持的接口。例如：
`dbus-send --session --type=method_call --print-reply --dest=org.freedesktop.DBus / org.freedesktop.DBus.Introspectable.Introspect`

从输出可以看到会话总线对象支持标准接口“org.freedesktop.DBus.Introspectable”和接口“org.freedesktop.DBus”。
接口“org.freedesktop.DBus”有 16 个方法和 3 个信号。下表列出了“org.freedesktop.DBus”的 12 个方法的简要说明：

```bash
org.freedesktop.DBus.RequestName (in STRING name, in UINT32 flags, out UINT32 reply)
请求公众名。其中flag定义如下：
DBUS_NAME_FLAG_ALLOW_REPLACEMENT 1
DBUS_NAME_FLAG_REPLACE_EXISTING 2
DBUS_NAME_FLAG_DO_NOT_QUEUE 4

返回值reply定义如下：
DBUS_REQUEST_NAME_REPLY_PRIMARY_OWNER 1
DBUS_REQUEST_NAME_REPLY_IN_QUEUE 2
DBUS_REQUEST_NAME_REPLY_EXISTS 3
DBUS_REQUEST_NAME_REPLY_ALREADY_OWNER 4
```

```bash
org.freedesktop.DBus.ReleaseName (in STRING name, out UINT32 reply)
释放公众名。返回值reply定义如下：
DBUS_RELEASE_NAME_REPLY_RELEASED 1
DBUS_RELEASE_NAME_REPLY_NON_EXISTENT 2
DBUS_RELEASE_NAME_REPLY_NOT_OWNER 3
```

```bash
org.freedesktop.DBus.Hello (out STRING unique_name)
一个应用在通过消息总线向其它应用发消息前必须先调用Hello获取自己这个连接的唯一名。返回值就是连接的唯一名。dbus没有定义专门的切断连接命令，关闭socket就是切断连接。
在1.2节的dbus-monitor输出中可以看到dbus-send调用消息总线的Hello方法。
```

```bash
org.freedesktop.DBus.ListNames (out ARRAY of STRING bus_names)
返回消息总线上已连接的所有连接名，包括所有公共名和唯一名。例如连接“org.fmddlmyy.Test”同时有公共名“org.fmddlmyy.Test”和唯一名“:1.21”， 这两个名称都会被返回。
```

```bash
org.freedesktop.DBus.ListActivatableNames (out ARRAY of STRING bus_names)
返回所有可以启动的服务名。dbus支持按需启动服务，即根据应用程序的请求启动服务。
```

```bash
org.freedesktop.DBus.NameHasOwner (in STRING name, out BOOLEAN has_owner)
检查是否有连接拥有指定名称。
```

```bash
org.freedesktop.DBus.StartServiceByName (in STRING name, in UINT32 flags, out UINT32 ret_val)
按名称启动服务。参数flags暂未使用。返回值ret_val定义如下：
1.服务被成功启动
2.已经有连接拥有要启动的服务名
```

```bash
org.freedesktop.DBus.GetNameOwner (in STRING name, out STRING unique_connection_name)
返回拥有指定公众名的连接的唯一名。
```

```bash
org.freedesktop.DBus.GetConnectionUnixUser (in STRING connection_name, out UINT32 unix_user_id)
返回指定连接对应的服务器进程的Unix用户id。
```

```bash
org.freedesktop.DBus.AddMatch (in STRING rule)
为当前连接增加匹配规则。
```

```bash
org.freedesktop.DBus.RemoveMatch (in STRING rule)
为当前连接去掉指定匹配规则。
```

```bash
org.freedesktop.DBus.GetId (out STRING id)
返回消息总线的ID。这个ID在消息总线的生命期内是唯一的。

接口“org.freedesktop.DBus”的3个信号是：

org.freedesktop.DBus.NameOwnerChanged (STRING name, STRING old_owner, STRING new_owner)
指定名称的拥有者发生了变化。

org.freedesktop.DBus.NameLost (STRING name)
通知应用失去了指定名称的拥有权。

org.freedesktop.DBus.NameAcquired (STRING name)
通知应用获得了指定名称的拥有权。
```

```other
*  The workflow of a DLC developer involves following few tasks:
- [ ] [Create a DLC]
- [ ] [Write platform code to request DLC]
- [ ] [Install a DLC for dev/test]
- [ ] [Write tests for a DLC]


crosini_client
vm_concierge
vm_concone
arc
cros_disks


dbus
C++(虚函数 模版)

rust(cxx crate)
```
