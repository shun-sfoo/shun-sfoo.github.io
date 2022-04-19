---
title: '极简开发环境的搭建'
date: 2021-12-29T11:11:41+08:00
draft: false
---

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

systemd-boot (recommend)

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

### nvidia

early loading nvidia

[kms](https://wiki.archlinux.org/title/Kernel_mode_setting)

```bash
vim /etc/mkinitcpio.conf
MODULES=(i915? nvidia nvidia_modeset nvidia_uvm nvidia_drm)
mkinitcpio -p linux
```

To use GBM as a wayland backend

[wayland](https://wiki.archlinux.org/title/wayland#Requirements)

[nvidia-drm](https://wiki.archlinux.org/title/NVIDIA#DRM_kernel_mode_setting)

[Wayland_environment](https://wiki.archlinux.org/title/Environment_variables#Wayland_environment)

```bash
vim .pam_enviroment
# or should edit in  ~/.config/environment.d/envvars.conf
# need to checkin
GBM_BACKEND=nvidia-drm
__GLX_VENDOR_LIBRARY_NAME=nvidia
```

pacman hook

```bash
/etc/pacman.d/hooks/nvidia.hook

[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
Target=nvidia
Target=linux
# Change the linux part above and in the Exec line if a different kernel is used

[Action]
Description=Update Nvidia module in initcpio
Depends=mkinitcpio
When=PostTransaction
NeedsTargets
Exec=/bin/sh -c 'while read -r trg; do case $trg in linux) exit 0; esac; done; /usr/bin/mkinitcpio -P'
```

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

ssh generator key

`ssh-keygen -t rsa -C "youremail@example.com" `

git 使用 ssh 拉取代码

拉取子项目

`git clone --recurse-submodules git@github.com:shun-sfoo/xmonad.git`

### 常用工具

in the futrue user wayland native
waybar wayfire ...
now uses sway as wm

fcitx5

`pacman -S fcitx5-im fcitx5-rime`

yarn 使用国内镜像

```bash
yarn global add yrm
yrm ls
yrm use taobao
yrm test taobao
```

格式化工具

`yarn global add prettier pyright`

docker

```bash
sudo gpasswd -a user docker
sudo systemctl enable docker
```

vnc tools (support wayland)

```bash
sudo pacman -S remmina
sudo pacman -S libvncserver
```

aur-helper

```bash
sudo pacman -S --needed base-devel
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si
```
