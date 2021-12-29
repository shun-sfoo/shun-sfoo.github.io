---
title: '极简开发环境的搭建'
date: 2021-12-29T11:11:41+08:00
draft: true
---

## 杀手级应用

gcc gdb qemu docker

### archlinux 安装

#### 无线连接

```bash
iwctl
device list
station *device* scan
station *device* get-networks
station *device* connect SSID
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
mkfs.xfs -f /dev/sda3
mkswap /dev/sda2
```

挂载

```bash
mount /dev/sda3 /mnt
mkdir -p /mnt/boot/efi
mount /dev/vda1 /mnt/boot/efi
swapon /dev/sda2
lsblk -f ## 查看分区情况
```

安装系统

`pacstrap /mnt linux linux-firmware linux-headers base base-devel neovim git zsh`

生成文件系统的表文件

```bash
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

进入新系统

```bash
arch-chroot /mnt
```

设置时区

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

本地化

```bash
vim /etc/locale.gen

en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
locale-gen

/etc/locale.conf
LANG=en_US.UTF-8
```

网络配置

```bash
vim /etc/hostname
archlinux
```

生成对应的 hosts

```bash
vim /etc/hosts
127.0.0.1 localhost
::1 localhost
127.0.1.1 archlinux.localdomain archlinux
```

`pacman -S grub efibootmgr efivar intel-ucode (iwd)`

````bash
grub-install /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
passwd
输入密码
`

```bash
exit
umount /mnt/boot/efi
umount /mnt
reboot
```

### 有线连接使用 system-networkd 的 dhcp 功能

```bash
/etc/systemd/network/20-wired.network
[Match]
Name=enp3s0

[Network]
DHCP=yes
```

```bash
systemctl start systemd-networkd.service
systemctl enable systemd-networkd.service
systemctl start  systemd-resolved.service
systemctl enable  systemd-resolved.service
```

### iwd dhcp 功能

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
````
