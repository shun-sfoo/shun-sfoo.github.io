# Gentoo 安装教程

使用 LiveUSB （推荐 Fedora） 可以直接从磁盘分区开始

更新：心累，还是用 archlinux 😂

## 磁盘分区

形式： GPT + UEFI

| 类型  | 大小       | 挂载点 | 格式化     |
| ----- | ---------- | ------ | ---------- |
| EFI   | 512M       | boot   | mkfs.vfat  |
| btrfs | free space | root   | mkfs.btrfs |

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

## 正式开始

1. 通过 su 获取 root 权限,
2. `mkdir -p /mnt/gentoo`
3. 将 root 挂载到 `/mnt/gentoo` 下 `mount /dev/sda2 /mnt/gentoo`
4. 下载 Stage3 `wget https://mirrors.ustc.edu.cn/gentoo/releases/amd64/autobuilds/current-stage3-amd64-systemd/stage3-amd64-systemd-最新版.tar.xz`
5. 解压 `tar vxpf stage3-amd64-systemd-最新版.tar.xz`
6. `rm stage3-amd64-systemd-最新版.tar.xz`

### 配置 make.confg

```config
# /usr/share/portage/config/make.conf.example

# GCC
COMMON_FLAG="-march=native -O2 -pipe"
CFLAGS="${COMMON_FLAG}"
CXXFLAGS="${CFLAGS}"
CHOST="x86_64-pc-linux-gnu"
CPU_FLAGS_X86="aes avx avx2 fma3 mmx mmxext pclmul popcnt sse sse2 sse3 sse4_1 sse4_2 ssse3"
FCFLAGS="${COMMON_FLAG}"
FFLAGS="${COMMON_FLAG}"

# Portage
LC_MESSAGES=C
USE="-bindist"
MAKEOPTS="-j5"
PORTDIR="/var/db/repos/gentoo"
DISTDIR="/var/cache/distfiles"
PKGDIR="/var/cache/binpkgs"
GENTOO_MIRRORS="https://mirrors.ustc.edu.cn/gentoo/"
EMERGE_DEFAULT_OPTS="--keep-going --with-bdeps=y"
ACCEPT_KEYWORDS="~amd64"
ACCEPT_LICENSE="*"

# Language
L10N="en-US zh-CN en zh"
LINGUAS="en_US zh_CN en zh"

# other
# intel + nvidia
VIDEO_CARDS="intel i965 nvidia"
# amd + nvida
# VIDEO_CARDS="amdgpu radeonsi nvidia"

QEMU_SOFTMMU_TARGETS="alpha aarch64 arm i386 mips mips64 mips64el mipsel ppc ppc64 s390x sh4 sh4eb sparc sparc64 x86_64"
QEMU_USER_TARGETS="alpha aarch64 arm armeb i386 mips mipsel ppc ppc64 ppc64abi32 s390x sh4 sh4eb sparc sparc32plus sparc64"
```

### 选项解释

1. `COMMON_FLAGS=`，这里添加 `-march=native` ，个人认为 native 比特指的 CPU 型号优化的效果更好，GCC 会根据你计算机的处理器集成的算法来优化，个人推荐。
2. `CHOST=` 和 `CPU_FLAGS_X86=`，这里特指牙膏厂的处理器，如果是 AMD 的锐龙，可以在后续构建系统以后，通过安装 cpuid2cpuflags 工具，查看输出填写。
3. `USE=`，这里只添加了一个 `-bindist` ，其余如果有默认的先删掉，保证初次构建一切顺利。
4. `MAKEOPTS=`，根据计算机的虚拟核心数填写。
5. `EMERGE_DEFAULT_OPTS=` 这个设置的目的是在遇到编译错误的时候不要停止，而是继续编译下去。
6. `QEMU_SOFTMMU_TARGETS=` 和`QEMU_USER_TARGETS=`，安装虚拟机 KVM 的需要，如果没有这方面的需求，可以不用填写。

### 配置源镜像

`mkdir -p /mnt/gentoo/etc/portage/repos.conf`
`vi /mnt/gentoo/etc/portage/repos.conf/gentoo.conf`

```config
[gentoo]
location = /usr/portage
sync-type = rsync
sync-uri = rsync://rsync.mirrors.ustc.edu.cn/gentoo-portage/
auto-sync = yes
```

### chroot 和 第一次构建

先了解下什么是 Chroot。以现在的安装为例，
目前运行的软件和内核是 LiveUSB 提供的，根目录是 LiveUSB 的，
而 Gentoo 系统的根目录在 /mnt/gentoo/ ，也没有直接运行的能力，
因为运行环境也不是 Gentoo 系统的，那么下一步可以通过 Chroot 一系列操作，
实现从 LiveUSB 转移到 Gentoo 系统下。

1. 首先复制 DNS 到 Gentoo 系统下：
   `cp - dereference /etc/resolv.conf /mnt/gentoo/etc/`

2. 挂载必要文件系统：

   ```bash
   mount -t proc /proc /mnt/gentoo/proc
   mount - rbind /sys /mnt/gentoo/sys
   mount - make-rslave /mnt/gentoo/sys
   mount - rbind /dev /mnt/gentoo/dev
   mount - make-rslave /mnt/gentoo/dev
   ```

3. 进入 Chroot：

   ```bash
   chroot /mnt/gentoo /bin/bash
   source /etc/profile
   ```

4. 挂载其余所有的分区：

   ```bash
   mount /dev/sda1 /boot

   mount /dev/其他分区 /home （如果有）
   mount /dev/其他分区 /opt (如果有)
   ```

### 第一阶段

1. 快照更新 Profile 然后使用 rsync 同步
   `emerge-webrsync`
   `emerge - sync`

2. 选择 默认 （kde 或 gnome）作为默认 profile

   ```bash
   profile list
   eselect profile X
   emerge -auvDN - with-bdeps=y @world
   ```

3. 现在开始了漫长的编译过程，如果这个时候出现某些依赖无法满足的情况，
   我们可以通过以下几种方法解决：

   ```bash
   emerge -auvDN - with-bdeps=y - autounmark-write @world
   etc-update - automode -3
   emerge -auvDN - with-bdeps=y @world
   ```

4. 确认没有更新

   ```bash
    emerge @preserved-rebuild
    perl-cleaner - all
    emerge -auvDN --with-bdeps=y @world
   ```

#### 配置时区

```bash
echo "Asia/Shanghai" > /etc/timezone
emerge --config sys-libs/timezone-data

echo "en_US.UTF-8 UTF-8 zh_CN.UTF-8 UTF-8" >> /etc/locale.gen

locale-gen

eselect locale list
```

这时，应该能够看到列出的中文，但是目前建议暂时不要用 eselect 选择使用中文

#### fstab

先了解下 fstab，就像一张表，在 Linux 开机的告诉 mount 应该把哪个分区以什么文件系统，
以什么方式挂载到系统对应位置。这里提供一份使用 btrfs 的 fstab 书写方式。

其次，就是挂载时需要注意的挂载选项，无论你是使用 ext4 文件系统，
还是使用 btrfs 文件系统，使用合适的挂载选项有助于最大性能的发挥你的文件系统的性能，
一方面实现快速读取和写入，另一方面最小化文件丢失

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

针对你喜好的文件系统安装相对应的工具

- [x] btrfs `emerge sys-fs/btrfs-progs`
- [ ] xfs `emerge sys-fs/xfsprogs`
- [ ] jfs `emerge sys-fs/jfsutils`

### 杂项处理

1. 网络连接使用 NetworkManager `emerge -av networkmanager`
   如果出现什么依赖需要解决，还是那三板斧：

   ```bash
   emerge - autounmask-write networkmanager
   etc-update - automode -3
   emerge networkmanager
   ```

   默认开机启动
   `systemctl enable NetworkManager`

2. 设置主机名：
   `echo hostname=\"Matrix\" > /etc/conf.d/hostname`

3. 安装系统工具：

   ```bash
   emerge app-admin/sysklogd sys-process/cronie sudo layman grub
   sed -i 's/\# \%wheel ALL=(ALL) ALL/\%wheel ALL=(ALL) ALL/g' /etc/sudoers
   passwd #设置root密码
   ```

4. systemd 再操作一波：

   ```bash
   sed -i 's/\# GRUB_CMDLINE_LINUX=\"init=\/usr\/lib\/systemd\/systemd\"/GRUB_CMDLINE_LINUX=\"init=\/usr\/lib\/systemd\/systemd\"/g' /etc/default/grub
   ln -sf /proc/self/mounts /etc/mtab
   systemd-machine-id-setup
   ```

### 安装内核 (待优化)

不定制自动配置

```bash
emerge -av genkernel
genkernel --menuconfig all
genkernel --install initramfs

make -jX #将 X 替换为你想编译时的线程数
make modules_install
make install
genkernel --install initramfs
```

### 安装 GRUB

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Gentoo
grub-mkconfig -o /boot/grub/grub.cfg
```

如果出现 No space left on device
请运行：

```bash
mount -t efivarfs efivarfs /sys/firmware/efi/efivars
rm /sys/firmware/efi/efivars/dump-*
```

创建用户名并设置密码

```bash
useradd -m -G users,wheel,portage,usb,video #这里换成你的用户名(小写)
passwd #用户名
```

检查

1. `/boot` 下是否有内核文件生成
2. `/etc/fstab` 文件内容是否有误
3. 插着网线
   重启

### 安装显卡

intel + nvidia
`emerge -av x11-drivers/nvidia-drivers x11-drivers/xf86-video-intel xrandr`
`emerge -av xorg-server`

kde
`emerge -av plasma-desktop plasma-nm plasma-pa sddm konsole`

gnome
`emerge -av gnome gnome-desktop gnome-shell gdm gnome-terminal`

## 参考链接

[Langley Houge](https://medium.com/@langleyhouge/gentoo%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B%E5%8F%8A%E6%80%BB%E7%BB%93-1db269cfa8c7)

[yangmame](https://blog.yangmame.org/Gentoo%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B.html)
