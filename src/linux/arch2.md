# arch part2

## Change Shell

```bash
chsh -s $(which zsh)
zsh
```

## User directories

```bash
sudo pacman -S xdg-user-dirs
xdg-user-dirs-update
```

## Nerd Fonts

```bash
sudo pacman -S ttf-nerd-fonts-symbols-2048-em
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji
```

[kitty fonts](https://sw.kovidgoyal.net/kitty/conf/#conf-kitty-fonts)

[kitty set nerd fonts](https://www.reddit.com/r/KittyTerminal/comments/r5hssh/kitty_nerd_fonts/)

[Glyph](https://github.com/ryanoasis/nerd-fonts/wiki/Glyph-Sets-and-Code-Points)

In kitty you can config the symbol fonts by code point So need not install addition nerd fonts.

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

## necessary tools

`sudo pacman -S --needed base-devel`

```bash
sudo pacman --need -S \
kitty jq yt-dlp gtk4 glade`# favorite terminal` \
musl `# cros compile` \
lua-language-server typescript-language-server rust-analyzer `# lsp` \
prettier  `# formatter`  \
clang llvm lldb cmake meson ninja gdb valgrind`# c tools`  \
fcitx5 fcitx5-rime fcitx5-gtk `# fcitx` \
exa bat starship zoxide mcfly ripgrep stylua rust-analyzer vivid mdbook `# rust cli`  \
zsh-autosuggestions zsh-syntax-highlighting `# zsh plugins` \
alsa-utils mpv ranger imagemagick cmus `# audio and video`
```

HDD

```bash
sudo pacman -S exfatprogs ntfs-3g
```

## AUR Helper

```bash
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si
# sudo nvim /etc/pacman.conf
# uncomment Color
# sudo nvim /etc/paru.conf
# uncomment BottomUp
```

## wayland

`sudo dmesg |rg i915`

if there has error `i915 [drm] ERROR` meanings intel drm failed.
it may appear in the machine which has tow graphics cards like `intel + nvidia`

to resolve it by edit `/boot/loader/entries/arch.conf`
add the options `i915.modeset=0 nouveau.modeset=0`

[Systemd-boot Configuration](https://wiki.archlinux.org/title/Systemd-boot#Configuration)

[Disabling_modesetting](https://wiki.archlinux.org/title/kernel_mode_setting#Disabling_modesetting)

```bash
pacman -S wayland polkit wlroots glm
# Graphics driver
sudo pacman -S mesa vulkan-intel # intel
sudo pacman -S nvidia
paru wayfire
paru wlr-randr-git # output display info
# paru swaylock-effects-git
# pacman -S waybar
pacman -S wofi swaybg wl-clipboard
# pacman -S swayidle
pacman -S mako #notify
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

## Software

### python

if not write python scirpt recomment use the system python

pyenv

```bash
curl https://pyenv.run | bash

export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv virtualenv-init -)"
```

### Rust

`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`

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

## keychron keyboard

[apple keyboard](https://wiki.archlinux.org/title/Apple_Keyboard)

```bash
sudo vim /etc/modprobe.d/hid_apple.conf

options hid_apple fnmode=2
```

`sudo mkinitcpio -P`

### Vnc

`sudo pacman -S libvncserver remmina`

### BtTorrent

```bash
sudo pacman -S transmission-cli
transmission-daemon --auth --username arch --password linux \
--port 9091 --allowed "127.0.0.1"
```

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
