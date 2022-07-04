---
title: 'Wayland'
date: 2022-05-26T19:36:51+08:00
draft: false
---

## Wayland

`sudo pacman -S wayland`

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

### Apps

wayfire dependency

```bash
sudo pacman -S polkit
```

```bash
# if wayfire is out of update use wayfire-git instead of
paru wayfire
paru wlr-randr-git # output display info
# paru swaylock-effects-git
# pacman -S waybar
pacman -S wofi
pacman -S swaybg
# pacman -S swayidle
pacman -S wl-clipboard
pacman -S mako
```

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
