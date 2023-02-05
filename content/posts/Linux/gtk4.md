---
title: "gtk4学习笔记"
date: 2023-02-05T17:54:28+08:00
tags: ["Linux"]
categories: ["编程"]
draft: false
---

# gtk4

[gtk4 tutorial](https://toshiocp.github.io/Gtk4-tutorial/)

模板代码

```c
#include <gtk/gtk.h>

int main(int argc, char **argv) {
  GtkApplication *app;
  int stat;

  app = gtk_application_new("com.github.shun-sfoo.tfe", G_APPLICATION_DEFAULT_FLAGS);
  stat = g_application_run(G_APPLICATION(app), argc, argv);
  g_object_unref(app);
  return stat;
}
```

直接编译运行会得到

Your application does not implement g_application_activate() and has no handlers
connected to the 'activate' signal. It should do one of these.

的错误信息。

### 信号

这个错误说明了

1. GtkApplication 没有实现 g_application_activate()
2. 没有处理程序连接到到 “activate” 信号
3. 需要处理其中的一个问题。

有事件发生时就会发出信号。例如，创建一个窗口，销毁一个窗口，等等。当应用程序被激活或启动时发出“activate”信号。如果信号连接到一个称为信号处理程序或简单处理程序的函数，则在信号发出时调用该函数。

这个流程是:

1. 一些事件发生
2. 如果它被联系到一个存在的信号，则这个信号发出。
3. 如果这个信号连接到一个处理程序，这个处理程序就被调用。

信号定义在object中，例如 “activate”信号属于 GApplication object 他是
GtkApplication object 的父对象。

GApplication object 又是 GObject
的子对象。子对象集成父对象的信号，方法，属性。所以 GtkApplcation Object也有
“activate”信号。

### GtkWindow 和 GtkApplicationWindow

Widget 是一个抽象概念，它包括所有 GUI
接口，如窗口、对话框、按钮、多行文本、容器等

而且 GtkWidget 是一个基对象，所有 GUI 对象都从它派生而来. GtkWindow
是GtkWidget的子对象

```c
static void
app_activate (GApplication *app, gpointer user_data) {
  GtkWidget *win;

  win = gtk_window_new ();
  gtk_window_set_application (GTK_WINDOW (win), GTK_APPLICATION (app));
  gtk_widget_show (win);
}

GtkWidget *
gtk_window_new (void);
```

根据这个定义，它返回一个指向 GtkWidget 的指针，而不是
GtkWindow。它实际上创建了一个新的 GtkWindow 实例(不是 GtkWidget)
，但是返回一个指向 GtkWidget 的指针。但是，指针指向 GtkWidget，同时也指向包含
GtkWidget 的 GtkWindow。

如果希望使用 win 作为指向 GtkWindow 的指针，则需要对其进行强制转换。

`(GtkWindow *) win`

更推荐的方法是使用 `GTK_WINDOW` 宏。

`GTK_WINDOW (win)`

函数 gtk _ window _ set _ application 用于将 GtkWindow 连接到 GtkApplication。

GtkApplication 继续运行，直到相关窗口被销毁。如果没有连接 GtkWindow 和
GtkApplication，GtkApplication 会立即自毁。因为没有窗口连接到
GtkApplication，所以 GtkApplication
不需要等待任何东西。当它自我销毁时，GtkWindow 也会被销毁。

函数 gtk _ widget _ show 用于显示窗口。

**GtkApplicationWindow**

GtkApplicationWindow 是 GtkWindow 的子对象。它有一些额外的功能，可以更好地与
GtkApplication 集成。建议在使用 GtkApplication 时使用它而不是 GtkWindow。

```c
static void app_activate(GApplication *app, gpointer user_data) {
  GtkWidget *win;

  win = gtk_application_window_new(GTK_APPLICATION(app));
  gtk_window_set_title(GTK_WINDOW(win), "demo");
  gtk_window_set_default_size(GTK_WINDOW(win), 400, 300);
  gtk_widget_show(win);
}
```

创建 GtkApplicationWindow 时，需要将 GtkApplication
实例作为参数提供。然后它自动连接这两个实例。因此，不再需要调用 gtk _ window _
set _ application。

### GtkLabel

它是一个包含文本的小部件

函数 GTK _ window _ set _ child (GTK _ WINDOW (win) ，lab)使 label lab 成为窗口
win 的子窗口小部件。

子窗口小部件总是位于屏幕上的父窗口小部件中.

窗口 `win`
没有父部件。我们称这样的窗口为顶层窗口。一个应用程序可以有多个顶级窗口。

### GtkButton

`gtk_button_new_with_label`

### GtkBox

GtkWindow 和 GtkApplicationWindow
只能有一个子级.如果要在一个窗口中添加两个或多个小部件，则需要一个容器小部件。它将两个或多个子窗口小部件排列成一行或一列。

```c
box = gtk_box_new(GTK_ORIENTATION_VERTICAL, 5);
gtk_box_set_homogeneous (GTK_BOX (box), TRUE);
```

切换文本的方式

```c
static void click1_cb(GtkButton *btn, gpointer user_data) {
  const gchar *s;
  s = gtk_button_get_label(btn);

  if (g_strcmp0(s, "Hello.") == 0)
    gtk_button_set_label(btn, "Good-bye");
  else
    gtk_button_set_label(btn, "Hello.");
}
```

`gtk_box_append(GTK_BOX(box), btn1);` 将部件加入到box中

## GtkTextView, GtkTextBuffer and GtkScrolledWindow

GtkTextView 是一个用于多行文本编辑的小部件。 GtkTextBuffer 是一个连接到
GtkTextView 的文本缓冲区

```c
GtkWidget *tv;
GtkTextBuffer *tb;
tv = gtk_text_view_new();
tb = gtk_text_view_get_buffer(GTK_TEXT_VIEW(tv));
gtk_text_buffer_set_text(tb, text, -1);
gtk_text_view_set_wrap_mode(GTK_TEXT_VIEW(tv), GTK_WRAP_WORD_CHAR);
gtk_window_set_child(GTK_WINDOW(win), tv);
```

在创建 GtkTextView 实例时，还将创建一个 GtkTextBuffer 实例并自动连接到
GtkTextView。

```c
GtkWidget *win;
GtkWidget *scr;
gtk_window_set_child(GTK_WINDOW(win), scr);
gtk_scrolled_window_set_child(GTK_SCROLLED_WINDOW(scr), tv);
```

在glib中 g_new g_free 等同于 malloc free

g_new(struct_type, n_struct);

```c
char *s;
s = g_new(char, 10);
struct tuple {int x, y} *t;
t = g_new(struct tuple, 5);2kj
g_free(s);
g_Free(t);
```

g_strdup

```c
char *s;
s = g_strdup ("Hello");
g_free (s);
```

## G_APPLICATION_HANDLES_OPEN flag

[https://docs.gtk.org/gio/flags.ApplicationFlags.html](https://docs.gtk.org/gio/flags.ApplicationFlags.html)

`[G_APPLICATION_DEFAULT_FLAGS](https://docs.gtk.org/gio/flags.ApplicationFlags.html#default_flags)`
意味着不允许带参数。

如果想处理命令行参数，使用`G_APPLICATION_HANDLES_COMMAND_LINE`

`G_APPLICATION_HANDLES_OPEN` 只允许带文件的参数列表。

使用 `G_APPLICATION_HANDLES_OPEN` 时，会有两个信号发出。

1. activate 信号，
2. open 信号， 只有有一个参数

open 信号的处理函数定义如下：

```c
void user_function (GApplication *application,
                   gpointer      files,
                   gint          n_files,
                   gchar        *hint,
                   gpointer      user_data)
```

参数解释：

application 通常是 GtkApplication

files 一个 GFiles 数组。 数组长度是 n_files, 数组类型是 GFile

n_files files元素的数量

hint 调用实例提供的提示 （通常被忽略)

user_data 用户创建信号处理函数时的数据集

gtk 读取文件中的文本信息

```c
if (g_file_load_contents(files[0], NULL, &contents, &length, NULL, NULL)) {
    gtk_text_buffer_set_text(tb, contents, length);
    g_free(contents);
    if ((filename = g_file_get_basename(files[0])) != NULL) {
      gtk_window_set_title(GTK_WINDOW(win), filename);
      g_free(filename);
    }
    gtk_widget_show(win);
  } else {
    if ((filename = g_file_get_path(files[0])) != NULL) {
      g_print("No such file: %s.\n", filename);
      g_free(filename);
    }
    gtk_window_destroy(GTK_WINDOW(win));
  }
```

## GtkNoteBook

GtkNotebook
是一个使用Tab并包含多个子级的容器小部件。显示的子级取决于选择了哪个选项卡。

```c
  GtkWidget* nb;
  GtkWidget* lab;
  GtkNotebookPage* nbp;
  nb = gtk_notebook_new();
  gtk_window_set_child(GTK_WINDOW(win), nb);

  for (i = 0; i < n_files; i++) {
    if (g_file_load_contents(files[i], NULL, &contents, &length, NULL, NULL)) {
      scr = gtk_scrolled_window_new();
      tv = gtk_text_view_new();
      tb = gtk_text_view_get_buffer(GTK_TEXT_VIEW(tv));
      gtk_text_view_set_wrap_mode(GTK_TEXT_VIEW(tv), GTK_WRAP_WORD_CHAR);
      gtk_text_view_set_editable(GTK_TEXT_VIEW(tv), FALSE);
      gtk_scrolled_window_set_child(GTK_SCROLLED_WINDOW(scr), tv);

      gtk_text_buffer_set_text(tb, contents, length);
      g_free(contents);
      if ((filename = g_file_get_basename(files[i])) != NULL) {
        lab = gtk_label_new(filename);
        g_free(filename);
      } else
        lab = gtk_label_new("");
      gtk_notebook_append_page(GTK_NOTEBOOK(nb), scr, lab);
      nbp = gtk_notebook_get_page(GTK_NOTEBOOK(nb), scr);
      g_object_set(nbp, "tab-expand", TRUE, NULL);
    } else if ((filename = g_file_get_path(files[i])) != NULL) {
      g_print("No such file: %s.\n", filename);
      g_free(filename);
    } else
      g_print("No valid file is given\n");
  }
```

`gtk_notebook_append_page(GTK_NOTEBOOK(nb), scr, lab);`

将 GtkScrolledWindow 作为页追加到 GtkNotebook，并使用选项卡
GtkLabel。此时将自动创建一个 GtkNoteBookPage 小部件。

`GtkNotebook -- GtkNotebookPage -- GtkScrolledWindow`

GtkNotebookPage 有一个名为“ tab-expand”的属性集。如果设置为
TRUE，则选项卡将尽可能长地水平展开。如果是
FALSE，则选项卡的宽度由标签的大小决定。

g_object _set 是一个通用函数，用于设置对象中的属性。

## 自定义的新类型

```c
#define TFE_TYPE_TEXT_VIEW tfe_text_view_get_type()
G_DECLARE_FINAL_TYPE(TfeTextView, tfe_text_view, TFE, TEXT_VIEW, GtkTextView)

struct _TfeTextView {
  GtkTextView parent;
  GFile *file;
};

G_DEFINE_TYPE(TfeTextView, tfe_text_view, GTK_TYPE_TEXT_VIEW);

static void tfe_text_view_init(TfeTextView *tv) {}

static void tfe_text_view_class_init(TfeTextViewClass *class) {}

void tfe_text_view_set_file(TfeTextView *tv, GFile *f) { tv->file = f; }

GFile *tfe_text_view_get_file(TfeTextView *tv) { return tv->file; }

GtkWidget *tfe_text_view_new(void) { return GTK_WIDGET(g_object_new(TFE_TYPE_TEXT_VIEW, NULL)); }
```

TfeTextView

1. TfeTextView 分为两个部分。Tfe 和 TextView。Tfe
   称为前缀、命名空间或模块。TextView 称为对象。
2. 有三种不同的标识符模式. TfeTextView (驼峰大小写) ，tfe_text _view
   (用于编写函数)和 TFE_TEXT_VIEW (用于转换指向TfeTextView 类型的指针)。
3. 首先把 tfe_text_view_get_type() 方法定义为 TFE_TYPE_TEXT_VIEW
   宏。这个名称总是以`(前缀)_TYPE_(object)`的全大写的形式，用来取代
   `(前缀)_(object)_get_type()` 全小写的形式。
4. 接着使用 G_DECLARE_FINAL_TYPE
   宏。参数是驼峰大小写的子对象名，带下划线的小写，前缀(大写)，对象(大写加下划线)和父对象名(驼峰大小写)。1
5. 声明结构 `_TfeTextView`
   下划线是必要的。第一个成员是父对象。注意，这不是一个指针，而是对象本身。第二个成员和之后是子对象的成员。TfeTextView
   结构有一个指向 GFile 实例的指针作为成员。
6. 使用`G_DEFINE_TYPE`
   宏。参数是驼峰大小写的子对象名称、带下划线的小写和父对象类型
   `(前缀)_TYPE_(模块)` 。
7. 定义实例 init 函数 tfe_text_view_init(), 通常不用写任何实现。
8. 定义类 init 函数 tfe_text_view_class_init()。 在这个例子中不需要任何实现。
9. 编写要添加的函数代码((tfe_text_view_set_file 和 tfe_text_view_get_file)。tv
   是指向 TfeTextView 对象实例的指针，该实例是用 tag `_TfeTextView` 声明的 C
   结构。
10. 编写一个函数来创建一个实例。名称是 `(前缀)_(object)_new`
    形式。如果父对象函数需要参数，那么这个函数也需要参数。使用 g _ object _ new
    函数创建实例。参数是 `(前缀)_TYPE_(object)`，
    初始属性的列表和NULL。并且返回值强转成 GtkWidget。

### **Close-request 信号**

关闭win前做的保存操作

```c
static gboolean before_close(GtkWindow *win, gpointer user_data) {
  GtkWidget *nb = GTK_WIDGET(user_data);
  GtkWidget *scr;
  GtkWidget *tv;
  GFile *file;
  char *pathname;
  GtkTextBuffer *tb;
  GtkTextIter start_iter;
  GtkTextIter end_iter;1
  char *contents;
  unsigned int n;
  unsigned int i;

  n = gtk_notebook_get_n_pages(GTK_NOTEBOOK(nb));
  for (i = 0; i < n; ++i) {
    scr = gtk_notebook_get_nth_page(GTK_NOTEBOOK(nb), i);
    tv = gtk_scrolled_window_get_child(GTK_SCROLLED_WINDOW(scr));
    file = tfe_text_view_get_file(TFE_TEXT_VIEW(tv));
    tb = gtk_text_view_get_buffer(GTK_TEXT_VIEW(tv));
    gtk_text_buffer_get_bounds(tb, &start_iter, &end_iter);
    contents = gtk_text_buffer_get_text(tb, &start_iter, &end_iter, FALSE);
    if (!g_file_replace_contents(file, contents, strlen(contents), NULL, TRUE, G_FILE_CREATE_NONE, NULL, NULL, NULL)) {
      pathname = g_file_get_path(file);
      g_print("ERROR : Can't save %s.", pathname);
      g_free(pathname);
    }
    g_free(contents);
  }
  return FALSE;
}
```

### 资源文件

gtk4-builder-tool

`gtk4-builder-tool validate <ui file name>` 验证 ui 文件。如果 ui
文件包含一些语法错误，gtk4-Builder-tool 将打印该错误。

`gtk4-builder-tool simplify <ui file name>` 简化 ui 文件并打印结果。如果给定了
`--replace` 选项，它将用简化的 ui 文件替换该 ui 文件。如果 ui
文件指定了一个属性值，但它是默认值，那么它将被删除。一些值被简化了。例如，“
TRUE”和“ FALSE”分别变成“1”和“0”。但是，“ TRUE”或“ FALSE”更适合于维护。

在编译之前检查下ui文件

## GtkBuilder

```c
  GtkBuilder *build;

  build = gtk_builder_new_from_file ("tfe3.ui");
  win = GTK_WIDGET (gtk_builder_get_object (build, "win"));
  gtk_window_set_application (GTK_WINDOW (win), GTK_APPLICATION (app));
  nb = GTK_WIDGET (gtk_builder_get_object (build, "nb"));
  g_object_unref(build);
```

ui文件在运行时需要用到。

### Gresource

使用 Gresource 类似于使用 string。但是 Gresource
是压缩的二进制数据，而不是文本数据。还有一个编译器可以将 ui 文件编译成
Gresource。它不仅可以编译文本文件，也可以编译二进制文件，如图像，声音等。在编译之后，它将它们捆绑到一个
Gresource 对象中。

对于资源编译器 `glib-compile-resources` 来说，xml
文件是必需的，它描述了资源文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<gresources>
  <gresource prefix="/com/github/shun-sfoo/tfe3">
    <file>tfe3.ui</file>
  </gresource>
</gresources>
```

1. Gresources 标记可以包含多个 gresources (gresource 标记)。
2. Gressource 有一个前缀`/com/github/shun-sfoo/tfe3`。
3. 最终指向 `/com/github/shun-sfoo/tfe3/tfe3.ui` 可以添加多个文件。

保存为`tfe3.gresource.xml`

`glib-compile-resources tfe3.gresource.xml --target=resources.c --generate-source`

引用通过

```c
#include "resources.c"
build = gtk_builder_new_from_resource ("/com/github/ToshioCP/tfe3/tfe3.ui");
```

## meson

推荐的做法

```
project('tfe', 'c')

gtkdep = dependency('gtk4')

gnome=import('gnome')
resources = gnome.compile_resources('resources','tfe.gresource.xml')

sourcefiles=files('tfe.c', 'tfetextview.c')

executable('tfe', sourcefiles, resources, dependencies: gtkdep)
```

### 实例与类

实例是结构化的内存。这个结构通过c语言的结构体描述。实例保存着实例的状态。类主要由指向函数的指针组成。这些方法用于对象本身和对象的子对象。

在 dispose处理函数中通常使用 `g_clear_object` 而不是 `g_object_unref`

## 信号

在gtk编程中，每一个对象都是封装好的。并且不推荐使用全局变量因为他们会使程序趋于复杂。我们需要能在对象间通信的方法。有两种方式实现：

1. 方法。比如: `tb = gtk_text_view_get_buffer (tv)` 这个调用通过 tv 得到
   tb,这个调用联系了 tv 和 GtkTextBuffer的的实例。
2. 信号。GApplicaton 对象中的 activate
   信号。当应用启动，信号发出。然后处理函数被信号关联执行了。

方法的调用或者信号关联的处理函数都是在对象之外。他们之间一个不同点在于主动和被动。在方法中是对象被动地回应调用者。在信号中是主动地向处理者发送信号。

Gobject 信号注册，关联和发出。

1. 信号是发出的对象类型注册。通常是对象类初始化完成后注册完成。
2. 信号通过`g_connect_signal` 和它的家族函数关联。这个关联是独立于对象之外的。
3. 当信号发出，关联的处理函数执行。信号在对象的实例中发出。

### 信号注册

在 TfeTextView 中注册两个信号。

1. “change-file”信号。当 `tv→file` 改变时这个信号发出。
2. “open-response” 信号。 `tfe_text_view_open` 函数不能够返回状态，因为他使用了
   GtkFileChooserDialog。使用这个信号来代替通过函数返回值。

定义信号ID

```c
enum {
  CHANGE_FILE,
  OPEN_RESPONSE,
  NUMBER_OF_SIGNALS
};

static guint tfe_text_view_signals[NUMBER_OF_SIGNALS];
```

信号的注册放在类初始化函数中

```c
static void
tfe_text_view_class_init (TfeTextViewClass *class) {
  GObjectClass *object_class = G_OBJECT_CLASS (class);

  object_class->dispose = tfe_text_view_dispose;
  tfe_text_view_signals[CHANGE_FILE] = g_signal_new ("change-file",
                                 G_TYPE_FROM_CLASS (class),
                                 G_SIGNAL_RUN_LAST | G_SIGNAL_NO_RECURSE | G_SIGNAL_NO_HOOKS,
                                 0 /* class offset */,
                                 NULL /* accumulator */,
                                 NULL /* accumulator data */,
                                 NULL /* C marshaller */,
                                 G_TYPE_NONE /* return_type */,
                                 0     /* n_params */
                                 );
  tfe_text_view_signals[OPEN_RESPONSE] = g_signal_new ("open-response",
                                 G_TYPE_FROM_CLASS (class),
                                 G_SIGNAL_RUN_LAST | G_SIGNAL_NO_RECURSE | G_SIGNAL_NO_HOOKS,
                                 0 /* class offset */,
                                 NULL /* accumulator */,
                                 NULL /* accumulator data */,
                                 NULL /* C marshaller */,
                                 G_TYPE_NONE /* return_type */,
                                 1     /* n_params */,
                                 G_TYPE_INT
                                 );
}
```

使用 `g_singal_new` 函数注册类型，通常不需要默认处理函数，如果需要 使用
`g_signal_new_class_handler`
函数。`g_singal_new`函数返回值是signal_id类型为guint 和 unsigned
int一样。这个值用于 g_signal_emit。 `open-response`
有一个参数，参数类型为跟在参数数量后面的 `G_TYPE_INT`

处理函数如下定义：

```c
/* "change-file" signal handler */
void
user_function (TfeTextView *tv,
               gpointer user_data)

/* "open-response" signal handler */
void
user_function (TfeTextView *tv,
               TfeTextViewOpenResponseType response-id,
               gpointer user_data)
```

1. 因为 `change-file` 信号没有参数，这个处理函数的参数为 TfeTextView
   实例和用户数据。
2. 因为 `open-response` 信号有一个参数，这个处理函数的参数为
   TfeTextView实例，信号的parameter和用户数据。
3. tv 是发出信号的对象实例。
4. 用户数据来自于 `g_signal_connect` 的第四个参数。
5. `parameter` 来自于 `g_signal_emit` 的第四个参数。

parameter的定义

```c
/* "open-response" signal response */
enum
{
  TFE_OPEN_RESPONSE_SUCCESS,
  TFE_OPEN_RESPONSE_CANCEL,
  TFE_OPEN_RESPONSE_ERROR
};
```

1. 当 tfe_text_view_open 成功打开一个文件并且读入时，这个parameter
   设置为`TFE_OPEN_RESPONSE_SUCCESS`
2. 当用户取消时，这个值设置为 `TFE_OPEN_RESPONSE_CANCEL`
3. 当有错误出现是，这个值设置为`TFE_OPEN_RESPONSE_ERROR`

### 连接信号

通过 `g_signal_connect` 连接信号，具有相似功能的函数有:
`g_signal_connect_after g_signal_connect_swapped` 。做常用的还是
`g_signal_connect`。

```c
g_signal_connect (GTK_TEXT_VIEW (tv), "change-file", G_CALLBACK (file_changed), nb);

g_signal_connect (TFE_TEXT_VIEW (tv), "open-response", G_CALLBACK (open_response), nb);
```

### 发出信号

信号从实例中发出，实例的类型是
`g_signal_new`的第二个参数。对象类型和信号的关系在信号注册时就被确定了。

```c
g_signal_emit (tv, tfe_text_view_signals[CHANGE_FILE], 0);
g_signal_emit (tv, tfe_text_view_signals[OPEN_RESPONSE], 0, TFE_OPEN_RESPONSE_SUCCESS);
g_signal_emit (tv, tfe_text_view_signals[OPEN_RESPONSE], 0, TFE_OPEN_RESPONSE_CANCEL);
g_signal_emit (tv, tfe_text_view_signals[OPEN_RESPONSE], 0, TFE_OPEN_RESPONSE_ERROR);
```

1. 第一个参数发出信号的实体
2. 第二个参数是信号是信号ID
3. 第三个是信号的详情。`change-file` 信号和 `open-response`
   信号没有详情并且参数是0时没有详情。
4. `change-file` 信号没有paramter,所以没有第四个参数。
5. `open-response` 信号有一个参数，所以第四个参数是 parameter。

`g_return_val_if_fail`
函数用来测试结果和返回一个默认值。他同时会在标准输出或标准错误中打印日志。

GtkFileChooserDialog

`g_object_ref_sink`
把悬浮指针转换成普通指针，如果这个实例没有**悬浮指针**，它只是简单地把指针计数增加1.

### 我的总结

首先定义信号类型，然后在实例类上生成信号描述g_signal_new，别的地方关联g_songal_connect

此时关联的信号还未生效，在某种特性情况下通过g_signal_emit使信号发出。

以tfe中的例子来说，定义了自己的打开信号，注册到打开按钮上，当使用打开按钮，调用gtk的回调函数，在这个回调函数中emit返回自己的信号，然后调用注册在按钮上的回调函数，结果上来看就是自定义好的信号响应了操作。其实是使用在系统的信号调用基础上实现了自己的。

## startup 信号处理

开始信号在GtkApplication实例初始化时发出。那些信号需要在初始化应用时处理？

1. 通过ui文件构建部件
2. 把按钮信号和他们的处理信号关联
3. 设置css样式
