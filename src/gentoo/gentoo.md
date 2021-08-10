# Gentoo 安装

## 磁盘分区

使用 sfdisk -z `磁盘名称` (`-z` 选项可以选择文件类型 gpt)

一块硬盘通常分成三个分区

- `/boot` 作为引导
- `/` 根目录
- `swap` 交换分区

有多余的硬盘考虑挂载到 (wiki 中的建议)

- `/home` 多用户
- `/opt` 游戏服务软件
- `/var` 邮件服务器

形式： GPT + UEFI

| 类型 | 大小       | 挂载点            | 格式化        |
| ---- | ---------- | ----------------- | ------------- |
| EFI  | 256M       | /mnt/gentoo/boot  | mkfs.vfat     |
| xfs  | free space | /mnt/gentoo       | mkfs.xfs      |
| xfs  | free space | /ment/gentoo/home | mkfs.xfs      |
| swap | 16G        | swap              | mkswap swapon |

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

为了控制页面长度，配置示例单独放在文件中。

[一份直接 copy 的配置](./make-conf.md#default)

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

### 从网路下载 gentoo ebuild 仓库

`emerge-webrsync`

websync 会将数据库同步到 24 小时之内，`emerge --sync` 会同步到 1 小时之内，

这样做很慢且没有必要。

### 选择 默认 profile

```bash
eselect profile list
eselect profile set X
```

如前言中所说，我搭建的环境会是 X + dwm 或是 wayland + sway 所以不需要桌面端，
选择默认就可以了。

### CPU-FLAGS-X86 设置

针对自己 cpu 优化

```bash
emerge -ask -verbose app-portage/cpuid2cpuflags
cpuid2cpuflags
```

make.conf 中启用 CPU-FLAGS-X86

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

提供 btrfs 和 xfs 两种方案

TODO: 这部分细节待研究

[fstab](./fstab.md)

### 安装必须的文件系统支持，否则无法访问硬盘上的分区

```bash
# emerge --ask sys-fs/btrfs-progs # btrfs
emerge --ask sys-fs/e2fsprogs #ext2、ext3、ext4
emerge --ask sys-fs/xfsprogs #xfs
emerge --ask sys-fs/dosfstools #fat32
emerge --ask sys-fs/ntfs3g #ntfs
emerge --ask sys-fs/fuse-exfat #exfat
emerge --ask sys-fs/exfat-utils #exfat
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
emerge --ask sys-kernel/linux-firmware # 一些额外固件，wifi等
make menuconfig
```

[使用 N 卡禁用 Nouveau 驱动](./core.md)

内核配置部分查看官网 handbook

[menuconfig](https://wiki.gentoo.org/wiki/Handbook:AMD64/Full/Installation#Kernel_configuration_and_compilation)

### 生成 initramfs

```bash
emerge --ask sys-kernel/genkernel
genkernel --install --kernel-config=/path/to/used/kernel.config initramfs
ls /boot/initramfs*
```

## 设置主机名：

```bash
echo hostname=\"matrix\" > /etc/conf.d/hostname
# Set the dns_domain_lo variable to the selected domain name
echo dns_domain_lo=\"homenetwork\" >> d/etc/conf.d/net
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

## 修改密码强度

查看两个配置

/etc/pam.d/passwd

/etc/pam.d/system-auth

后者说明相关配置文件在 /etc/security/passwdqc.conf

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
sudo rc-update add udev sysinit
```

## 网络连接

`ip addr` 查看网卡信息

### 有线 netifrc

```bash
emerge net-misc/netifrc
vim /etc/conf.d/net

# 设置动态分配
config_eth0="dhcp"

cd /etc/init.d
ln -s net.lo net.eth0
rc-update add net.eth0 default
```

### wifi iwd

```bash
emerge -av iwd #wifi
rc-update add iwd default
```

## bootloader

TODO: 目前使用 grub2，后续修改成 efibootmgr

保证 make.conf 中设置了 GRUB-PLATFORMS

```bash
# echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
emerge --ask sys-boot/grub:2
grub-install --target=x86_64-efi --efi-directory=/boot # 注意 efi 文件夹位置
grub-mkconfig -o /boot/grub/grub.cfg
```

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
useradd -m -G users,wheel,audio,video,usb -s /bin/bash larry
passwd larry
```

## 清理及重启

`rm /stage3-*.tar.*`

## 检查

1. `/boot` 下是否有内核文件生成
2. `/etc/fstab` 文件内容是否有误

## 参考链接

[Langley Houge](https://medium.com/@langleyhouge/gentoo%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B%E5%8F%8A%E6%80%BB%E7%BB%93-1db269cfa8c7)

[yangmame](https://blog.yangmame.org/Gentoo%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B.html)

[医学生折腾 Gentoo Linux 记](https://www.zhihu.com/column/c_1271625347856310272)
