# 前言

在知乎上看到一个医学生做的一系列 gentoo 安装和内核调整的教程，重新对安装 gentoo
产生了兴趣，对一件事没把握最主要是对他不了解，看到这个系列教程有种豁然开朗的感觉
他里面提到的一个终点就是把通过内核配置，把驱动完全加载到了内核，省去 initramfs
然后内核也能压缩到 10m，这样不论运行速度和启动速度都能提升，也符合 gentoo 的为自己的机器
完全配置的想法。 然后是坚决不用 systemd。
我也打算仿照他的记录一下，尽量详细，符合自己的安装习惯，该简化的简化，觉得有深入
价值的就深化一下。
首先说下机器情况，一个台式机是 i5 4590 加 nvidia 组合，由于 sway 不支持 nvidia 驱动，所以在
这个台式机上是 X + dwm 的组合，。
还有一个 nuc8i5 这个可以用 sway 了就不用 X 。
终端模拟器用 alacritty,都有虚拟化的需求用到 kvm。
驱动问题， 台式机直接用 nvidia 闭源驱动，
intel 不需要 xf86-video-intel 好几年没有维护
使用 mesa 如果用 X 的话 还要 xserver-driver X 也没必要全部安装
只用 xserver 然后加上一些小组件。

## Gentoo 的概念

### Portage

Gentoo 的包管理器是 Portage，portage 包含了 emerge 和 ebuild 两部分，
emerge 是一个可执行程序，负责将 ebuild 中的内容按照规则进行编译和安装，
ebuild 这个文件，更像是个脚本，emerge 按照它进行下载源代码，打补丁，
一定的规则编译，然后安装

### USE

这是一个 Gentoo Linux 发行版比较独特的“标签”，
它定义了编译和构建整个系统需要依赖什么，不需要依赖什么，
尽可能让系统简洁，轻快，高效。

### Cflags

这是 GCC 针对源代码编译进行一定优化的标签，通过这些功能，
可以将我们需要的软件源码进行一定级别的优化，然后生成可执行程序，
从而提高运行效率。

### pacakge use mask

#### package.use

目录 `/etc/portage/package.use/` 这个目录里存放自己对每个软件的 USE 自定义，
每个软件包根据自己的喜好进行取舍,通常以该软件名为配置名称
比方说 针对 gnome 配置
定义文件 `/etc/portage/package.use/gnome`
打开 accessibility 功能， 关闭 games 功能
`gnome-base/gnome accessibility`
`gnome-base/gnome-extra-apps -games`

#### pacakge.mask

比方说用了某个 1.1 版本软件会导致编译不过去，指定只使用
1.1 版本以下的，那么就可以把 1.1 以上的版本隐藏起来
以 foo 为例

```bash
vi /etc/portage/package.mask/foo
>= xxx/foo1.1:gentoo  # 隐藏了大于等于1.1版本的软件

```

#### package.accept_keywords/

`/etc/portage/package.accept_keywords/`
用法同上面两个一样，功能是定义版本，在 gentoo 的仓库中
搜索软件可以看到 版本有 9999 和它的稳定版本号。
使用 `~amd64` 可以用最新版本
9999 表示最新的 github 版本。

```bash
vi /etc/portage/package.accept_keywords/neovim
app-eidtors/neovim ~amd64
app-eidtors/neovim-9999 **
```

#### make.conf

`/etc/portage/make.conf`
定义整个系统在构建的过程中按照怎样的规则来进行编译，具有一定的全局属性。
比方说编译器的选择，系统语言的设定，使用什么特性来编译，系统架构，显卡类型等等
