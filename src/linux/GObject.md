# GObject

本教程翻译自 [GObject tutorial](https://toshiocp.github.io/Gobject-tutorial/index.html)

## 类与实例

GObject 实例被 `g_object_new` 创建。 GObject 不仅有实例还有类型。

- 一个GObject类在第一次调用 `g_object_new` 时被创建。并且只存在一个Gobject类。
- GObject 实例在每一次`g_object_new` 被调用是创建。所以，两个或更多的GObject 实例能同时存在。

在大的语义中来看，GObject 对象意味着包含了类和实例。在窄的语义中，Gobject是一个C结构体的定义

```c
typedef struct _GObject  GObject;
struct  _GObject
{
  GTypeInstance  g_type_instance;

  /*< private >*/
  guint          ref_count;  /* (atomic) */
  GData         *qdata;
};
```

`g_object_new` 函数分配 `GObject-sized` 大小的内存， 初始化内存并返回该内存的指针。这片内存就是GObject实例。

同样，GObject 类内存被 `g_object_new` 分配并且他的结构被GObjectClass定义。

```c
#include <glib-object.h>

int
main (int argc, char **argv) {
  GObject *instance1, *instance2;
  GObjectClass *class1, *class2;

  instance1 = g_object_new (G_TYPE_OBJECT, NULL);
  instance2 = g_object_new (G_TYPE_OBJECT, NULL);
  g_print ("The address of instance1 is %p\n", instance1); // 0x55895eaf7ad0
  g_print ("The address of instance2 is %p\n", instance2); // 0x55895eaf7af0

  class1 = G_OBJECT_GET_CLASS (instance1);
  class2 = G_OBJECT_GET_CLASS (instance2);
  g_print ("The address of the class of instance1 is %p\n", class1); // 0x55895eaf7880
  g_print ("The address of the class of instance2 is %p\n", class2); // 0x55895eaf7880

  g_object_unref (instance1);
  g_object_unref (instance2);

  return 0;
}
```

- `G_TYPE_OBJECT` 是Gobject 的类型。这个类型有别与 C语言中的char或 int 这些类型。他是 GObject 系统中的基石 _类型系统_ 。
  每一个数据类型像是 GObject 这样的都必须注册到类型系统中。 类型系统有一系列函数用于注册。如果这些注册函数中的一个被
  调用， 类型系统就会决定这个对象的 GType 类型的类型值并把他返回给调用者。GType 可能是一个 `unsigned long` 整数类型
  这取决于硬件。`g_object_new` 分配 `GObject-sized` 大小的内存并返回内存的首地址。
- `G_OBJECT_GET_CLASS` 返回其参数的类的指针。
- `g_object_unref` 摧毁实例并释放内存。

## 引用计数

GObject 实例拥有其内存。当它创建时被系统分配。如果他们不被使用，其内存应该被释放。然而，我们如何决定它是否无用？GObject 系统
提供了引用计数来解决这个问题。

一个实例被创建了并被其他实例或者主程序使用。这意味着实例被引用了。如果实例被A和B引用，那么他的引用数就是2。这个数字就被成为
引用计数。让我们设想这样一个场景:

- A 调用 `g_object_new` 拥有一个实例 G。 一个引用指向G， 所以引用计数是1。
- B 也需要使用G。 B 调用 `g_object_ref` 增加了一个引用计数。现在引用计数是2。
- A 不再使用G。 A 调用 `g_object_unref` 减少一个引用计数。 现在引用计数是1。
- B 不再使用G。 B 调用 `g_object_unref` 减少一个引用计数。 现在引用计数是0。
- 因为引用计数是0了。 G知道没有对象引用它了。G 开启最终流程，G不可见并且内存被释放。
