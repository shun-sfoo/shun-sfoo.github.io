---
title: '极简开发环境的搭建'
date: 2021-12-29T11:11:41+08:00
draft: false
---

由追求完美的环境过渡到实用为主，快即实用。

## 杀手级应用

gcc gdb qemu docker tmux ssh

### archlinux installation

linux 选择 archlinux 作为服务器，mac 远程连接

无线连接

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
archlinux
```

生成对应的 hosts

```bash
# vim /etc/hosts
127.0.0.1 localhost
::1 localhost
127.0.1.1 archlinux.localdomain archlinux
```

启动管理

```bash
pacman -S grub efibootmgr efivar intel-ucode (iwd)
grub-install /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
# 输入密码
passwd
```

重启

```bash
exit
umount /mnt/boot/efi
umount /mnt
reboot
```

有线连接使用 system-networkd 的 dhcp 功能

```bash
/etc/systemd/network/20-wired.network
[Match]
Name=enp3s0

[Network]
DHCP=yes

systemctl start systemd-networkd.service
systemctl enable systemd-networkd.service
systemctl start  systemd-resolved.service
systemctl enable  systemd-resolved.service
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

docker

`gpasswd -a user docker`

格式化工具

`yarn global add prettier pyright lua-fmt-fork`

### bugfix

如果出现 `OpenSSL SSL_connect: SSL_ERROR_SYSCALL in connection github:443`

不确定是不是因为上述网络设置的原因

```bash
# 解决方法，为git设置代理
git config --global http.proxy=127.0.0.1:7890
git config --global https.proxy=127.0.0.1:7890
# 取消设置
git config --global --unset http.proxy
git config --global --unset https.proxy
# 查看所有git配置
git config --global -l
```

### 环境设置

python: pyenv

```bash
curl https://pyenv.run | bash

export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv virtualenv-init -)"
```

rust

`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`

### 常用命令

拉取子项目

`git clone --recurse-submodules https://github.com/chaconinc/MainProject`
