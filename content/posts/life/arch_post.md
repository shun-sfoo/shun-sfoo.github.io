+++
title = "arch安装之后"
date = "2022-04-21T09:35:01+08:00"
author = "shun-sfoo"
authorTwitter = "" #do not include @
cover = ""
tags = ["linux", "arch"]
keywords = ["arch", "tutorial"]
description = "arch安装之后"
showFullContent = false
readingTime = true
+++

## Cli

`sudo pacman -S exa bat starship zoxide mcfly zsh-autosuggestions ripgrep stylua rust-analyzer`

## AUR Helper

```bash
sudo pacman -S --needed base-devel
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si
# sudo nvim /etc/pacman.conf
# uncomment Color
# sudo nvim /etc/paru.conf
# uncomment BottomUp
```

## Graphics driver

### Intel

`sudo pacman -S mesa vulkan-intel`

#### Bug Fix

`sudo dmesg |rg i915`

if there has error `i915 [drm] ERROR` meanings intel drm failed.
it may appear in the machine which has tow graphics cards like `intel + nvidia`

to resolve it by edit `/boot/loader/entries/arch.conf`
add the options `i915.modeset=0 nouveau.modeset=0`

[Systemd-boot Configuration](https://wiki.archlinux.org/title/Systemd-boot#Configuration)

[Disabling_modesetting](https://wiki.archlinux.org/title/kernel_mode_setting#Disabling_modesetting)

### Nvidia

`sudo pacman -S nvidia`

#### KMS

in order to enable **Wayland** in nvidia must enable nvidia kms

[KMS](https://wiki.archlinux.org/title/Kernel_mode_setting)

edit `/boot/loader/entries/arch.conf`

add options `nvidia-drm.modeset=1`

#### Early KMS start

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

## Wayland

`sudo pacman -S wayland`

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

### Firefox

```bash
vim .pam_enviroment
MOZ_ENABLE_WAYLAND=1
```

### Renderer

```bash
vim .pam_enviroment
## in the future may use vulkan
WLR_RENDERER=gles2
```

### Apps

```bash
paru wayfire-git
paru wlr-randr-git # output display info
paru swaylock-effects-git
pacman -S waybar
pacman -S swaybg
pacman -S swayidle
pacman -S mako
```

## git

### ssh

ssh generator key

`ssh-keygen -t rsa -C "youremail@example.com" `

enter all the time

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

## developer

### python

pyenv

```bash
curl https://pyenv.run | bash

export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv virtualenv-init -)"
```

### rust

`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`

### node

`sudo pacman -S nodejs npm yarn`

#### yarn use taobao mirror

```bash
yarn global add yrm
yrm ls
yrm use taobao
yrm test taobao
```

### Docker

consider use podman

```bash
sudo gpasswd -a user docker
sudo systemctl enable docker
```

### Format Tools

`yarn global add prettier pyright`

### Vnc

`sudo pacman -S libvncserver remmina`

### BtTorrent

`sudo pacman -S transmission`

### Fcitx5

`sudo pacman -S fcitx5 fcitx5-rime`
