---
title: 'Chromeos Dbus Bindings'
date: 2021-08-23T10:58:31+08:00
tags: ['chromium']
categories: ['developer']
draft: false
---

## Chromeos Dbus Bindings

创建 chromeos-dbus-bindings 是为了补充 libbrillo 并简化 D-Bus 守护进程和代理的实现。
它从 D-Bus 接口的 XML 规范生成 c++类
生成的绑定不会直接处理 MethodCall 对象并手动解包参数，而是负责为您编组和解包 D-Bus 方法调用参数

此外，还提供了一个 Rust crate `chromeos_dbus_bindings`，用于从内省 XML 数据生成带有 D-Bus 绑定的 Rust 库
大部分逻辑已经由 dbus-codegen-rust 提供了，但是源 XML 并不总是对 crate 文件可用，因此它包装了生成的源文件。

### 设置 chromeos-dbus-bindings

定义对象和接口的 XML 格式与[内省 API] (https://dbus.freedesktop.org/doc/dbus-specification.html#introspection-format) 中使用的格式相同。方法和信号处理程序从这个 XML 文件生成。如果您以前使用过 `dbus-c++`，那么您可能正在使用 `xml2cpp` 从 XML 规范生成 c++绑定。
如果没有，您可能需要编写 XML 规范格式。

之后，您将需要在`BUILD.gn`中设置一些操作给它的服务端和使用者。
在你的服务中会是这样的:

```gn
import("//common-mk/generate-dbus-adaptors.gni")

generate_dbus_adaptors("frobinator-adaptors") {
  sources = [
    "dbus_bindings/service.name.of.Frobinator.xml",
  ]
  dbus_adaptors_out_dir = "include/frobinator/dbus_adaptors"
  dbus_service_config = "dbus_bindings/dbus-service-config.json"
}
```

这在你服务的用户中(或在客户端库目标中):

```gn
import("//common-mk/generate-dbus-proxies.gni")

generate_dbus_proxies("frobinator-proxies") {
  sources = [
    "path/to/frobinator/dbus_bindings/service.name.of.Frobinator.xml",
  ]
  proxy_output_file = "include/frobinator/dbus-proxies.h"
}
```

JSON 服务配置文件如下所示:

```json
{
  "service_name": "service.name.of.Frobinator"
}
```

然后，在你的服务中，你可以`#include "frobinator/dbus_adaptors/service.name.of. frobinator .h"`来获取 frobinator 的接口和适配器类，
用户可以`#include <frobinator/dbus-proxies.h>`来获取代理类。
尝试遵循最佳[实践文档](https://chromium.googlesource.com/chromiumos/docs/+/HEAD/dbus_best_practices.md)，只为您的服务仅导出一个对象。

### D-Bus 类型 vs. c++类型

D-Bus 方法、信号和属性都有类型签名。当生成绑定时，chromeos-dbus 绑定会像这样将 D-Bus 类型映射到 c++类型:

| D-Bus type signature | C++ type                                               |
| -------------------- | ------------------------------------------------------ |
| y                    | `uint8_t`                                              |
| b                    | `bool`                                                 |
| n                    | `int16_t`                                              |
| q                    | `uint16_t`                                             |
| i                    | `int32_t`                                              |
| u                    | `uint32_t`                                             |
| x                    | `int64_t`                                              |
| t                    | `uint64_t`                                             |
| d                    | `double`                                               |
| s                    | `std::string`                                          |
| h                    | `brillo::dbus_utils::FileDescriptor`, `base::ScopedFD` |
| o                    | `dbus::ObjectPath`                                     |
| v                    | `(variant) brillo::Any`                                |
| (TU...)              | `std::tuple<T, U, ...>`                                |
| aT                   | `std::vector<T>`                                       |
| a{TU}                | `std::map<T, U> `                                      |
| a{sv}                | `brillo::VariantDictionary`                            |

[`brillo::dbus_utils::FileDescriptor`](https://chromium.googlesource.com/aosp/platform/external/libbrillo/+/HEAD/brillo/dbus/file_descriptor.h)

[`base::ScopedFD`](https://chromium.googlesource.com/aosp/platform/external/libchrome/+/HEAD/base/files/scoped_file.h)

[`dbus::ObjectPath`](https://chromium.googlesource.com/aosp/platform/external/libchrome/+/HEAD/base/files/scoped_file.h)

[`brillo::Any`](https://chromium.googlesource.com/aosp/platform/external/libbrillo/+/HEAD/brillo/any.h)

[`brillo::VariantDictionary`](https://chromium.googlesource.com/aosp/platform/external/libbrillo/+/HEAD/brillo/variant_dictionary.h)

这种类型映射也是递归的，即类型`a{s(io)}`的参数将被映射到`std::map<std::string, std::tuple<int32_t, dbus::ObjectPath>>`

### 方法生成

假设您有一个具有以下 XML 规范的服务:

```xml
<node name="/org/chromium/Frobinator">
  <interface name="org.chromium.Frobinator">
    <method name="Frobinate">
      <arg name="foo" type="i" direction="in" />
      <arg name="bar" type="a{sv}" direction="in" />
      <arg name="baz" type="s" direction="out" />
    </method>
  </interface>
</node>
```

生成器将生成一个类`org::chromium::FrobinatorInterface`，带有以下 c++方法签名:

```cpp

void RegisterStatusUpdateAdvancedSignalHandler(const std::Vector<uint8_t> status);

void SendStatusUpdateAdvancedSignal(const std::Vector<uint8_t> status);

```
