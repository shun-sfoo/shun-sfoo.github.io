# archlinux 手动挡

## archlinux installation 安装步骤

### 网络连接

无线连接 (optional)

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

### 在其他机器上远程连接安装

更新系统时间

`timedatectl set-ntp true`

通过 ssh 链接当前主机（可选）

```bash
passwd
ip -brief address
ssh -o StrictHostKeyChecking=no root@<刚刚查看的 IP 地址>
```

### 磁盘操作

磁盘分区

```bash
cfdisk -z 磁盘
```

格式化

```bash
mkfs.vfat /dev/sda1
mkswap /dev/sda2
swapon /dev/sda2
mkfs.ext4 /dev/sda3
# mkfs.ext4 /dev/sdb1
# mkfs.btrfs -f /dev/sda2
# if have another disk mkfs.btrfs -f /dev/sdb1
```

挂载

[systemd-boot](https://wiki.archlinux.org/title/EFI_system_partition#Typical_mount_points)

```bash
mount /dev/sda3 /mnt
mkdir -p /mnt/boot
# mkdir -p /mnt/home
mount /dev/sda1 /mnt/boot
# mount /dev/sdb1 /mnt/home
lsblk -f ## 查看分区情况
```

### 安装系统

```bash
# uncomment /etc/pacman.conf ParallelDownloads
reflector --country china > /etc/pacman.d/mirrorlist
pacman -Sy
pacman -S archlinux-keyring
pacstrap /mnt linux linux-firmware linux-headers base
# btrfs-progs
```

生成文件系统的表文件

```bash
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

进入新系统

```bash
arch-chroot /mnt
passwd
alias vim=nvim
pacman -S sudo neovim
```

设置时区

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

### 本地化

```bash
# vim /etc/locale.gen

en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
locale-gen

# vim /etc/locale.conf
LANG=en_US.UTF-8
```

### 网络配置

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
# 140.82.114.4 github.com
```

### 引导管理

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
# if occur [i915 drm error] add `i915.modeset=0 nouveau.modeset=0`
```

有线连接

`ip link set $device up`

使用 system-networkd 配置网络

```bash
vim /etc/systemd/network/20-wired.network
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
umount /mnt/boot
# umount /mnt/home
umount /mnt
reboot
```

### 第三部分：驱动安装

[kitty fonts](https://sw.kovidgoyal.net/kitty/conf/#conf-kitty-fonts)

[kitty set nerd fonts](https://www.reddit.com/r/KittyTerminal/comments/r5hssh/kitty_nerd_fonts/)

[Glyph](https://github.com/ryanoasis/nerd-fonts/wiki/Glyph-Sets-and-Code-Points)

In kitty you can config the symbol fonts by code point So need not install addition nerd fonts.

### git fetch submodules

`git clone --recurse-submodules git@github.com:shun-sfoo/xmonad.git`

### Bug Fix

if occurs `OpenSSL SSL_connect: SSL_ERROR_SYSCALL in connection github:443`

may set git config proxy

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

### necessary tools

`sudo pacman -S --needed base-devel`

```bash
sudo pacman --need -S \
kitty jq yt-dlp gtk4 glade p7zip git clash htop btop`# favorite terminal` \
musl `# cros compile` \
lua-language-server typescript-language-server rust-analyzer `# lsp` \
prettier  `# formatter`  \
clang llvm lldb cmake meson ninja gdb valgrind `# c tools`  \
fcitx5 fcitx5-rime fcitx5-gtk rime-double-pinyin `# fcitx` \
exa bat starship zoxide mcfly ripgrep stylua rust-analyzer vivid mdbook `# rust cli`  \
zsh-autosuggestions zsh-syntax-highlighting `# zsh plugins` \
alsa-utils mpv ranger imagemagick cmus `# audio and video`  \
gmtp scrcpy `# android` \
net-tools bind tcpdump inetutils wget lsof`# net tools`
```

HDD

```bash
sudo pacman -S exfatprogs ntfs-3g
```

### wayland

wayfire 为以前的记录

`sudo dmesg |rg i915`

if there has error `i915 [drm] ERROR` meanings intel drm failed.
it may appear in the machine which has tow graphics cards like `intel + nvidia`

to resolve it by edit `/boot/loader/entries/arch.conf`
add the options `i915.modeset=0 nouveau.modeset=0`

[Systemd-boot Configuration](https://wiki.archlinux.org/title/Systemd-boot#Configuration)

[Disabling_modesetting](https://wiki.archlinux.org/title/kernel_mode_setting#Disabling_modesetting)

```bash
sudo pacman -S wayland polkit wlroots glm grim slurp
# Graphics driver
sudo pacman -S mesa vulkan-intel # intel
sudo pacman -S nvidia
paru wayfire
paru wlr-randr-git # output display info
# paru swaylock-effects-git
# pacman -S waybar
sudo pacman -S wofi swaybg wl-clipboard
# pacman -S swayidle
sudo pacman -S mako #notify
```

### Nvidia GBM

To use GBM as a wayland backend

[wayland](https://wiki.archlinux.org/title/wayland#Requirements)

[Wayland_environment](https://wiki.archlinux.org/title/Environment_variables#Wayland_environment)

```bash
vim .pam_enviroment
# or should edit in  ~/.config/environment.d/envvars.conf
# TODO need to check
GBM_BACKEND=nvidia-drm
__GLX_VENDOR_LIBRARY_NAME=nvidia
```

### KMS

in order to enable **Wayland** in nvidia must enable nvidia kms

[KMS](https://wiki.archlinux.org/title/Kernel_mode_setting)

edit `/boot/loader/entries/arch.conf`

add options `nvidia-drm.modeset=1`

### Early KMS start

[Early_KMS_start](https://wiki.archlinux.org/title/kernel_mode_setting#Early_KMS_start)

[nvidia-drm](https://wiki.archlinux.org/title/NVIDIA#DRM_kernel_mode_setting)

```bash
sudo vim /etc/mkinitcpio.conf
MODULES=(i915? nvidia nvidia_modeset nvidia_uvm nvidia_drm)
sudo mkinitcpio -p linux
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

### Docker

consider use podman

```bash
sudo pacman -S docker docker-compose
sudo systemctl enable docker
sudo systemctl start docker
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

### Zathura

`sudo pacman -S zathura-pdf-mupdf`

usage

- [x] `a` page-fit

- [x] `s` width-fit

- [x] `C-r` invert color

### Firefox

```bash
vim .pam_enviroment
MOZ_ENABLE_WAYLAND=1
```

### Electron

```bash
vim  ~/.config/electron-flags.conf
--enable-features=UseOzonePlatform
--ozone-platform=wayland
```

### Renderer

```bash
vim .pam_enviroment
## in the future may use vulkan
WLR_RENDERER=gles2
```

### keychron keyboard

[apple keyboard](https://wiki.archlinux.org/title/Apple_Keyboard)

```bash
sudo vim /etc/modprobe.d/hid_apple.conf

options hid_apple fnmode=2
```

`sudo mkinitcpio -P`

### Vnc

`sudo pacman -S libvncserver remmina`

### lambda

```bash
# prefer to install racket , racket-minimal need a lot of packages
# alternate choice is chicken, but it's  sicp eggs not use in current version
sudo pacman -S fennel fnlfmt racket
```

```bash
# racket included xrepl
raco pkg install sicp
# fmt tools use by raco fmt -i <file>
raco pkg install fmt
```

[xrepl](https://marvinsblog.net/post/2021-03-10-racket-xrepl/)

[vim in racket](https://docs.racket-lang.org/guide/Vim.html)

I have to use `set filetype=racket` in the racket file.

`vim :h lisp lispword`

And could find the lisp config in [vim-racket](https://github.com/wlangstroth/vim-racket) plugin

like `rg lisp` in the plugins dir

In my neovim setting I solved the **colorscheme** and **indent** by simply setting.

### Chrome

`paru google-chrome-stable`

configuration `chrome-flags.conf` enable wayland

### Obsidian

`sudo pacman -S obsidian`

[ArchWiki:Electron Per User](ttps://wiki.archlinux.org/title/Wayland#Per_user)

if wayland is not work, consider link the obsidian `electron<version>-flags.conf`

`ln -s ~/.config/electron-flags.conf ~/.config/electron17-flags.conf`

### gimp

**before build read INSTALL.in and meson.make**

`sudo pacman -S mypaint-brushes1`

mypaint-brushes use the 1 version not default(2)

gimp now use meson build
By default meson install in `/usr/local/bin` and the shared library is `/usr/local/lib`
and the enviroment path don't include. so if build by defalut it occur error.
There has two way to solved:

1. `export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib` not recomment
2. set meson `prefix =/usr` . I choose the way.
   In detail: edit the meson.make and comment the `prefix=$(home)/.local`, and
   we only have the `prefix=/usr`, that's we want. `./meson.make` and cd \_build
   run `meson configure` and we see the directories section prefix is `/usr`.
   Then install it.

```bash
# ERROR: if it error python module giscanner not found
# https://archlinux.org/packages/extra/x86_64/gobject-introspection/files/
# the python file in /usr/bin
# reboot could save the problem
# if not `export PATH="/usr/bin:$PATH"`
```
