# 源代码梳理

update_engine 更新策略，采用 A/B 面无扰的 OTA 升级模式，A/B 面都是完整的运行环境（这点也可以从启动镜像的 boot 目录中看出来）, 更新过程中，只在没有运行的那一面环境进行更新操作，只有更新真正完成（即重启后成功进入系统）。才会切换把运行环境切换到升级的那一面。

## 更新策略

可通过实现更新策略管理器(Policy Management)中定义的接口，定义更新的行为：比如说更新检查多久运行一次等。

从更新形式上有增量(delta)更新，和全局更新。方式可通过 p2p 和联网更新。

## client-headers

根据 `../dbus_bindings/org.chromium.UpdateEngineInterface.dbus-xml`
生成 dbus 服务

```
nodename /org/chromium/UpdateEngine

interface name="org.chromium.UpdateEngineInterface"

fn AttemptUpdate(s:app_version, s:omaha_url)

-- See AttemptUpdateFlags enum in update_engine/dbus-constants.h
fn AttemptUpdateWithFlags(s:app_version, s:omaha_url, i: flags)

fn  AttemptInstall(s: omaha_url, as: dlc_ids)

fn AttemptRollback(b: powerwash) -> (b: can_rollback)

fn ResetStatus()

fn SetDlcActiveValue(b: is_active, s: dlc_id)

fn GetStatusAdvanced() -> (ay: status)

fn RebootIfNeeded()

fn SetChannel(s: target_channel, b: is_powerwash_allowed)

fn GetChannel(b: get_current_channel) -> (s:channel)

fn SetCohortHint(s: cohort_hint)

fn SetP2PUpdatePermission(b: enabled)

fn GetP2PUpdatePermission() -> (b:enabled)

fn SetUpdateOverCellularPermission(b: allowed)

fn SetUpdateOverCellularTarget(s: taget_version, x: target_size)

fn GetUpdateOverCellularPermission() -> (b: allowed)

fn ToggleFeature(b: feature, b: enabled)

fn GetDurationSinceUpdate() -> (x: usec_wallclock)

Signal  StatusUpdateAdvanced() -> (ay : status)

fn GetPrevVersion() -> (s: prev_version)

fn GetRollbackPartition() -> (s: rollback_partition_name)


fn GetLastAttemptError() -> (i: last_attempt_eror)
```

## client-library

update-engine 的主要方法 更新，安装等等。

## payload-consumer

自定义 payload 相关

## payload_generator

AB 更新相关代码

## script

python 脚本，提供一些类似检验 sha256 的脚本

## update_manager

各种更新策略的实现。

# developer console

默认用户名 chronos 密码 test0000

# chrome os 更新进程

更新引擎是一个单线程进程运行在整个时间。它运行优先级很低，并且是最后一个启动的进程在启动后。 不同的客户端 像是 chrome 和其他服务能给更新引擎发送检查更新的请求。 这个细节如何发送给更新请求的是依赖系统的，但是在 chrome os 中是
dbus 接口。

一下几点包括但不限于提供了弹性功能

1. 如果更新引擎崩溃了他会自动重启
2. 在更新期间它周期性地检查点更新状态，并且如果它如果在更新中失败了或中途崩溃了， 它会从最后的检查点开始。
3. 网络错误会重试
4. 如果它应用 delta 部署 失败几次后，他会切换到完整部署

更新器会将他的活动表现写入 `/var/lib/update_engine/prefs`中， 这些数据帮助它跟踪他的生命周期内更新 变化，以便从错误中回复

### 方案管理

在 chrome os 中，设备被允许接受不同的方案从他们的管理组织，一些方案影响合适或如何执行他们的更新。 比如说，一个组织可能想去分发他们的更新整天，（这段翻译地不懂） updateManager 可以存储像是这样的 方案管理。

## 回滚操作 略

### 交互式 vs 非交互式 vs 强制更新

非交互式 更新周期地运行在后台， 交互式的发生在用户点击检查更新。 在更新服务方案中，交互式优先级高于 非交互式。强制更新可以通过

```bash
update_engine_client --interactive=false --check_for_update
```

### p2p 更新

很多组织可能没有足够的带宽达到更新所有设备的要求。 chrome os 可以扮演一个为相同分支网络下为其他 客户端设备提供负载服务。

### logs

更新引擎的日志都存放在 `/var/log/update_engine` 文件夹， 以时间作为后缀，最近的活动日志 会被连接到 `/var/log/update_engine.log`

## 更新负载带

更新负载带是将一组分区或文件变成更新服务器可以理解的，和确保安全的格式的进程。 这个进程涉及分由于网络带宽下载负载, 把分区分解成更小的模块，并且压缩他们的。 对于每一次生成负载
，都有一个一致属性的文件储存为包含了元数据信息 json 格式的文件。 通常位于与生成的负载在同一位置，并且他的文件名是后缀上加`.json`.
`/path/to/payload.bin` -> `/path/to/payload.bin.json`

`delta_generator` 为不同类型的更新负载提供了很大范围可供选择的选项。

代码位于 `update_engine/payload_generator`
不建议使用`delta_generator`, 要想更容易地生成负载，使用`cros_generate_update_payloads`
大多数高等级的操作位于`chromite/lib/paygen`下。

### 更新负载文件特征码 略

### delta vs full update payloads 略

#### 生成 delta 负载

delta 负载生成通过查看所有的源代码和目标镜像数据在一个文件和元数据基本信息
(更精确的说是 文件系统层级在每一个合适的分区)。我们生成 delta 负载的原因是 chrome os 是只读的。 所以我们在客户端设备上能高度确信活动分区(active partitions)
是与通过镜像生成标志原始分区的相位(phase)比特级别进行比较。 其过程可以粗纳归结为

1. 寻找所有 zero-filled 在目标分区上的块并且运行 `Zero` 操作。
   `ZERO` 操作基本上是 丢弃有关联的块（依赖于其具体实现）
2. 找到所有这样的块： 通过直接一对一的比较源和目标块并且执行过 `SOURCE_COPY` 操作还没有改变区别。
3. 列出所有源文件和目标文件和他们关联的的块，并且移除我们在上两个步骤已经生成操作的块和文件。 分配每一个分区剩余元数据作为文件
4. 如果一个文件是新的，为每一个依赖与生成了更小二进制大数据 执行`REPALCE`,
   `REPALCE_XZ`, `REPALCE_BZ` 操作
5. 对于其他的文件，比较他的源和目标块，给生成更小的二进制大文件执行 `SOURCE_DSDDIFF`
   或者 `PUFFDIFF` 操作
6. 基于他们目标块偏移排序这些操作
7. 可选的 合并相同或者相近的操作成为一个更大的操作 更好的效率和潜在的更小的负载。

### 主要版本和次要版本 略

### 签名和未签名负载

`delta_generator` 可以为生成元数据和负载 hashes 和签名

## 更新负载脚本

更新负载包含了一些 python 脚本。
`brillo_update_payload` 脚本可以用于生成和测试应用于在主机设备上测试应用一个负载。 这些测试能够在动态测试中查看不必用于一个真实的设备。其他的一些`update_payload` 脚本 像是
`check_update_payload` 能够静态检查一个负载是正确的状态和应用工作正确。

## 安装后

`Postinstall` 是一个进程用于更新客户端写入一个新镜像到非活动分区(inactive partitions)
postinsatll 的一个主要之策是重新创造 dm-verity 树 hash 在根分区的最底端。 在其他事情中间，它下载新固件更新或者任一主板特征进程。因此它确实从剩余的活动运行系统中分离出来。
任何需要在更新后和重启前需要做的事，应该要实现 postinstall 操作。

## 构建更新引擎

你同样可以构建`update_engin` 跟其他设备应用一样。

```bash
(chroot) $ emerge-${BOARD} update_engine
```

or to build without the source copy:

```bash
(chroot) $ cros_workon_make --board=${BOARD} update_engine
```

在更改为 update_engine 后台之后，不管是生成镜像还是安装镜像到设备使用 cros flash. 或者用 `cros deploy` 仅仅只是安装`update_engin` 服务到设备中。

```bash
(chroot) $ cros deploy update_engine
```

你应该需要重启 update_engine 查看更改后的效果

```bash
# SSH into the device.
restart update-engine # with a dash not underscore.
```

其他负载生成工具比如 `delta_generator` 是主板不可知的只能用于 sdk 中。如果要修改 应该构建 sdk:

```bash
# Do it only once to start building the 9999 ebuild from ToT.
(chroot) $ cros_workon --host start update_engine

(chroot) $ sudo emerge update_engine
```

如果你做了任何 dbus 接口改变，确保使用 `system_api` `update_engine-client`
和 `update_engine`
被标记从 999 ebuild 构建并且这些包要按照这样的顺序:

```bash
(chroot) $ emerge-${BOARD} system_api update_engine-client update_engine
```

如果任何改变在 `update_engine` protobufs 在 `system_api` 首先构建 `system_api` 包。

## 运行单元测试

```bash
(chroot) $ FEATURES=test emerge-<board> update_engine
```

或者

```bash
(chroot) $ cros_workon_make --board=<board> --test update_engine
```

或者

```bash
(chroot) $ cros_run_unit_tests --board ${BOARD} --packages update_engine
```

上面的命令会运行所有的单元测试，但是`update_engine` 太大了并且他会花费很长时间。 运行所有的单元测试在一个测试类里运行:

```bash
(chroot) $ FEATURES=test \
    P2_TEST_FILTER="*OmahaRequestActionTest.*-*RunAsRoot*" \
    emerge-amd64-generic update_engine
```

运行一个精确的单元测试运行：

```bash
(chroot) $ FEATURES=test \
    P2_TEST_FILTER="*OmahaRequestActionTest.MultiAppUpdateTest-*RunAsRoot*" \
    emerge-amd64-generic update_engine
```

运行 `update_payload` 单元测试进入 `update_engine/script` 文件夹并且运行 `unittet.py`

## 初始化一个配置的更新

有以下几种方式初始化一个更新：

1. 点击设置界面的检查更新按钮，这种方式没有配置选项。
2. 使用`update_engine_client` 程序，有一些选项可以配置。
3. 在 crosh 调用`autest` ,主要是 QA team 使用。
4. 使用 `cros flash` 。它内部会使用 update_engine 去刷入一个镜像
5. 运行众多自动更新自动测试中的一个。
6. 开始一个 `Dev Server` 在你的主机中，并且发送一个特殊的 http 请求 查看 `cros_au` api 在 Dev Serer 代码中)，需要你的 chromebook 的 ip 地址和哪里的
   更新负载去开始一个更新（会编译，不推荐）

`update_engine_client` 是一个客户端程序可以帮助初始化，一个更新或者得到更多的信息关于 更新器的客户端的状态。他有一些选项像是初始化一个交互式还是非交互式更新，改变 channels，
获取当更新进程现在的状态，回滚操作，改变 Omaha URL 去下载负载（这是最重要的一个）.

`update_engine` 守护进程读取 设备中的`/etc/lsb-release` 文件 去识别不同的更新参数像是 更新服务 Omaha url, 当前 channel 等。然而，重写任意一个参数，创建
带有需求配置参数的`/mnt/stateful_partition/etc/lsb-release` 文件。比如说这个可以被 用作标记一个开发者版本跟新服务器并且允许 update_engine 安排周期性的更新从特定服务器。

## 开发者和维护者需要注意的

当改变 update engine 源代码要特别的关心这些事：

### 不要破坏后台完整性

在每一个发行循环中我们应该能够生成完全和 delta 负载，这样能正确的应用在老设备运行早一版的 update engine client。 比如： 移除或者没有通过元数据原型文件可能会破坏旧文件。
或者操作不能理解旧版的客户端会破坏他们。无论任何时候在修改任何负载生成进程，问一问一个问题 是否会破坏旧版客户端。

### 想想未来 略

### 不要倾向于实现功能通过 更新客户端

如果一个功能可以从服务端实现，不要从客户端实现，因为客户端更新器是不牢固的。后略。

## client-headers

根据 `../dbus_bindings/org.chromium.UpdateEngineInterface.dbus-xml`
生成 dbus 服务

## client-library

update-engine 的主要方法 更新，安装等等。

## payload-consumer

# developer console

默认用户名 chronos 密码 test0000

# chrome os 更新进程

更新引擎是一个单线程进程运行在整个时间。它运行优先级很低，并且是最后一个启动的进程在启动后。 不同的客户端 像是 chrome 和其他服务能给更新引擎发送检查更新的请求。 这个细节如何发送给更新请求的是依赖系统的，但是在 chrome os 中是
dbus 接口。

一下几点包括但不限于提供了弹性功能

1. 如果更新引擎崩溃了他会自动重启
2. 在更新期间它周期性地检查点更新状态，并且如果它如果在更新中失败了或中途崩溃了， 它会从最后的检查点开始。
3. 网络错误会重试
4. 如果它应用 delta 部署 失败几次后，他会切换到完整部署

更新器会将他的活动表现写入 `/var/lib/update_engine/prefs`中， 这些数据帮助它跟踪他的生命周期内更新 变化，以便从错误中回复

### 方案管理

在 chrome os 中，设备被允许接受不同的方案从他们的管理组织，一些方案影响合适或如何执行他们的更新。 比如说，一个组织可能想去分发他们的更新整天，（这段翻译地不懂） updateManager 可以存储像是这样的 方案管理。

## 回滚操作 略

### 交互式 vs 非交互式 vs 强制更新

非交互式 更新周期地运行在后台， 交互式的发生在用户点击检查更新。 在更新服务方案中，交互式优先级高于 非交互式。强制更新可以通过

```bash
update_engine_client --interactive=false --check_for_update
```

### p2p 更新

很多组织可能没有足够的带宽达到更新所有设备的要求。 chrome os 可以扮演一个为相同分支网络下为其他 客户端设备提供负载服务。

### logs

更新引擎的日志都存放在 `/var/log/update_engine` 文件夹， 以时间作为后缀，最近的活动日志 会被连接到 `/var/log/update_engine.log`

## 更新负载带

更新负载带是将一组分区或文件变成更新服务器可以理解的，和确保安全的格式的进程。 这个进程涉及分由于网络带宽下载负载, 把分区分解成更小的模块，并且压缩他们的。 对于每一次生成负载
，都有一个一致属性的文件储存为包含了元数据信息 json 格式的文件。 通常位于与生成的负载在同一位置，并且他的文件名是后缀上加`.json`.
`/path/to/payload.bin` -> `/path/to/payload.bin.json`

`delta_generator` 为不同类型的更新负载提供了很大范围可供选择的选项。

代码位于 `update_engine/payload_generator`
不建议使用`delta_generator`, 要想更容易地生成负载，使用`cros_generate_update_payloads`
大多数高等级的操作位于`chromite/lib/paygen`下。

### 更新负载文件特征码 略

### delta vs full update payloads 略

#### 生成 delta 负载

delta 负载生成通过查看所有的源代码和目标镜像数据在一个文件和元数据基本信息
(更精确的说是 文件系统层级在每一个合适的分区)。我们生成 delta 负载的原因是 chrome os 是只读的。 所以我们在客户端设备上能高度确信活动分区(active partitions)
是与通过镜像生成标志原始分区的相位(phase)比特级别进行比较。 其过程可以粗纳归结为

1. 寻找所有 zero-filled 在目标分区上的块并且运行 `Zero` 操作。
   `ZERO` 操作基本上是 丢弃有关联的块（依赖于其具体实现）
2. 找到所有这样的块： 通过直接一对一的比较源和目标块并且执行过 `SOURCE_COPY` 操作还没有改变区别。
3. 列出所有源文件和目标文件和他们关联的的块，并且移除我们在上两个步骤已经生成操作的块和文件。 分配每一个分区剩余元数据作为文件
4. 如果一个文件是新的，为每一个依赖与生成了更小二进制大数据 执行`REPALCE`,
   `REPALCE_XZ`, `REPALCE_BZ` 操作
5. 对于其他的文件，比较他的源和目标块，给生成更小的二进制大文件执行 `SOURCE_DSDDIFF`
   或者 `PUFFDIFF` 操作
6. 基于他们目标块偏移排序这些操作
7. 可选的 合并相同或者相近的操作成为一个更大的操作 更好的效率和潜在的更小的负载。

### 主要版本和次要版本 略

### 签名和未签名负载

`delta_generator` 可以为生成元数据和负载 hashes 和签名

## 更新负载脚本

更新负载包含了一些 python 脚本。
`brillo_update_payload` 脚本可以用于生成和测试应用于在主机设备上测试应用一个负载。 这些测试能够在动态测试中查看不必用于一个真实的设备。其他的一些`update_payload` 脚本 像是
`check_update_payload` 能够静态检查一个负载是正确的状态和应用工作正确。

## 安装后

`Postinstall` 是一个进程用于更新客户端写入一个新镜像到非活动分区(inactive partitions)
postinsatll 的一个主要之策是重新创造 dm-verity 树 hash 在根分区的最底端。 在其他事情中间，它下载新固件更新或者任一主板特征进程。因此它确实从剩余的活动运行系统中分离出来。
任何需要在更新后和重启前需要做的事，应该要实现 postinstall 操作。

## 构建更新引擎

你同样可以构建`update_engin` 跟其他设备应用一样。

```bash
(chroot) $ emerge-${BOARD} update_engine
```

or to build without the source copy:

```bash
(chroot) $ cros_workon_make --board=${BOARD} update_engine
```

在更改为 update_engine 后台之后，不管是生成镜像还是安装镜像到设备使用 cros flash. 或者用 `cros deploy` 仅仅只是安装`update_engin` 服务到设备中。

```bash
(chroot) $ cros deploy update_engine
```

你应该需要重启 update_engine 查看更改后的效果

```bash
# SSH into the device.
restart update-engine # with a dash not underscore.
```

其他负载生成工具比如 `delta_generator` 是主板不可知的只能用于 sdk 中。如果要修改 应该构建 sdk:

```bash
# Do it only once to start building the 9999 ebuild from ToT.
(chroot) $ cros_workon --host start update_engine

(chroot) $ sudo emerge update_engine
```

如果你做了任何 dbus 接口改变，确保使用 `system_api` `update_engine-client`
和 `update_engine`
被标记从 999 ebuild 构建并且这些包要按照这样的顺序:

```bash
(chroot) $ emerge-${BOARD} system_api update_engine-client update_engine
```

如果任何改变在 `update_engine` protobufs 在 `system_api` 首先构建 `system_api` 包。

## 运行单元测试

```bash
(chroot) $ FEATURES=test emerge-<board> update_engine
```

或者

```bash
(chroot) $ cros_workon_make --board=<board> --test update_engine
```

或者

```bash
(chroot) $ cros_run_unit_tests --board ${BOARD} --packages update_engine
```

上面的命令会运行所有的单元测试，但是`update_engine` 太大了并且他会花费很长时间。 运行所有的单元测试在一个测试类里运行:

```bash
(chroot) $ FEATURES=test \
    P2_TEST_FILTER="*OmahaRequestActionTest.*-*RunAsRoot*" \
    emerge-amd64-generic update_engine
```

运行一个精确的单元测试运行：

```bash
(chroot) $ FEATURES=test \
    P2_TEST_FILTER="*OmahaRequestActionTest.MultiAppUpdateTest-*RunAsRoot*" \
    emerge-amd64-generic update_engine
```

运行 `update_payload` 单元测试进入 `update_engine/script` 文件夹并且运行 `unittet.py`

## 初始化一个配置的更新

有以下几种方式初始化一个更新：

1. 点击设置界面的检查更新按钮，这种方式没有配置选项。
2. 使用`update_engine_client` 程序，有一些选项可以配置。
3. 在 crosh 调用`autest` ,主要是 QA team 使用。
4. 使用 `cros flash` 。它内部会使用 update_engine 去刷入一个镜像
5. 运行众多自动更新自动测试中的一个。
6. 开始一个 `Dev Server` 在你的主机中，并且发送一个特殊的 http 请求 查看 `cros_au` api 在 Dev Serer 代码中)，需要你的 chromebook 的 ip 地址和哪里的
   更新负载去开始一个更新（会编译，不推荐）

`update_engine_client` 是一个客户端程序可以帮助初始化，一个更新或者得到更多的信息关于 更新器的客户端的状态。他有一些选项像是初始化一个交互式还是非交互式更新，改变 channels，
获取当更新进程现在的状态，回滚操作，改变 Omaha URL 去下载负载（这是最重要的一个）.

`update_engine` 守护进程读取 设备中的`/etc/lsb-release` 文件 去识别不同的更新参数像是 更新服务 Omaha url, 当前 channel 等。然而，重写任意一个参数，创建
带有需求配置参数的`/mnt/stateful_partition/etc/lsb-release` 文件。比如说这个可以被 用作标记一个开发者版本跟新服务器并且允许 update_engine 安排周期性的更新从特定服务器。

## 开发者和维护者需要注意的

当改变 update engine 源代码要特别的关心这些事：

### 不要破坏后台完整性

在每一个发行循环中我们应该能够生成完全和 delta 负载，这样能正确的应用在老设备运行早一版的 update engine client。 比如： 移除或者没有通过元数据原型文件可能会破坏旧文件。
或者操作不能理解旧版的客户端会破坏他们。无论任何时候在修改任何负载生成进程，问一问一个问题 是否会破坏旧版客户端。

### 想想未来 略

### 不要倾向于实现功能通过 更新客户端

如果一个功能可以从服务端实现，不要从客户端实现，因为客户端更新器是不牢固的。后略。
