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

一个例子：

```bash

# GCC
# Please consult /usr/share/portage/config/make.conf.example for a more
COMMON_FLAGS="-march=native -O2 -pipe"
# 无论何种intel、amd的CPU，且无论何种新老CPU架构，均建议-march=native，CPU指令集自动识别全面
# 程序员的用户注意了，“-fomit-frame-pointer"这一项会导致你编译出来的程序无法debug；
# 不做程序开发或debug的普通用户可以放心开启。

CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
FCFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"
MAKEOPTS="-j4"
# 推荐值为CPU中核心/逻辑处理器的数量，可用lscpu命令查看，结果为“CPU(s):”后面的数字

# CPU_FLAGS_X86="aes avx avx2 f16c fma3 mmx mmxext pclmul popcnt sse sse2 sse3 sse4_1 sse4_2 ssse3"
# cpu参数用cpuid2cpuflags命令看，这里先不用管，暂时注释掉，后面再来配置

EMERGE_DEFAULT_OPTS="--with-bdeps=y --ask --verbose=y --load-average --keep-going --deep"
# 这个设置的目的是在遇到编译错误的时候不要停止，而是继续编译下去

# NOTE: This stage was built with the bindist Use flag enabled
PORTDIR="/var/db/repos/gentoo"
DISTDIR="/var/cache/distfiles"
PKGDIR="/var/cache/binpkgs"
PORTAGE_TMPDIR="/tmp"
# 大内存(8G、16G) 设置 小于 4G内存不用设置。

# This sets the language of build output to English.
# Please keep this setting intact when reporting bugs.
LC_MESSAGES=C

MINUS="-bindist -mdev -systemd -consolekit -bluetooth -gtk -netifrc -oss -gpm -iptables"
DESKTOP="-gnome-shell -gnome -gnome-keyring -wayland X cjk"
AUDIO="-pulseaudio alsa jack"
VIDEO="vulkan nvidia"
COMPILE="fortran lto pgo openmp minizip"
ELSE="sudo ccache aria2"
# USE="plugins ${MINUS} ${DESKTOP} ${AUDIO}  ${VIDEO} ${COMPILE} ${ELSE}"
# 建议不要在 make.conf 中定义 USE 去 /etc/portage/package.use/ 中定义。

ACCEPT_LICENSE="*"
ACCEPT_KEYWORDS="amd64"
# "amd64"是使用稳定版的较旧的软件，"~amd64"是使用不稳定版的更新的软件

L10N="en-US zh-CN en zh"
LINGUAS="en-US zh-CN en zh"
AUTO_CLEAN="yes"

GRUB_PLATFORMS="efi-64"
# UEFI 64位系统引导必须项

VIDEO_CARDS="nvidia"
# VIDEO_CARDS="intel i965 iris"
# VIDEO_CARDS="intel i965 iris nvidia"

ALSA_CARDS="hda-intel"
# intel HD声卡
# INPUT_DEVICES="libinput synaptics"
# 笔记本电脑的触控板
MICROCODE_SIGNATURES="-S"
# 如果想把CPU的microcode直接编译进内核，则需要设置为“-S”；否则注释掉

LLVM_TARGETS="X86"

GENTOO_MIRRORS="https://mirrors.ustc.edu.cn/gentoo/"

#FEATURES="ccache"
#CCACHE_DIR="/var/cache/ccache"
#此处先注释掉,配置完ccache后再去掉注释
```

## fstab

分别记录 btrfs 和 xpf 的 fstab 方案

```bash
blkid >> /etc/fstab    #将输出结果追加到fstab配置文件末尾，然后根据追加的内容进行下述修改
```

### btrfs

如果使用 btrfs 文件系统的话，非常推荐如下的挂载选项

`defaults,noatime,space_cache,space_cache=v2,autodefrag,discard=async,ssd,compress=zstd:1`

使用 discard=async 与 fstrim 是不冲突的；另外透明压缩不需要启动等级 3，第一等级就足够了，
如果有需求使用 btrfs 下的 swap ，那不能启用透明压缩功能；并且启用了自动碎片整理功能。这样对文件系统是比较好的。

整个 fstab 的书写就是这样的 UUID 可以使用 blkid 命令查看
`nona /etc/fstab`

```conf
UUID=boot-uuid  /boot       vfat  defaults 0 0
UUID=root-uuid  /           btrfs subvol=@,defaults,noatime,space_cache,space_cache=v2,autodefrag,discard=async,ssd,compress=zstd:1 0 1
UUID=home-uuid  /home       btrfs defaults,noatime,space_cache,space_cache=v2,autodefrag,discard=async,ssd,compress=zstd:1 0 2
UUID=opt-uuid   /opt        btrfs defaults,noatime,space_cache,space_cache=v2,autodefrag,discard=async,ssd,compress=zstd:1,commit=120 0 2
tmpfs           /tmp        tmpfs size=8G,notaime 0 0
tmpfs           /var/tmp    tmpfs size=8G,notaime 0 0
```

最后加了 tmpfs 的内容，建议所有不论你安装什么桌面环境，不论用于什么生产环境，都加上

### xpf

```bash
nano -w /etc/fstab：      #请参考我的配置，建议使用uuid的形式设置（不建议使用/dev/sdX？的形式）(是uuid，而不是partuuid哦)
UUID=......      /boot/efi      vfat      noauto,defaults,noatime,umask=0077                               0 2
UUID=......      /boot          ext4      defaults,noatime,discard                                         0 2
UUID=......      /              xfs       defaults,noatime                                                 0 1
UUID=......      /home          xfs       noatime,discard                                                  0 2
UUID=......      none           swap      sw,noatime,discard                                               0 0
tmpfs            /tmp           tmpfs     rw,nosuid,noatime,nodev,relatime,mode=1777,size=6G               0 0

# 内存tmpfs(/tmp目录)的大小，2G内存设为1G、4G内存设为2G、8G内存可设为4-6G、16G内存可设为10-13G
# 根分区/不建议设置discard参数，你得记得每个星期定期执行一遍"sudo fstrim -v /"命令来优化根分区/
# discard和fstrim都是专门针对SSD固态硬盘的优化，并且你的SSD必须确保支持TRIM；否则在不支持TRIM的SSD上盲目使用discard和fstrim优化很可能会有数据丢失的风险，2017年以后的SSD基本上都支持TRIM了。
```

我的台式机的 ssd 在 2017 年前生产，考虑去掉 discard 和 fstrim
