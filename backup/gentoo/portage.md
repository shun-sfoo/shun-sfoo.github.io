# Portage 的概念

Gentoo 的包管理器是 Portage，portage 包含了 emerge 和 ebuild 两部分，

emerge 是一个可执行程序，负责将 ebuild 中的内容按照规则进行编译和安装，

ebuild 这个文件，更像是个脚本，emerge 按照它进行下载源代码，打补丁， 一定的规则编译，然后安装

## USE

这是一个 Gentoo Linux 发行版比较独特的“标签”，它定义了编译和构建整个系统需要依赖什么，不需要依赖什么，
尽可能让系统简洁，轻快，高效。

## Cflags

这是 GCC 针对源代码编译进行一定优化的标签，通过这些功能，
可以将我们需要的软件源码进行一定级别的优化，然后生成可执行程序，
从而提高运行效率。

## package.use

目录 `/etc/portage/package.use/` 这个目录里存放自己对每个软件的 USE 自定义，

每个软件包根据自己的喜好进行取舍,通常以该软件名为配置名称

比方说 针对 gnome 配置:

1. 定义文件 `/etc/portage/package.use/gnome`
2. 打开 accessibility 功能， 关闭 games 功能

```bash
gnome-base/gnome accessibility
gnome-base/gnome-extra-apps -games
```

## pacakge.mask

比方说用了某个 1.1 版本软件会导致编译不过去，指定只使用
1.1 版本以下的，那么就可以把 1.1 以上的版本隐藏起来
以 foo 为例

```bash
vi /etc/portage/package.mask/foo
>= xxx/foo1.1:gentoo  # 隐藏了大于等于1.1版本的软件

```

## package.accept-keywords

`/etc/portage/package.accept_keywords/`
用法同上面两个一样，功能是定义版本，

在 gentoo 的仓库中搜索软件可以看到 版本有 9999 和它的稳定版本号。
使用 `~amd64` 可以用最新版本
9999 表示最新的 github 版本。

```bash
vi /etc/portage/package.accept_keywords/neovim
app-eidtors/neovim ~amd64
app-eidtors/neovim-9999 **
```
