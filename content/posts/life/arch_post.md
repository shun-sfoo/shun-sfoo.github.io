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

## Change Shell

```bash
chsh -s $(which zsh)
zsh
```

## Cli

Rust

```bash
sudo pacman -S exa bat starship zoxide mcfly ripgrep stylua rust-analyzer vivid
```

zsh plugins

```bash
sudo pacman -S zsh-autosuggestions zsh-syntax-highlighting
```

HDD

```bash
sudo pacman -S exfatprogs ntfs-3g
```

audio and video

```bash
sudo pacamn -S alsa-utils mpv ranger imagemagick
```

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

## terminal

kitty

- [x] kitty `c and python opengl termial`
- [x] vivid `LSCOLOR`
- [x] imagemagick `operator picture`

## keychron keyboard

[apple keyboard](https://wiki.archlinux.org/title/Apple_Keyboard)

```bash
sudo vim /etc/modprobe.d/hid_apple.conf

options hid_apple fnmode=2
```

`sudo mkinitcpio -P`

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

### Rust

`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`

### Node

`sudo pacman -S nodejs npm yarn`

#### Yarn Choose Mirror

```bash
yarn global add yrm
yrm ls
yrm use taobao
yrm test taobao
```

#### Yarn tools

`yarn global add prettier pyright`

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

### Vnc

`sudo pacman -S libvncserver remmina`

### BtTorrent

```bash
sudo pacman -S transmission-cli
transmission-daemon --auth --username arch --password linux --port 9091 --allowed "127.0.0.1"
```

### Zathura

`sudo pacman -S zathura-pdf-mupdf`

usage

- [x] `a` page-fit

- [x] `s` width-fit

- [x] `C-r` invert color

### lambda

```bash
sudo pacman -S fennel fnlfmt racket-minimal
```

```bash
# `xrepl` will install a hugo lot of dependencies and now I don't need it
# if need  xrepl  install racket better
raco pkg install sicp
# fmt tools use by raco fmt -i <file>
raco pkg install fmt
```

[xrepl](https://marvinsblog.net/post/2021-03-10-racket-xrepl/)

[vim in racket](https://docs.racket-lang.org/guide/Vim.html)

I have to use `set filetype=racket` in the racket file.

### lsp

```bash
sudo pacman -S lua-language-server
```

### llvm

`sudo pacman -S clang llvm lldb`

### Fcitx5

`sudo pacman -S fcitx5 fcitx5-rime fcitx5-gtk`

### Chrome

`paru google-chrome-stable`

configuration `chrome-flags.conf` enable wayland

### Obsidian

`sudo pacman -S obsidian`

[ArchWiki:Electron Per User](ttps://wiki.archlinux.org/title/Wayland#Per_user)

if wayland is not work, consider link the obsidian `electron<version>-flags.conf`

`ln -s ~/.config/electron-flags.conf ~/.config/electron17-flags.conf`
