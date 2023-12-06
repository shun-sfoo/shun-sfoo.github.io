# GObject

本教程翻译自 [GObject tutorial](https://toshiocp.github.io/Gobject-tutorial/index.html)

final类型的定义是不包含任何子类，包含子类的类型被称为可派生类型(derivable)。

## 类与实例

GObject 实例被 `g_object_new` 创建。 GObject 不仅有实例(instance)还有类(class)。

- 一个 GObject 类在第一次调用 `g_object_new` 时被创建。并且只存在一个 GObject 类。
- GObject 实例在每一次 `g_object_new` 被调用时创建。所以，两个或更多的 GObject 实例能同时存在。

在大的语义中来看，GObject 对象意味着包含了类和实例。在窄的语义中，GObject 是一个 C 结构体的定义

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

- `G_TYPE_OBJECT` 是Gobject 的类型。这个类型有别与 C 语言中的 char 或 int 这些类型。他是 GObject 系统中的基石 _类型系统_ 。
  每一个数据类型像是 GObject 这样的都必须注册到类型系统中。 类型系统有一系列函数用于注册。如果这些注册函数中的一个被调用， 类型系统就会决定这个对象的 GType 类型的类型值并把他返回给调用者。GType 可能是一个 `unsigned long` 整数类型这取决于硬件。`g_object_new` 分配 `GObject-sized` 大小的内存并返回内存的首地址。
- `G_OBJECT_GET_CLASS` 返回其参数的类的指针。
- `g_object_unref` 摧毁实例并释放内存。

### 引用计数

GObject 实例拥有其内存。当它创建时被系统分配。如果他们不被使用，其内存应该被释放。然而，我们如何决定它是否无用？GObject 系统
提供了引用计数来解决这个问题。

一个实例被创建了并被其他实例或者主程序使用。这意味着实例被引用了。如果实例被 A 和 B 引用，那么他的引用数就是2。这个数字就被称为引用计数。让我们设想这样一个场景:

- A 调用 `g_object_new` 拥有一个实例 G。 一个引用指向G， 所以引用计数是1。
- B 也需要使用G。 B 调用 `g_object_ref` 增加了一个引用计数。现在引用计数是2。
- A 不再使用G。 A 调用 `g_object_unref` 减少一个引用计数。 现在引用计数是1。
- B 不再使用G。 B 调用 `g_object_unref` 减少一个引用计数。 现在引用计数是0。
- 因为引用计数是0了。 G知道没有对象引用它了。G 开启最终流程，G不可见并且内存被释放。

## 类型系统和注册流程

GObject 是一个基本对象。我们通常不使用 GObject 本身。由于 GObject 非常简单，使用他不足以应付多数场景。所以，我们通常使用GObject的子类(descenddant object)， 像是很多的 GtkWidget 类型。可以说这个可衍生性是 GObject 最重要的特性。

这一节会描述如何定义一个 GObject 的子对象。

### 命名约定

这节的例子是一个代表实数的对象。在 C 语言是 dobule 类型来代表实数。

第一步要了解命名约定。一个对象的名称由命名空间和名称组成。比如说 `GObject` 由命名空间 `G` 和名称 `object` 组成。
`GtkWidget`是由命名空间 `Gtk` 和名称 `Widget` 组成。我们用命名空间 `T` 和名称 `Double` 来命名我们新对象。

`TDouble` 是一个对象名。它是GObject的子类, 代表着实数并且他的数字的类型是double。它有一些有用的函数。

### 定义 TDoubleClass 和 TDouble

当我们说到类型，它可能是类型系统中的类型，也可能是C语言中的类型。比如： GObject 是系统类型中的类型名称。char, int 或者 double
是 C 语言类型。当我们在上下文中对类型的定义很明确时，我们直接使用类型。但如果模糊，我们就会称呼为C 类型，或者类型系统中的类型。

TDouble 对象有类和实例。这个类的C类型是 TDoubleClass。他的结构如下:

```c
typedef struct _TDoubleClass TDoubleClass;
struct _TDoubleClass {
  GObjectClass parent_class;
};
```

```c
typedef struct _TDoubleClass TDoubleClass;
struct _TDoubleClass {
  GObjectClass parent_class;
};
```

`_TDoubleClass`是一个C结构标签名， `TDoubleClass` 是 `struct _TDoubleClass`。 `TDoubleClass` 是新创建的 C 类型。

- 用 typedef 去定义一个类类型。
- 第一个成员必须是父类的类结构。

TDoubleClass 没有他自己的成员。

TDouble 的实例的 C 类型是TDouble

```c
typedef struct _TDouble TDouble;
struct _TDouble {
  GObject parent;
  double value;
};
```

这和类型的结构相似。

- 用 typedef 去定义一个实例类型
- 结构体的第一个成员必须是父类实例的结构体。

TDouble 有他自己的成员，"value"。他是TDouble实例的值。要始终保持这种代码约定。

## 创建GObject子类的过程

TDouble 类型的创建过程与创建 GObject 类型类似。

1. 注册 TDouble 类型到类型系统
2. 类型系统为 TDoubleClass 和 TDouble 分配内存。
3. 初始化 TDoubleClass.
4. 初始化 TDouble。

### 注册

通常注册通过像 `G_DECLARE_FINAL_TYPE` 和 `G_DEFINE_TYPE` 这种合适的宏完成。
在glib 2.70之后对 final 类型类使用 `G_DEFINE_FINAL_TYPE` 来取代 `G_DEFINE_TYPE`。
所以，不需要关注注册详情。在这个教程中重点是理解GObject 类型系统。所以第一步是展示不带宏的注册。

类型分为静态和动态两种。即使所有实例被销毁后静态类型也不会销毁他的类(class)。 在最后一个实例销毁后动态类型会销毁他的类。
静态对象的子类也是静态的。 `g_type_register_static` 会注册一个静态类型对象。 `gtype.h` 中的源码

```c
GType
g_type_register_static (GType           parent_type,
                        const gchar     *type_name,
                        const GTypeInfo *info,
                        GTypeFlags      flags);
```

他的参数如下：

- parent_type: 父类型
- type_name: 类型的名称。比如： "TDouble"
- info: 类型信息。 GTypeInfo 结构后面会解释
- flags: 标志。如果这个类型是抽象类型或者抽象值类型，就设置标志。否则设置为0。

由于类型系统维护类型的父子关系。`g_type_register_static` 有一个父类型参数。并且类型系统也保存了类型的信息。在注册之后 `g_type_register_static` 返回新对象的类型。

`GTypeInfo` 结构如下定义:

```c
typedef struct _GTypeInfo  GTypeInfo;

struct _GTypeInfo
{
  /* interface types, classed types, instantiated types */
  guint16                class_size;

  GBaseInitFunc          base_init;
  GBaseFinalizeFunc      base_finalize;

  /* interface types, classed types, instantiated types */
  GClassInitFunc         class_init;
  GClassFinalizeFunc     class_finalize;
  gconstpointer          class_data;

  /* instantiated types */
  guint16                instance_size;
  guint16                n_preallocs;
  GInstanceInitFunc      instance_init;

  /* value handling */
  const GTypeValueTable  *value_table;
};
```

这个结构需要在注册前创建

- class_size: 类的大小。比如： TDouble 的类大小是 `sizeof(TDoubleClass)`。
- base_init, base_finalize: 动态成员的初始化和终结过程。在大多数情况下他们不是必须的，设置成 NULL.
- `class_init`: 初始化类的静态成员。把类的初始化函数赋值给 `class_init` 成员。通常约定，名称是 `<namespace>_<name>_class_init`，比如:`t_double_class_init`
- class_finalize: 类的终结流程，由于GObject子类的类型是静态的，他不需要finalize函数。把 NULL 赋值给 `class_finalize` 成员。
- class_data: 用户提供给 `init/finalize` 函数的数据。通常赋值 NULL。
- instance_size: 实例的大小，比如： TDouble 的实例大小是 `sizeof(TDouble)`.
- n_preallocs: 这个可以忽略，用于旧版本的glib
- `instance_init`: 初始化实例成员。把实例初始化函数赋值给 `instance_init` 成员。按照约定，他的名称是 `<namespace>_<name>_init` ,比如说：`t_double_init`。
- value_table: 这只对基本类型有用。如果类型是GObject的子类，就赋值为NULL.

这些信息被类型系统持有，在类创建和销毁时调用。类大小和实例大小信息用来给类和实例分配内存。类和实例初始化函数在类或实例创建时调用。

```c
#include <glib-object.h>

#define T_TYPE_DOUBLE  (t_double_get_type ())

typedef struct _TDouble TDouble;
struct _TDouble {
  GObject parent;
  double value;
};

typedef struct _TDoubleClass TDoubleClass;
struct _TDoubleClass {
  GObjectClass parent_class;
};

static void
t_double_class_init (TDoubleClass *class) {
}

static void
t_double_init (TDouble *self) {
}

GType
t_double_get_type (void) {
  static GType type = 0;
  GTypeInfo info;

  if (type == 0) {
    info.class_size = sizeof (TDoubleClass);
    info.base_init = NULL;
    info.base_finalize = NULL;
    info.class_init = (GClassInitFunc)  t_double_class_init;
    info.class_finalize = NULL;
    info.class_data = NULL;
    info.instance_size = sizeof (TDouble);
    info.n_preallocs = 0;
    info.instance_init = (GInstanceInitFunc)  t_double_init;
    info.value_table = NULL;
    type = g_type_register_static (G_TYPE_OBJECT, "TDouble", &info, 0);
  }
  return type;
}

int
main (int argc, char **argv) {
  GType dtype;
  TDouble *d;

  dtype = t_double_get_type (); /* or dtype = T_TYPE_DOUBLE */
  if (dtype)
    g_print ("Registration was a success. The type is %lx.\n", dtype);
  else
    g_print ("Registration failed.\n");

  d = g_object_new (T_TYPE_DOUBLE, NULL);
  if (d)
    g_print ("Instantiation was a success. The instance address is %p.\n", d);
  else
    g_print ("Instantiation failed.\n");
  g_object_unref (d); /* Releases the object d. */

  return 0;
}
```

- 16-22行：类的初始化函数和实例初始化函数。参数 `class` 指向类结构，参数 `self` 指向实例结构。他们什么都没做但是对于注册是必需的。
- 24-43行：`t_double_get_type` 函数。这个函数返回 TDouble 对象的类型。函数的名称通常是 `<name_space>_<name>_get_type`。并且 `<NAME_SPACE>_TYPE_<NAME>` 形式的宏会把这个函数替换。第三行中。`T_TYPE_DOUBLE` 是一个被`t_double_get_type()`替换的宏。这个函数有一个静态变量 `type` 保存对象的类型。在第一次调用这个函数时,type是0。然后他调用 `g_type_register_static` 把对象注册到类型系统。在第二次和接下来的调用中，这个函数直接返回 `type`,由于静态变量 `type` 通过`g_type_register_static`被赋值成一个非零值并持有这个值。
- 30-40行: 设置info结构体并调用 `g_type_register_static`。
- 45-64行：主函数。获得TDouble对象的类型并展示他。函数 `g_object_new` 用于实例化对象。GObject API 手册说这个函数返回GObject实例的指针但他实际上返回 gpointer。 Gpointer 与 `void *` 一样可以赋值给任意类型的指针。所以表达式 `d = g_object_new (T_TYPE_DOUBLE, NULL); ` 是正确的。如果 `g_object_new` 返回了 `GObject *` ,他就需要把返回的指针进行转换。在创建之后他展示了实例的地址。最后，通过函数 `g_object_unref` 释放并销毁实例。。

### G_DEFINE_TYPE 宏

上面的注册流程总是遵循相同的流程,因此他能被定义成 `G_DEFINE_TYPE` 这样的宏。

`G_DEFINE_TYPE` 做如下流程:

- **声明**一个类初始化函数。他的名称是 `<namespace>_<name>_class_init`。比如：如果对象名称是 `TDouble`。他就是 `t_double_class_init`。这是个申明而不是定义，需要定义它。
- **声明**一个实例初始化函数。他的名称是 `<namespace>_<name>_init`.比如：如果对象名称是 `TDouble`。他就是 `t_double_init`。这是个申明而不是定义，需要定义它。
- **定义**一个静态变量指向他的父类。他的名称是 `<namespace>_<name>_parent_class`。比如： 如果对象名称是 `TDouble`，他就是`t_double_parent_class`。
- **定义**一个 `<namespace>_<name>_get_type()` 函数，比如：如果对象名称是TDouble, 他就是`t_double_get_type`。跟上节一样，注册流程在这个函数中完成。

用这个宏可以简化程序代码。

```c
#include <glib-object.h>

#define T_TYPE_DOUBLE  (t_double_get_type ())

typedef struct _TDouble TDouble;
struct _TDouble {
  GObject parent;
  double value;
};

typedef struct _TDoubleClass TDoubleClass;
struct _TDoubleClass {
  GObjectClass parent_class;
};

G_DEFINE_TYPE (TDouble, t_double, G_TYPE_OBJECT)

static void
t_double_class_init (TDoubleClass *class) {
}

static void
t_double_init (TDouble *self) {
}

int
main (int argc, char **argv) {
  GType dtype;
  TDouble *d;

  dtype = t_double_get_type (); /* or dtype = T_TYPE_DOUBLE */
  if (dtype)
    g_print ("Registration was a success. The type is %lx.\n", dtype);
  else
    g_print ("Registration failed.\n");

  d = g_object_new (T_TYPE_DOUBLE, NULL);
  if (d)
    g_print ("Instantiation was a success. The instance address is %p.\n", d);
  else
    g_print ("Instantiation failed.\n");
  g_object_unref (d);

  return 0;
}
```

由于 `G_DEFINE_TYPE` 可以从编写烦人的 `GTypeInfo` 和 `g_type_register_static` 这些代码中解放了。需要重点关注的是初始化函数的命名规范。在Glib 2.70之后对于 final 类型可以用 `G_DEFINE_FINAL_TYPE` 来取代 `G_DEFINE_TYPE`。

### G_DECLARE_FINAL_TYPE 宏

另一个有用的宏是 `G_DECLARE_FINAL_TYPE` 宏。这个宏用于 final 类型。一个final 类型没有任何子类。如果一个类型有子类，他就是一个可派生类型。

如果你要定义一个可派生类型对象，使用 `G_DECLARE_DERIVABLE_TYPE` 来替换。然而大多数情况下只需要使用final类型。

`G_DECLARE_FINAL_TYPE` 做如下事情:

- **申明** `<name_space>_<name>_get_type()` 函数。这仅仅是一个申明，你需要定义它。但是你可以用 `G_DEFINE_TYPE`,它的扩展包括该函数的定义。因此，实际上不用写定义。
- 对象的 C 类型定义为结构体的typedef。比如：对象名称是 `TDouble` ,那么 `typedef struct _TDouble TDouble`将会包含在拓展中。但是你需要在使用宏 `G_DEFINE_TYPE` 之前定义结构体 `struct _TDboule`。
- `<NAMESPACE>_<NAME>` 宏被定义。比如： 如果对象是 `TDouble` 这个宏就是`T_DOUBLE`。他会被拓展成一个函数用来把参数的指针强转成对象，比如：`T_DOUBLE(obj)` 会把 obj 强转成 `TDouble *`。
- `<NAMESPACE>_IS_<NAME>` 宏被定义。 比如：对象是 `TDouble` 这个宏就是 `T_IS_DOUBLE`。他会被拓展成检查参数是不是TDouble实例的指针。如果参数直接是 TDouble 的子类就会返回true.
- 类结构会被定义。final 类型对象不需要有自己的类结构成员。其定义类似于

```c
typedef struct _TDoubleClass TDoubleClass;
struct _TDoubleClass {
  GObjectClass parent_class;
};

```

需要在 `G_DECLARE_FINAL_TYPE` 之前写对象类型的宏定义。如果类型是 TDouble ,那么

```c
#define T_TYPE_DOUBLE  (t_double_get_type ())
```

需要在 `G_DECLARE_FINAL_TYPE` 前被定义。

对于下面代码，简单总结一下就是相较于原始版本
`G_DEFINE_TYPE` 宏实现了 t_double_get_type() 函数的定义， `G_DECLARE_FINAL_TYPE` 宏实现了类的定义和一些有用的宏定义。

自定义的宏放在最前，然后是 G_DECLARE_FINAL_TYPE， 最后是 G_DEFINE_TYPE。

```c
#include <glib-object.h>

#define T_TYPE_DOUBLE  (t_double_get_type ())
G_DECLARE_FINAL_TYPE (TDouble, t_double, T, DOUBLE, GObject)

struct _TDouble {
  GObject parent;
  double value;
};

G_DEFINE_TYPE (TDouble, t_double, G_TYPE_OBJECT)

static void
t_double_class_init (TDoubleClass *class) {
}

static void
t_double_init (TDouble *self) {
}

int
main (int argc, char **argv) {
  GType dtype;
  TDouble *d;

  dtype = t_double_get_type (); /* or dtype = T_TYPE_DOUBLE */
  if (dtype)
    g_print ("Registration was a success. The type is %lx.\n", dtype);
  else
    g_print ("Registration failed.\n");

  d = g_object_new (T_TYPE_DOUBLE, NULL);
  if (d)
    g_print ("Instantiation was a success. The instance address is %p.\n", d);
  else
    g_print ("Instantiation failed.\n");

  if (T_IS_DOUBLE (d))
    g_print ("d is TDouble instance.\n");
  else
    g_print ("d is not TDouble instance.\n");

  if (G_IS_OBJECT (d))
    g_print ("d is GObject instance.\n");
  else
    g_print ("d is not GObject instance.\n");
  g_object_unref (d);

  return 0;
}
```

### 分离头文件和源码

tdoule.h

```c
#pragma once

#include <glib-object.h>

#define T_TYPE_DOUBLE  (t_double_get_type ())
G_DECLARE_FINAL_TYPE (TDouble, t_double, T, DOUBLE, GObject)

gboolean
t_double_get_value (TDouble *self, double *value);

void
t_double_set_value (TDouble *self, double value);

TDouble *
t_double_new (double value);
```

- 头文件是公共的，他对任何文件开放。头文件包含给出类型信息，转换和类型检查的宏和一些公共函数。
- `T_TYPE_DOUBLE` 是公共的， `G_DECLARE_FINAL_TYPE` 拓展成公共定义
- 8-12行是公共定义，用于设置和获取对象的值。他们被称为实例方法。用于面向对象语言。
- 14-15行：对象实例化函数。

tdouble.c

```c
#include "tdouble.h"

struct _TDouble {
  GObject parent;
  double value;
};

G_DEFINE_TYPE (TDouble, t_double, G_TYPE_OBJECT)

static void
t_double_class_init (TDoubleClass *class) {
}

static void
t_double_init (TDouble *self) {
}

gboolean
t_double_get_value (TDouble *self, double *value) {
  g_return_val_if_fail (T_IS_DOUBLE (self), FALSE);

  *value = self->value;
  return TRUE;
}

void
t_double_set_value (TDouble *self, double value) {
  g_return_if_fail (T_IS_DOUBLE (self));

  self->value = value;
}

TDouble *
t_double_new (double value) {
  TDouble *d;

  d = g_object_new (T_TYPE_DOUBLE, NULL);
  d->value = value;
  return d;
}
```

- 3-6行：申明实例结构。由于 `G_DECLARE_FINAL_TYPE` 宏定义拓展了 `typeder struct _TDouble TDouble`,结构体的标签名必须是`_TDouble`.
- 10-16行： 类和实例的初始化函数。当前他们没有任何动作。
- 18-24行： Getter. 参数 value 是指向double类型的指针。对象的 value 赋值给他。如果成功就返回 True。`g_return_val_if_fail` 函数是用来检查参数类型，如果 `self` 参数不是 TDouble 类型，他会输出错误并马上返回 False。这个函数报告给程序员错误，不应该用于运行时报错。函数 g_return_val_if_fail 不能静态类函数中使用，静态类函数是私有的，因为静态函数只能从同一个文件中的函数调用，调用者知道参数的类型。
- 37行： 参数 `T_TYPE_DOUBLE` 扩展为函数 `t_double_get_type ()`。如果这是对t_double_get_type的第一次调用，则将执行类型注册。

## 信号

信号提供对象间通信的一种手段。信号在一些事情发生或完成时发出。

信号编程的步骤如下：

1. 注册一个信号。 信号属于对象，所以注册是在对象的类初始化函数中完成的。
2. 编写一个信号处理。当信息发出时信号处理函数被调用。
3. 关联信号和处理函数。信号通过 `g_connect_signal` 和他家族的函数连接到处理函数。
4. 发出信息

步骤一和步骤四是在信号所属的对象上完成的。步骤三通常在对象之外完成。

### 信号注册

在本章的例子是在 `division-by-zero` 发生时信号发出。首先需要确定名称，信号名称由字母数字，连接号， 下划线组成。第一个字符必须是字母。

有四个函数去注册信号。这里是使用 `g_signal_new` 注册 "div-by-zero" 信号。

```c
guint
g_signal_new (const gchar *signal_name,
              GType itype,
              GSignalFlags signal_flags,
              guint class_offset,
              GSignalAccumulator accumulator,
              gpointer accu_data,
              GSignalCMarshaller c_marshaller,
              GType return_type,
              guint n_params,
              ...);
```

在 `tdouble.c` 中使用的参数解释

```c
t_double_signal =
g_signal_new ("div-by-zero",
              G_TYPE_FROM_CLASS (class),
              G_SIGNAL_RUN_LAST | G_SIGNAL_NO_RECURSE | G_SIGNAL_NO_HOOKS,
              0 /* class offset.Subclass cannot override the class handler (default handler). */,
              NULL /* accumulator */,
              NULL /* accumulator data */,
              NULL /* C marshaller. g_cclosure_marshal_generic() will be used */,
              G_TYPE_NONE /* return_type */,
              0     /* n_params */
              );
```

- `t_double_signal` 是一个静态guint 变量。guint 类型与 `unsigned int` 一样。它被设置为函数 g_signal_new 返回的信号 id。
- 第二个参数是信号所属对象的类型(GType) , `G_TYPE_FROM_CLASS (class)` 返回类对应的类型。`class` 参数是指针指向对象的类型。
- 第三个参数是信号标志。需要很多页来解释这个标志。所以，我现在就想把他们排除在外。上面的标志设置可以在许多情况下使用。
- 返回值是 `G_TYPE_NONE` 意味着没有返回值
- `n_params` 是参数的数量，这个信号没有参数所以设置为0

这个函数位于类的初始化函数中(t_double_class_init)

可以用另一个函数类似于`g_singal_newv`。

### 信号处理

信号处理函数在信号发出时调用。他有两个参数。

- 信号从属的实例
- 一个指针，改指针指向给信号连接中的用户数据 (对应 n_params 及之后的)

“div-by-zero" 信号不需要用户数据

```c
void div_by_zero_cb (TDouble *self, gpointer user_data) { ... ... ...}
```

第一个参数`self` 是信号发出的实例，可以省略第二个参数。

```c
void div_by_zero_cb (TDouble *self) { ... ... ...}
```

如果一个信号有参数，参数在实例和用户数据之间。比如：GtkApplication 中的 "window-added" 信号处理是：

```c
void window_added (GtkApplication* self, GtkWindow* window, gpointer user_data);
```

第二个参数 `window` 是信号的参数。这个 "window-added" 信号在应用一个新窗口被添加时调用。参数 `window` 指向新增加的window。

"div-by-zero" 信号处理函数现在设置为展示错误信息。

```c
static void
div_by_zero_cb (TDouble *self, gpointer user_data) {
  g_print ("\nError: division by zero.\n\n");
}
```

### 信号连接

信号和处理函数通过`g_signal_connect` 连接

```c
g_signal_connect (self, "div-by-zero", G_CALLBACK (div_by_zero_cb), NULL);
```

- `self` 是信号从属的实例
- 第二个参数是信号名称
- 第三个函数是信号的处理函数，他必须被强转成`G_CALLBACK`
- 最后一个参数是用户数据。这个信号不需要用户数据，所以设置为NULL.

### 信号发射

信号在对象中发射。

```c
TDouble *
t_double_div (TDouble *self, TDouble *other) {
... ... ...
  if ((! t_double_get_value (other, &value)))
    return NULL;
  else if (value == 0) {
    g_signal_emit (self, t_double_signal, 0);
    return NULL;
  }
  return t_double_new (self->value / value);
}
```

如果除数为0， 信号就会发射。`g_signal_emit` 有三个参数

- 第一个参数发射信号的实例
- 第二个参数是信号的id，信号id通过 g_signal_new 返回。
- 第三个参数是详情。 "dev-by-zero" 没有详情，所以这个参数是0.

### 完整示例

tdouble.h

```c
#pragma once

#include <glib-object.h>

#define T_TYPE_DOUBLE  (t_double_get_type ())
G_DECLARE_FINAL_TYPE (TDouble, t_double, T, DOUBLE, GObject)

/* getter and setter */
gboolean
t_double_get_value (TDouble *self, double *value);

void
t_double_set_value (TDouble *self, double value);

/* arithmetic operator */
/* These operators create a new instance and return a pointer to it. */
TDouble *
t_double_add (TDouble *self, TDouble *other);

TDouble *
t_double_sub (TDouble *self, TDouble *other);

TDouble *
t_double_mul (TDouble *self, TDouble *other);

TDouble *
t_double_div (TDouble *self, TDouble *other);

TDouble *
t_double_uminus (TDouble *self);

/* create a new TDouble instance */
TDouble *
t_double_new (double value);
```

tdouble.h

```c
#include "tdouble.h"

static guint t_double_signal;

struct _TDouble {
  GObject parent;
  double value;
};

G_DEFINE_TYPE (TDouble, t_double, G_TYPE_OBJECT)

static void
t_double_class_init (TDoubleClass *class) {
  t_double_signal = g_signal_new ("div-by-zero",
                                 G_TYPE_FROM_CLASS (class),
                                 G_SIGNAL_RUN_LAST | G_SIGNAL_NO_RECURSE | G_SIGNAL_NO_HOOKS,
                                 0 /* class offset.Subclass cannot override the class handler (default handler). */,
                                 NULL /* accumulator */,
                                 NULL /* accumulator data */,
                                 NULL /* C marshaller. g_cclosure_marshal_generic() will be used */,
                                 G_TYPE_NONE /* return_type */,
                                 0     /* n_params */
                                 );
}

static void
t_double_init (TDouble *self) {
}

/* getter and setter */
gboolean
t_double_get_value (TDouble *self, double *value) {
  g_return_val_if_fail (T_IS_DOUBLE (self), FALSE);

  *value = self->value;
  return TRUE;
}

void
t_double_set_value (TDouble *self, double value) {
  g_return_if_fail (T_IS_DOUBLE (self));

  self->value = value;
}

/* arithmetic operator */
/* These operators create a new instance and return a pointer to it. */
#define t_double_binary_op(op) \
  if (! t_double_get_value (other, &value)) \
    return NULL; \
  return t_double_new (self->value op value);

TDouble *
t_double_add (TDouble *self, TDouble *other) {
  g_return_val_if_fail (T_IS_DOUBLE (self), NULL);
  g_return_val_if_fail (T_IS_DOUBLE (other), NULL);
  double value;

  t_double_binary_op (+)
}

TDouble *
t_double_sub (TDouble *self, TDouble *other) {
  g_return_val_if_fail (T_IS_DOUBLE (self), NULL);
  g_return_val_if_fail (T_IS_DOUBLE (other), NULL);
  double value;

  t_double_binary_op (-)
}

TDouble *
t_double_mul (TDouble *self, TDouble *other) {
  g_return_val_if_fail (T_IS_DOUBLE (self), NULL);
  g_return_val_if_fail (T_IS_DOUBLE (other), NULL);
  double value;

  t_double_binary_op (*)
}

TDouble *
t_double_div (TDouble *self, TDouble *other) {
  g_return_val_if_fail (T_IS_DOUBLE (self), NULL);
  g_return_val_if_fail (T_IS_DOUBLE (other), NULL);
  double value;

  if ((! t_double_get_value (other, &value)))
    return NULL;
  else if (value == 0) {
    g_signal_emit (self, t_double_signal, 0);
    return NULL;
  }
  return t_double_new (self->value / value);
}
TDouble *
t_double_uminus (TDouble *self) {
  g_return_val_if_fail (T_IS_DOUBLE (self), NULL);

  return t_double_new (-self->value);
}

TDouble *
t_double_new (double value) {
  TDouble *d;

  d = g_object_new (T_TYPE_DOUBLE, NULL);
  d->value = value;
  return d;
}
```

main.c

```c
#include <glib-object.h>
#include "tdouble.h"

static void
div_by_zero_cb (TDouble *self, gpointer user_data) {
  g_printerr ("\nError: division by zero.\n\n");
}

static void
t_print (char *op, TDouble *d1, TDouble *d2, TDouble *d3) {
  double v1, v2, v3;

  if (! t_double_get_value (d1, &v1))
    return;
  if (! t_double_get_value (d2, &v2))
    return;
  if (! t_double_get_value (d3, &v3))
    return;

  g_print ("%lf %s %lf = %lf\n", v1, op, v2, v3);
}

int
main (int argc, char **argv) {
  TDouble *d1, *d2, *d3;
  double v1, v3;

  d1 = t_double_new (10.0);
  d2 = t_double_new (20.0);
  if ((d3 = t_double_add (d1, d2)) != NULL) {
    t_print ("+", d1, d2, d3);
    g_object_unref (d3);
  }

  if ((d3 = t_double_sub (d1, d2)) != NULL) {
    t_print ("-", d1, d2, d3);
    g_object_unref (d3);
  }

  if ((d3 = t_double_mul (d1, d2)) != NULL) {
    t_print ("*", d1, d2, d3);
    g_object_unref (d3);
  }

  if ((d3 = t_double_div (d1, d2)) != NULL) {
    t_print ("/", d1, d2, d3);
    g_object_unref (d3);
  }

  g_signal_connect (d1, "div-by-zero", G_CALLBACK (div_by_zero_cb), NULL);
  t_double_set_value (d2, 0.0);
  if ((d3 = t_double_div (d1, d2)) != NULL) {
    t_print ("/", d1, d2, d3);
    g_object_unref (d3);
  }

  if ((d3 = t_double_uminus (d1)) != NULL && (t_double_get_value (d1, &v1)) && (t_double_get_value (d3, &v3))) {
    g_print ("-%lf = %lf\n", v1, v3);
    g_object_unref (d3);
  }

  g_object_unref (d1);
  g_object_unref (d2);

  return 0;
}
```

### 默认信号处理

你可能觉得在main.c设置错误信息很奇怪。实际上，错误发生在 `tdouble.c` 中。所以这个信息应该在 `tdouble.c` 中管理。
GObject 系统有一个默认的信号处理可以在对象中设置。也被称为默认处理器和对象方法处理器。可以通过 `g_signal_new_class_handler` 设置。

```c
guint
g_signal_new_class_handler (const gchar *signal_name,
                            GType itype,
                            GSignalFlags signal_flags,
                            GCallback class_handler, /*default signal handler */
                            GSignalAccumulator accumulator,
                            gpointer accu_data,
                            GSignalCMarshaller c_marshaller,
                            GType return_type,
                            guint n_params,
                            ...);

```

与 `g_signal_new` 的区别在第四个参数。`g_signal_new` 使用类结构中函数指针的偏移量设置一个默认处理程序。 如果一个对象是可派生的，他拥有自己的类区域，因此你可以通过 `g_signal_new` 设置一个默认处理函数。但一个 final 类型没有他的类区域(指 TDoubleClass 只有父类成员， 没有其他成员)， 所以不可能通过 g_signal_new 设置默认处理函数。所以我们需要`g_signal_new_class_handler`。

在 `tdouble.c` 文件中这样修改。 `div_by_zero_default_cb` 被添加，并且用 `g_signal_new_class_handler` 替换 `g_signal_new`。默认信号处理程序没有用户数据参数。当用户连接他们的信号处理函数和信号时, user_data 参数在 g_signal_connect 家族函数中设置。默认信号处理函数被他们的实例管理，而不是用户，所以没有提供
用户数据作为参数。

```c
static void
div_by_zero_default_cb (TDouble *self) {
  g_printerr ("\nError: division by zero.\n\n");
}

static void
t_double_class_init (TDoubleClass *class) {
  t_double_signal =
  g_signal_new_class_handler ("div-by-zero",
                              G_TYPE_FROM_CLASS (class),
                              G_SIGNAL_RUN_LAST | G_SIGNAL_NO_RECURSE | G_SIGNAL_NO_HOOKS,
                              G_CALLBACK (div_by_zero_default_cb),
                              NULL /* accumulator */,
                              NULL /* accumulator data */,
                              NULL /* C marshaller */,
                              G_TYPE_NONE /* return_type */,
                              0     /* n_params */
                              );
}
```

如果你想通过用户提供处理函数依然可以通过使用 `g_signal_connect` 。

如果你想要在默认处理器之后调用你的处理器可以使用 `g_signal_connect_after` 来限定先后关系。

### 信号标志

符号标志是用来决定处理函数的顺序的。

- G_SIGNAL_RUN_FIRST: 默认处理器在其他任何用户提供的处理器之前。
- G_SIGNAL_RUN_LAST: 默认处理器在任何普通处理器之后（不能通过 `g_signal_connect_after` 来连接)。
- G_SIGNAL_RUN_CLEANUP: 默认处理器在任何用户提供的处理器之后。

`G_SIGNAL_RUN_LAST` 在大多数情况下倾向使用。
