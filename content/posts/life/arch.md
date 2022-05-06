+++
title = "Arch Installation"
date = "2021-12-29T11:11:41+08:00"
author = "shun-sfoo"
authorTwitter = "" #do not include @
cover = ""
tags = ["linux", "arch"]
keywords = ["arch", "tutorial"]
description = "systemd 提供了引导工具和网络连接及dhcpd, 使用系统自带可以免除安装grub等工具,减少外部依赖."
showFullContent = false
readingTime = true
+++

## archlinux installation

无线连接

```bash
iwctl
device list
station *device* scan
station *device* get-networks
station *device* connect SSID
```

```bash
# 旧设备无线连接
ip link set $device_name up
iwlist $device_name scan |grep ESSID
wpa_passphrase $net_ssid $password > $net.config
wpa_supplicant -c $net.config -i $device_name
pacman -S wireless-tools wpa_supplicant
```

更新系统时间

`timedatectl set-ntp true`

通过 ssh 链接当前主机（可选）

```bash
passwd
ip -brief address
ssh -o StrictHostKeyChecking=no root@<刚刚查看的 IP 地址>
```

磁盘分区

```bash
cfdisk -z 磁盘
```

格式化

```bash
mkfs.vfat /dev/sda1
mkfs.btrfs -f /dev/sda3
# if have another disk mkfs.btrfs -f /dev/sdb1
mkswap /dev/sda2
```

挂载

[systemd-boot](https://wiki.archlinux.org/title/EFI_system_partition#Typical_mount_points)

```bash
mount /dev/sda3 /mnt
mkdir -p /mnt/boot
# mkdir -p /mnt/home
mount /dev/sda1 /mnt/boot
# mount /dev/sdb1 /mnt/home
swapon /dev/sda2
lsblk -f ## 查看分区情况
```

choose mirror site

`reflector`

move tsinghua ustc mirro to the top

安装系统

`pacstrap /mnt linux linux-firmware linux-headers base base-devel neovim git zsh btrfs-progs`

生成文件系统的表文件

```bash
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

进入新系统

```bash
arch-chroot /mnt
passwd
```

设置时区

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

本地化

```bash
# vim /etc/locale.gen

en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
locale-gen

# vim /etc/locale.conf
LANG=en_US.UTF-8
```

网络配置

```bash
# vim /etc/hostname
arch
```

生成对应的 hosts

```bash
# vim /etc/hosts
127.0.0.1 localhost
::1 localhost
127.0.1.1 arch.localdomain arch
```

引导管理

systemd-boot

```bash
bootctl install
systemctl enable systemd-boot-update.service
pacman -S intel-ucode
```

configuration loader.conf

[systemd-boot configuration](https://wiki.archlinux.org/title/Systemd-boot#Configuration)

```bash
vim /boot/loader/loader.conf
#timeout 3
default 	arch.conf
console-mode 	max
editor 		no
```

configuration entries

```bash
vim /boot/loader/entries/arch.conf
title	Arch Linux
linux	/vmlinuz-linux
initrd	/intel-ucode.img
initrd	/initramfs-linux.img
options	root=/dev/sda3 rw
# enable nvidia-drm
# options	root=/dev/sda3 rw nvidia-drm.modeset=1
# if occur [i915 drm error] add `i915.modset=0 nouveau.modeset=0`
```

有线连接

`ip link set $device up`

使用 system-networkd 配置网络

```bash
/etc/systemd/network/20-wired.network
[Match]
Name=enp3s0

# dhcp 设置
[Network]
DHCP=yes

# 设置静态ip地址（推荐）

[Network]
Address=192.168.1.23/24
Gateway=192.168.1.1
DNS=192.168.1.1 8.8.8.8 114.114.114.114

systemctl enable systemd-networkd.service
systemctl enable systemd-resolved.service
```

iwd dhcp 功能

```bash
# vim /etc/iwd/main.conf
[General]
EnableNetworkConfiguration=true
[Network]
NameResolvingService=systemd
```

添加用户

`useradd --create-home neo`

设置密码

`passwd neo`

设置用户组

`usermod -aG wheel,users,storage,power,lp,adm,optical neo`

无需密码

`vim /etc/sudoers`

ssh

```bash
pacman -S openssh
systemctl enable sshd
```

重启

```bash
exit
umount /mnt/boot/efi
umount /mnt
reboot
```