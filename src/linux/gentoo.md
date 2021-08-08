# Gentoo 安装

使用 LiveUSB （推荐 Fedora） 可以直接从磁盘分区开始
好处是可以在终端复制粘贴

## 磁盘分区

使用 sfdisk -z `磁盘名称`
以下都是一块硬盘通常分成三个分区
有多余的硬盘考虑挂在 home opt 等目录
一般是来说 root 也就是 `/` 是挂在到 `/mnt/gentoo`
home 对应的硬盘空间挂载到 `/mnt/gentoo/home`

形式： GPT + UEFI

| 类型 | 大小       | 挂载点 | 格式化        |
| ---- | ---------- | ------ | ------------- |
| EFI  | 256M       | boot   | mkfs.vfat     |
| xfs  | free space | root   | mkfs.xfs      |
| xfs  | free space | home   | mkfs.xfs      |
| swap | 16G        | swap   | mkswap swapon |

目前来说的理解是 `/boot` 是必要的 一般 设置为 256M 格式化是 vfat ，
`/boot/efi` 是挂载在/boot 下 对应多系统的，如果系统中有 windows 那么
不用格式化它，直接挂载到 efi 上
`mkdir -p /mnt/gentoo/boot`
`mkdir -p /mnt/gentoo/boot/efi`

## 正式开始

### 挂载

```bash
mkdir -p /mnt/gentoo
mount /dev/sda3 /mnt/gentoo

mkdir -p /mnt/gentoo/boot
# 如果有多余硬盘挂载到home目录
mkdir -p /mnt/gentoo/home
mount /dev/sda1 /mnt/gentoo/boot
mount /dev/sdb1 /mnt/gentoo/home

# 如果是多系统
mkdir -p /mnt/gentoo/boot/efi
mount /dev/sdXx /mnt/gentoo/boot/efi
```

### 下载镜像解压

- 下载 Stage3 `wget https://mirrors.ustc.edu.cn/gentoo/releases/amd64/autobuilds/current-stage3-amd64/stage3-amd64-最新版.tar.xz`
- 解压 `tar vxpf stage3-amd64-最新版.tar.xz`
- `rm stage3-amd64-最新版.tar.xz`

### 配置 make.confg

为了控制页面长度，配置示例放到了前言中。

[示例](./intro.md)

### gcc 优化

打开 GCC lto 和 pgo 优化，新建 `/mnt/gentoo/etc/portage/package.use/gcc`,
输入以下内容：`sys-devel/gcc pgo lto`

### 配置源镜像

```bash
mkdir -p /mnt/gentoo/etc/portage/repos.conf
vi /mnt/gentoo/etc/portage/repos.conf/gentoo.conf

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

#### 首先复制 DNS 到 Gentoo 系统下：

`cp --dereference /etc/resolv.conf /mnt/gentoo/etc/`

##### 挂载必要文件系统：

```bash
mount -t proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
# mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
# mount --make-rslave /mnt/gentoo/dev
```

注意 `--make-rslave` 操作是安装 systemd 支持时所需要的,所以这里注释掉

#### 进入 Chroot：

```bash
chroot /mnt/gentoo /bin/bash
source /etc/profile
mkdir -p /var/db/repos/gentoo
env-update
source /etc/profile
export PS1="(chroot) ${PS1}"
```

### 第一阶段

#### 快照更新 Profile 然后使用 rsync 同步

`emerge-webrsync`

websync 会将数据库同步到 24 小时之内，`emerge --sync` 会同步到 1 小时之内，这样做很慢且没有必要。

#### 选择 默认 profile

```bash
eselect profile list
eselect profile set X
```

如前言中所说，我搭建的环境会是 X + dwm 或是 wayland + sway 所以不需要桌面端，选择默认就可以了。

#### 确认 CPU_FLAGS_X86

```bash
emerge -ask -verbose app-portage/cpuid2cpuflags
cpuid2cpuflags
```

make.conf 中启用 CPU_FLAGS_X86

#### 安装 ccache

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

#### 更新系统

`emerge --ask --verbose --update --deep --newuse @world`

现在开始了漫长的编译过程，如果这个时候出现某些依赖无法满足的情况，
我们可以通过以下几种方法解决：

```bash
emerge -auvDN --with-bdeps=y --autounmask-write @world
etc-update --automode -3
emerge -auvDN --with-bdeps=y @world
```

pyhon3.9 这个版本需要用 -bluetooth 这个 USE 编译一遍
`USE=-bluetooth emerge -av python`

如果中途因为某个包挂了，可以尝试以下两个命令：

```bash
emerge @preserved-rebuild -j
perl-cleaner --all
```

如果以上还是不能解决问题,则进入/etc/portage 目录
删掉 package.use,package.mask 和 package.unmask 文件或目录再次尝试

#### 配置时区

```bash
echo "Asia/Shanghai" > /etc/timezone
emerge --config sys-libs/timezone-data

echo "en_US.UTF-8 UTF-8 zh_CN.UTF-8 UTF-8" >> /etc/locale.gen

locale-gen

eselect locale list
env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
```

这时，应该能够看到列出的中文，但是目前建议暂时不要用 eselect 选择使用中文

#### fstab

先了解下 fstab，就像一张表，在 Linux 开机的告诉 mount 应该把哪个分区以什么文件系统，
以什么方式挂载到系统对应位置。这里提供一份使用 btrfs 的 fstab 书写方式。

其次，就是挂载时需要注意的挂载选项，无论你是使用 ext4 文件系统，
还是使用 btrfs 文件系统，使用合适的挂载选项有助于最大性能的发挥你的文件系统的性能，
一方面实现快速读取和写入，另一方面最小化文件丢失

[示例](./intro.md#fstab)

#### 安装必须的文件系统支持，否则无法访问硬盘上的分区

```bash

# emerge --ask sys-fs/btrfs-progs # btrfs
emerge --ask sys-fs/e2fsprogs #ext2、ext3、ext4
emerge --ask sys-fs/xfsprogs #xfs
emerge --ask sys-fs/dosfstools #fat32
emerge --ask sys-fs/ntfs3g #ntfs
emerge --ask sys-fs/fuse-exfat #exfat
emerge --ask sys-fs/exfat-utils #exfat
```

### 编译内核

#### 编译内核前，先安装一些必要工具并配置

```bash
emerge xz-utils
echo 'sys-apps/kmod lzma zlib' > /etc/portage/package.use/kmod
emerge --ask --verbose eix sudo pciutils usbutils hwinfo gentoolkit euses kmod layman
```

如果提示需要更新 USE 配置

```bash
etc-update --automode -3
```

```bash
echo "SOLARIZED=true" > /etc/eixrc/99-colour
depmod -a # 由于不是gentoo live cd 这一步不能用
```

#### 用取巧的方式，设置并编译安装 liunx 内核

```bash
emerge --ask sys-kernel/gentoo-sources  # 下载的是以内核作为名称 手动复制下
ls -l /usr/src/linux  #有输出结果表示内核初步下载成功
emerge --ask --verbose sys-kernel/linux-firmware    #时间较长30分钟左右，下载安装wifi网卡和intel核显的必要驱动固件
emerge --ask sys-firmware/intel-microcode sys-apps/iucode_tool   #下载速度慢 需要科学上网，微码的部分可先跳过
iucode_tool -S
iucode_tool -S -l /lib/firmware/intel-ucode/*
# 识别处理器签名并查找相应microcode文件名,其为“06-9e-0d”这样的编号格式，选择最末尾的那个（最新的）
# 建议把找到的结果用手机拍照下来，然后设置编译内核（make menuconfig）的时候
```

```bash
make menuconfig 中
Device Driver:
    Generic Driver Options:
         Firmware loader:
             Build name firmware blobs into the kernel binary

输入“intel-ucode/06-9e-0d”这样的编号格式，将微码文件直接编译进内核。
```

确保 emerge intel-microcode 之前就得在 已经在 make.conf 中设置好了
`MICROCODE_SIGNATURES="-S"`

配置内核时开启

```bash
CONFIG_MICROCODE=y
CONFIG_MICROCODE_INTEL=y
```

#### 永久禁用 nouveau

```bash
vim  /etc/modprobe.d/blacklist.conf：
blacklist nouveau
blacklist lbm-nouveau
options nouveau modeset=0
```

即便在编译内核前就已经设置内核禁用 Nouveau 驱动了，但是内核安装时还是会默认把 nouveau 驱动作为内核模块自动加载。
启用了 nouveau 驱动模块的内核会出现各式各样的莫名其妙的数不清的问题，所以为了避免以后出现这些问题，
从现在就开始永久禁用 nouveau 模块！这是很多教程包括 gentoo wiki 上都不曾提到过的大问题，也是让很多人遭坑的关键地方。

#### 在 genkernel 默认内核基础上修改

将 genkernel 的默认内核配置文件“generated-config”复制过来，
里面已经为你设置好了绝大部分应用场景以及绝大部分硬件驱动的配置，非常方便，值得借过来使用，
只需要在自己手动配置内核的时候将其加载，在其基础上做一点点轻微的修改或完全不修改都可以，
对内核新手极其友好！

```bash
emerge --ask sys-kernel/genkernel
cd /usr/src/linux
cp /usr/share/genkernel/arch/x86_64/generated-config /usr/src/linux/
```

将 generated-config 复制为 1.config 使用，而 generated-config 留作备份

```bash
cp /usr/src/linux/generated-config /usr/src/linux/1.config
```

想在以后支持 jack 低延迟实时音频组件（Jack-Audio-Connection-Kit），则还需要 vim 1.config，手动设置
顺便此时设置上一步提到的微码改动

```bash
# jack
CONFIG_CGROUPS=y
CONFIG_CGROUP_SCHED=y
CONFIG_RT_GROUP_SCHED=y

# mcirocode
CONFIG_MICROCODE=y
CONFIG_MICROCODE_INTEL=y
```

### 内核配置

`make menuconfig`
首先基本配置下

```bash
load 1.config
“Core 2/newer Xeon”，Preemption Model：“Low-Latency Desktop”，Timer frequecy 1000hz，Timer tick handling：“tickless idle”，Cputime accounting：“Simple tick based cputime accounting”，去掉了不需要的文件系统支持，去掉了对AMD CPU的支持，禁用Nouveau驱动）。“Support for extended (non-PC) x86 platforms”这一项取消掉。 之后“Save”你的设置，并且“Exit”即可
```

禁用 Nouveau 驱动
`CONFIG_I2C_NVIDIA_GPU` 禁用

### 编译内核

```bash
make -j4
make modules_install
make install
```

#### 使用 dracut 生成内核的 initramfs (可选)

```bash
emerge --ask sys-kernel/dracut
cd /boot
dracut --hostonly
```

#### 使用 genkernel 生成内核的 initramfs (推荐)

```bash
genkernel --install initramfs
```

### 设置 grub 引导

```bash
emerge --ask sys-boot/grub:2
emerge --ask sys-boot/os-prober

# grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Gentoo  多个系统
grub-install --target=x86_64-efi --efi-directory=/boot--bootloader-id=Gentoo
grub-mkconfig -o /boot/grub/grub.cfg
```

###

#### 网络连接使用 NetworkManager

#### 设置主机名：

`echo hostname=\"Matrix\" > /etc/conf.d/hostname`

#### 设置密码强度

查看两个配置

/etc/pam.d/passwd

/etc/pam.d/system-auth

后者告诉我们相关配置文件在 /etc/security/passwdqc.conf

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

#### 安装系统工具

安装必要的网络工具，系统日志工具和守护进程工具、文件索引工具、电源管理工具、设备管理工具

```bash
emerge -av networkmanager
rc-update add NetworkManager default

emerge --ask app-admin/sysklogd
rc-update add sysklogd default

emerge --ask sys-process/cronie
rc-update add cronie default

emerge --ask sys-apps/mlocate

emerge --ask sys-power/acpid
rc-update add acpid default

emerge sys-power/thermald
rc-update add thermald default

emerge virtual/udev
emerge --oneshot sys-fs/eudev
sudo rc-update add udev sysinit
```

```bash
sed -i 's/\# \%wheel ALL=(ALL) ALL/\%wheel ALL=(ALL) ALL/g' /etc/sudoers
passwd #设置root密码
```

#### 创建用户名并设置密码

```bash
useradd -m -G users,wheel,portage,usb,video,audio  #这里换成你的用户名(小写)
passwd #用户名
```

检查

1. `/boot` 下是否有内核文件生成
2. `/etc/fstab` 文件内容是否有误

## 参考链接

[Langley Houge](https://medium.com/@langleyhouge/gentoo%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B%E5%8F%8A%E6%80%BB%E7%BB%93-1db269cfa8c7)

[yangmame](https://blog.yangmame.org/Gentoo%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B.html)
