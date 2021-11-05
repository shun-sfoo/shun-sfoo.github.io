---
title: 'Gentoo'
date: 2021-08-22T18:03:26+08:00
draft: false
---

# Gentoo 安装

## 磁盘分区

使用 cfdisk -z `磁盘名称` (`-z` 选项可以选择文件类型 gpt)

一块硬盘通常分成三个分区

- `/boot` 作为引导
- `/` 根目录
- `swap` 交换分区

有多余的硬盘考虑挂载到 (wiki 中的建议)

- `/home` 多用户
- `/opt` 游戏服务软件
- `/var` 邮件服务器

形式： GPT + UEFI

| 类型  | 大小       | 挂载点            | 格式化        |
| ----- | ---------- | ----------------- | ------------- |
| EFI   | 256M       | /mnt/gentoo/boot  | mkfs.vfat     |
| btrfs | free space | /mnt/gentoo       | mkfs.btrfs    |
| btrfs | free space | /ment/gentoo/home | mkfs.btrfs    |
| swap  | 16G        | swap              | mkswap swapon |

目前来说的理解是 `/boot` 是必要的, 一般设置为 256M 格式化是 vfat ，

`/boot/efi` 是挂载在/boot 下 对应多系统的，

如果系统中有 windows 那么不用格式化它，直接挂载到 efi 上

```bash
mkdir -p /mnt/gentoo/boot
mkdir -p /mnt/gentoo/boot/efi
```

## 下载镜像

建议去中科大的镜像站下载 stage3。(ustc)

```bash
cd /mnt/gentoo/
wget http://mirrors.ustc.edu.cn/gentoo/releases/amd64/autobuilds/current-stage3-amd64/stage3-amd64-20210630T214504Z.tar.xz
tar vxpf stage3-amd64-20210630T214504Z.tar.xz
```

## 配置编译选项

一个示例

```bash
COMMON_FLAGS="-march=native -O2 -pipe"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
FCFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"
MAKEOPTS="-j8"

# i7-7700
# CPU_FLAGS_X86="aes avx avx2 f16c fma3 mmx mmxext pclmul popcnt rdrand sse sse2 sse3 sse4_1 sse4_2 ssse3"

# thinpad e480 i5-8250U
# CPU_FLAGS_X86="avx avx2 f16c fma3 mmx mmxext pclmul popcnt rdrand sse sse2 sse3 sse4_1 sse4_2 ssse3"
# cpu参数用cpuid2cpuflags命令看，这里先不用管，暂时注释掉，后面再来配置

# 注意核心数
EMERGE_DEFAULT_OPTS="--with-bdeps=y --ask --verbose=y --load-average --keep-going --deep"
PORTDIR="/var/db/repos/gentoo"
DISTDIR="/var/cache/distfiles"
PKGDIR="/var/cache/binpkgs"


PORTAGE_TMPDIR="/tmp"
# 大内存(8G、16G) 设置 小于 4G内存不用设置。
LC_MESSAGES=C

ACCEPT_LICENSE="*"
ACCEPT_KEYWORDS="~amd64"
# "amd64"是使用稳定版的较旧的软件，"~amd64"是使用不稳定版的更新的软件


# 括号表示根据情况选装
# vdpau vaapi 安装视频播放器需要
NEO_VIDEO="(-)nvidia vulkan vdpau vaapi"

# pluseaudio与alsa基本一致， oss太过古老
NEO_AUDIO="jack libsamplerate alsa -pulseaudio -oss"

NEO_COMPILE="ccache minizip openmp"

# nftables防火墙可以取代了iptables
NEO_NET="-iptables nftables netifrc (-)wifi -networkmanage -dhcpcd"

# elogind 取代了 consolekit
# policykit -> spidemoney -> rust 编译rust极大增加编译时间 使用 rust-bin 就好
NEO_DESKTOP="elogind -policykit -bindist -consolekit -gnome-shell -gnome -gnome-keyring -kde -systemd (-)X (-)wayland (-)bluetooth cjk dbus blkid"

USE="${NEO_VIDEO} ${NEO_AUDIO} ${NOE_COMPLE} ${NEO_NET} ${NEO_DESKTOP}"

L10N="en-US zh-CN en zh"
LINGUAS="en-US zh-CN en zh"
AUTO_CLEAN="yes"

ALSA_CARDS="hda-intel"
# intel HD声卡
# INPUT_DEVICES="libinput synaptics"
# 笔记本电脑的触控板

VIDEO_CARDS="nvidia"
# VIDEO_CARDS="intel i965 iris"
# VIDEO_CARDS="intel i965 iris nvidia"

LLVM_TARGETS="X86"
GENTOO_MIRRORS="https://mirrors.ustc.edu.cn/gentoo/"

GRUB_PLATFORMS="efi-64"
# UEFI 64位系统引导必须项

QEMU_SOFTMMU_TARGETS="arm aarch64 x86_64"
QEMU_USER_TARGETS="arm armeb aarch64 x86_64"
# 虚拟机

#FEATURES="ccache"
#CCACHE_DIR="/var/cache/ccache"
#此处先注释掉,配置完ccache后再去掉注释
```

### gcc 打开 lto pgo 优化

```bash
vim /mnt/gentoo/etc/portage/package.use/gcc
sys-devel/gcc pgo lto
```

### elogind 关闭 policykit

```bash
vim /mnt/gentoo/etc/portage/package.use/elogind
sys-auth/elogind -policykit
```

## 配置源镜像

```bash
mkdir -p /mnt/gentoo/etc/portage/repos.conf
vim /mnt/gentoo/etc/portage/repos.conf/gentoo.conf

[gentoo]
location = /usr/portage
sync-type = rsync
sync-uri = rsync://rsync.mirrors.ustc.edu.cn/gentoo-portage/
auto-sync = yes
```

## chroot

了解下什么是 Chroot。以现在的安装为例，目前运行的软件和内核是 LiveUSB 提供的，

根目录是 LiveUSB 的，而 Gentoo 系统的根目录在 /mnt/gentoo/ ，也没有直接运行的能力，

因为运行环境也不是 Gentoo 系统的，可以通过 Chroot 一系列操作，实现从 LiveUSB 转移到 Gentoo 系统下。

### 复制 DNS 到 Gentoo 系统下：

`cp --dereference /etc/resolv.conf /mnt/gentoo/etc/`

### 挂载必要文件系统：

```bash
mount -t proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
# mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
# mount --make-rslave /mnt/gentoo/dev
```

**注意** `--make-rslave` 操作是安装 systemd 支持时所需要的，所以这里注释掉

### 进入 Chroot：

```bash
chroot /mnt/gentoo /bin/bash
source /etc/profile
mkdir -p /var/db/repos/gentoo
env-update
source /etc/profile
export PS1="(chroot) ${PS1}"
```

## 配置 portage 设置

### 同步 gentoo ebuild 仓库

`emerge-webrsync`

websync 会将数据库同步到 24 小时之内，`emerge --sync` 会同步到 1 小时之内，

这样做很慢且没有必要。

### 选择 默认 profile

```bash
eselect profile list
eselect profile set X
```

选择默认就可以了。

### CPU-FLAGS-X86 设置

针对 cpu 优化

```bash
emerge -ask -verbose app-portage/cpuid2cpuflags
cpuid2cpuflags
```

make.conf 中启用 `CPU_FLAGS_X86`

### 编译安装最新版 gcc

```bash
emerge -av gcc
eselect gcc list
eselect gcc set X
gcc --version
env-update
source /etc/profile
export PS1="(chroot) ${PS1}"
gcc --version
```

### ccache 设置

将编译放在内存中，提升编译速度，保护硬盘。

```bash
emerge --ask dev-util/ccache
mkdir -p /var/cache/ccache
chown root:portage /var/cache/ccache
chmod 2775 /var/cache/ccache
```

编辑 ccache 配置文件 `/var/cache/ccache/ccache.conf` ，内容如下：

```bash
max_size = 100.0G
umask = 002
cache_dir_levels = 3
```

make.conf 中启用 ccache

## 更新系统

`emerge --ask --verbose --update --deep --newuse @world`

### python 3.10 循环依赖

解决方法忽略 3.9 以上的版本

```bash
mkdir package.mask
vim package.mask/pyhon
>=dev-lang/python-3.10
```

## 配置时区

```bash
echo "Asia/Shanghai" > /etc/timezone
emerge --config sys-libs/timezone-data
```

## 配置 locale

```bash
echo "en_US.UTF-8 UTF-8 zh_CN.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" >> /etc/env.d/02locale

eselect locale list
env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
```

这时，应该能够看到列出的中文，但是目前建议暂时不要用 eselect 选择使用中文

## 设置 fstab

提供 btrfs 方案

## fstab

```bash
blkid >> /etc/fstab    #将输出结果追加到fstab配置文件末尾，然后根据追加的内容进行下述修改
```

### btrfs

```conf
UUID="..." /boot  vfat  defaults                                                          0 0
UUID="..." /      btrfs defaults                                                          0 1
UUID="..." /home  btrfs defaults,compress=zstd                                            0 2
tmpfs      /tmp   tmpfs rw,nosuid,noatime,nodev,relatime,mode=1777,size=10G               0 0
```

由于没有经常更换硬盘的需求,使用分区名就可以了。
对文件系统优化的感知不强，目前就用默认配置
内存 tmpfs(/tmp 目录)的大小，2G 内存设为 1G、4G 内存设为 2G、8G 内存可设为 4-6G、16G 内存可设为 10-13G
根分区/不建议设置 discard 参数，你得记得每个星期定期执行一遍"sudo fstrim -v /"命令来优化根分区/
discard 和 fstrim 都是专门针对 SSD 固态硬盘的优化，并且你的 SSD 必须确保支持 TRIM；
否则在不支持 TRIM 的 SSD 上盲目使用 discard 和 fstrim 优化很可能会有数据丢失的风险，2017 年以后的 SSD 基本上都支持 TRIM 了。

### 安装必须的文件系统支持，否则无法访问硬盘上的分区

```bash
emerge --ask e2fsprogs dosfstools btrfs-progs ntfs3g fuse-exfat exfat-utils
# emerge --ask sys-fs/e2fsprogs #ext2、ext3、ext4
# emerge --ask sys-fs/dosfstools #fat32
# emerge --ask sys-fs/btrfs-progs # btrfs
# emerge --ask sys-fs/ntfs3g #ntfs
# emerge --ask sys-fs/fuse-exfat #exfat
# emerge --ask sys-fs/exfat-utils #exfat
```

## 下载内核源码

```bash
emerge --ask sys-kernel/gentoo-sources
eselect kernel list
eselect kernel set 1 # 这一步会将linux版本链接到linux目录
ls -l /usr/src/linux
```

## 配置内核

```bash
emerge --ask sys-apps/pciutils # lspci
emerge --ask sys-kernel/genkernel
cd /usr/src/linux
# linux-firmware  一些额外固件，wifi等 包含在了 genkernel中
```

## 使用默认配置

使用官方的配置可以做到开箱即用

```bash
cp /usr/share/genkernel/arch/x86_64/generated-config /usr/src/linux/
cd /usr/src/linux
cp generated-config 1.config

make menuconfig
load 1.config
# nvidia驱动禁用 nouveau
save as .config
make -j8 && make modules_install
make install
```

内核配置部分查看官网 handbook

[menuconfig](https://wiki.gentoo.org/wiki/Handbook:AMD64/Full/Installation#Kernel_configuration_and_compilation)

### 生成 initramfs

```bash
genkernel --install --kernel-config=/path/to/used/kernel.config initramfs
ls /boot/initramfs*
```

## 设置主机名：

```bash
echo hostname=\"matrix\" > /etc/conf.d/hostname
# Set the dns_domain_lo variable to the selected domain name
echo dns_domain_lo=\"homenetwork\" >> /etc/conf.d/net
```

## hosts

`vim /etc/hosts`

```bash
# This defines the current system and must be set
127.0.0.1     matrix.homenetwork matrix localhost

# Optional definition of extra systems on the network
192.168.0.5   jenny.homenetwork jenny
192.168.0.6   benny.homenetwork benny
```

## 安装系统工具

安装必要的系统日志工具和守护进程工具、文件索引工具、设备管理工具

```bash
emerge sys-apps/ifplugd

emerge --ask app-admin/sysklogd # system logger
rc-update add sysklogd default

emerge --ask sys-process/cronie # cron daemon
rc-update add cronie default

emerge --ask sys-apps/mlocate # file indexing

rc-update add sshd default # remote access

emerge virtual/udev
emerge --oneshot sys-fs/eudev
rc-update add udev sysinit

rc-update add elogind boot

emerge dev-vcs/git
emerge app-shells/zsh
emerge app-portage/gentoolkit
```

## 权限控制

sudo

```bash
emerge --ask sudo
vim /etc/sudoers
```

## 网络连接

`ip addr` 查看网卡信息

### 有线 netifrc

```bash
emerge net-misc/netifrc net-misc/dhcp

vim /etc/conf.d/net

# 设置动态分配
config_enp3s0="dhcp"

# 貌似不用以下步骤也可以正确 dhcp 和连接网络
cd /etc/init.d
ln -s net.lo net.enp3s0
rc-update add net.enp3s0 default

# 重新启动
rc-service net.enp3s0 restart
```

### wifi iwd

```bash
emerge -av iwd #wifi
rc-update add iwd default
```

### NOTE: iwd and netifrc

gentoo iwd wiki 提到的 netifrc 用不能共存的说法
配置一个就禁用另一个。

[iwd](https://wiki.gentoo.org/wiki/Iwd)

## bootloader

TODO: 目前使用 grub2，后续修改成 efibootmgr

保证 make.conf 中设置了 GRUB-PLATFORMS

```bash
# echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
emerge --ask sys-boot/grub:2
grub-install --target=x86_64-efi --efi-directory=/boot # 注意 efi 文件夹位置
grub-mkconfig -o /boot/grub/grub.cfg
```

## 修改密码强度

查看两个配置

/etc/pam.d/passwd

/etc/pam.d/system-auth

后者说明相关配置文件在 `vim /etc/security/passwdqc.conf`

```bash
min=disabled,24,11,8,7  => min=3,3,3,3,3
max=40                  => max=8
passphrase=8
match=4
similar=deny            => permit
random=47
enforce=everyone
retry=3
```

passwd

## 配置用户

日常用户组

| group   | description                                                                 |
| ------- | --------------------------------------------------------------------------- |
| audio   | Be able to access the audio devices.                                        |
| games   | Be able to play games.                                                      |
| portage | Be able to access portage restricted resources.                             |
| usb     | Be able to access USB devices.                                              |
| video   | Be able to access video capturing hardware and doing hardware acceleration. |
| wheel   | Be able to use su.                                                          |

```bash
useradd -m -G users,wheel,audio,video,usb -s /bin/bash bruce
passwd bruce
```

## 检查

1. `/boot` 下是否有内核文件生成
2. `/etc/fstab` 文件内容是否有误

## 清理及重启

`rm /stage3-*.tar.*`
umount 挂载点
`reboot`

## 显卡驱动及 xorg-server

### intel

```bash
sudo emerge x11-base/xorg-drivers
```

已包含了 xorg-server

### nvidia

```bash
sudo emerge x11-drivers/nvidia-drivers
```

**注意**：如果安装后提示 warning（红色"\*"号提示）当前内核配置的“CONFIG_I2C_NVIDIA_GPU=y”这一项不符合要求，
与 nvidia-drivers 冲突，需要按照提示将这一项内核配置禁用，并重新编译安装内核

**注意** ：以后每次重新编译安装内核 kernel 后，均须要运行一遍“emerge @module-rebuild”，重新编译安装 nvidia 驱动模块加载到内核之中，否则 nvidia 驱动无法加载！！！

```bash
lsmod | grep nvidia

sudo rmmod nvidia
sudo  modprobe nvidia

lsmod | grep nvidia


sudo vim /etc/modules-load.d/nvidia.conf:
nvidia

sudo vim /etc/modprobe.d/nvidia-drm.conf：
options nvidia-drm modeset=1


sudo rc-update add modules boot

sudo reboot   #重启系统
```

#### 关于 nvidia 的设置

从最后设置成功的步骤反过来试下到底那些步骤是必要的

1. gentoo wiki 中的 nvidia 的设置

```bash
# /etc/X11/xorg.conf.d/nvidia.conf
Section "Device"
   Identifier  "nvidia"
   Driver      "nvidia"
EndSection
```

2. 下载 polkit (不希望是这一步)

3. 永久禁用 nouveau 驱动模块（nvidia 非官方开源驱动) 这一步貌似必要

```bash
touch /etc/modprobe.d/blacklist.conf

vim /etc/modprobe.d/blacklist.conf：
blacklist nouveau
blacklist lbm-nouveau
options nouveau modeset=0
```

4. 禁用 CONFIG_I2C_NVIDIA_GPU

5. 增加到模块中

```bash
sudo vim /etc/modules-load.d/nvidia.conf:
nvidia

sudo vim /etc/modprobe.d/nvidia-drm.conf：
options nvidia-drm modeset=1
```

### xorg-server

```bash
sudo emerge x11-base/xorg-server
```

### nftables 设置

也可不设置

```bash
sudo nft flush ruleset
sudo nft add table inet filter
sudo nft add chain inet filter input { type filter hook input priority 0 \; policy drop \; }
sudo nft add chain inet filter forward { type filter hook forward priority 0 \; policy drop \; }
sudo nft add chain inet filter output { type filter hook output priority 0 \; policy accept \; }
sudo nft add rule inet filter input iif lo accept
sudo nft add rule inet filter input ct state related,established accept
sudo nft add rule inet filter input ct state invalid drop

sudo nft add rule inet filter input tcp dport 8080 accept  #开放本机8080/tcp端口
sudo nft add rule inet filter input udp dport 8080 accept  #开放本机8080/udp端口

sudo nft add rule inet filter input ssh dport accept # ssh
```

```bash
sudo /etc/init.d/nftables save
sudo /etc/init.d/nftables start
sudo /etc/init.d/nftables reload
sudo /etc/init.d/nftables list
sudo rc-update add nftables default
```

[a simple example](https://gist.github.com/dseg/3e0c4842b0c868e79c527f9f566de636)

### 音视频软件

```bash
media-sound/jack2
media-sound/alsa-utils
media-sound/cmus
media-video/ffmpeg
```

### 容器

使用 podman 取代 docker

```bash
# vim /etc/portage/package.use/podman
app-emulation/podman btrfs

# It is recommended to use app-emulation/crun as the OCI runtime provider
emerge --ask --oneshot app-emulation/crun app-emulation/podman
emerge --ask app-emulation/podman
```

### rust

使用 `/usr/bin/rustup-init-gentoo` 进行系统配置
下载 `rust-bin`
alacritty bat zoxide starship fd riggrep exa

## 参考链接

[Langley Houge](https://litterhougelangley.life/blog/2021/05/21/gentoo/)

[医学生折腾 Gentoo Linux 记](https://www.zhihu.com/column/c_1271625347856310272)
