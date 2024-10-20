# archlinux 安装简明教程

该教程分为两个部分：

1. 第一部分使用 archinstall 脚本快速安装，profile 选择 minimal
2. 第二部分在第一部分的基础上进一步配置常用工具。

## 第一部分：使用 archinstall 快速安装

注意虚拟机安装archlinux时，bios选项选择 uefi, 使用默认的bios时没有网络连接，原因待查.但不重要

- mirror region: 选择 china 同vim一样`/` 可以用来搜索
- Disk configuration: 选择 use a best-effort default partition layout ,格式选择btrfs 根目录分配200G
- user account: as superuser
- profile: 选择 `minimal`
- Audio: 选择 `no`
- Additional packages: `git neovim zsh`
- NetWork configuration : 选择手动设置成静态ip, 这种方式生成是 systemd-networkd 的配置(当然 copy 也是)
- kernels: 选择 `linux`

这一部分会自动根据当前cpu下载驱动，并把对应的 ucode 放在 boot 中。

## 第二部分： 系统配置

设置时区

```bash
vim /etc/mkinitcpio.conf
add r8169
```
```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

通过 archinstall 安装了基本的环境，但是还需要做一些配置。

第二部分的目的是可以保证做到可以写博客。

amd: `sudo pacman -S mesa vulkan-radeon`

audio 根据hyprland文档中指出选择pipewire 和  WirePlumber  (archlinux wiki 中指出  PipeWire Media Session已经被弃用 任何时候都不要选择)

`sudo pacman -S pipewire wireplumber`

另外把pipewire作为其他组件的后端需要下载 

`pipewire-alsa`  提供所有使用 alsa api的应用

`pipewire-audio` 代替 pulse-audio 和 pulse-bluetooth

`pipwire-jack`  jack支持

<!--使用eww作为status bar 需要下载-->
<!---->
<!--`gtk-layer-shell ` 和 `gtk3`-->

hyprland 组件

xdg-desktop-portal-hyprland

qt5-wayland 

qt6-wayland

hyprland

总结下载

`sudo pacman -S --need mesa vulkan-radeon  pipewire wireplumber pipewire-alsa  pipewire-audio pipwire-jack xdg-desktop-portal-hyprland  qt5-wayland qt6-wayland`

为了达到这个目的:

1. 首先需要chrome, 而chrome 需要 aur helper 我选择是 paru。
2. 另一方面是有写的工具，这需要 neovim 的配置， 而同步配置需要配置git环境。
3. 安装配置输入法

### 一些环境配置配置项

首先把 base 切换成 zsh

```bash
chsh -s $(which zsh)
zsh
```

生成 ssh 公钥

```bash
ssh-keygen -t rsa -C "ganymede0915@gmail.com"
```

关闭密码输入

```bash
sudo nvim /etc/sudoers.d/00_neo
neo ALL=(ALL) NOPASSWD: ALL
```

### 字体

```bash
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji ttf-nerd-fonts-symbols
```

字体详细配置是通过 `.font.conf`

### 必备软件

终端工具

```bash
sudo pacman -S --need install starship zoxide mcfly zsh-autosuggestions zsh-syntax-highlighting
```

nvim 的 mason 可以下载很多开发工具, 因此只需要把下载一些 mason 用到的工具

```bash
sudo apt install unzip nodejs npm cmake ninja fd wget ripgrep gdb bc valgrind
```

其他工具

```bash
sudo pacman -S btop htop neofetch openssh wl-clipboard ranger libmtp android-file-transfer
```

sudo systemctl enable sshd

### 输入法

fctix5 集合

```bash
sudo pacman -S  fcitx5 fcitx5-rime fcitx5-gtk fcitx5-configtool rime-double-pinyin
```

说明 fcitx5-gtk 是在用于 gtk 环境中的显示输入框。

开机运行通过 hyprland 中配置 exec-once fcitx5

### 音视频

```bash
sudo pacman -S mpd mpc mpv wf-recorder
```

1. 使用 `mpd` 先配置音乐，pipewire， mpd生成 database , 如果有端口问题可以kill mpd.socket services
2. `mpc update` `mpc listall |mpc add`
3. `mpc play` `mpc next` `mpc pause`

如果 mpc playlist 中有未识别的在字符，可以通过 `kid3` 调整 `id tag`

### Rust and Python

rust 与 python 特殊之处在于，一般不用系统自带的，需要另外配置环境

rust:

```bash
`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
```

python:

```bash
curl https://pyenv.run | bash

export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv virtualenv-init -)"
```


### VSCODE In Wayland
[link1](https://bbs.archlinux.org/viewtopic.php?id=286537)

EDIT: setting "window.titleBarStyle": "custom" in ~/.config/Code/User/settings.json does seem to mostly resolve.

and `.config/code-flags.conf`
 
```config
--enable-features=WaylandWindowDecorations
--ozone-platform-hint=auto
```

### Aur Helpr

```bash
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si
# sudo nvim /etc/pacman.conf
# uncomment Color
# sudo nvim /etc/paru.conf
# uncomment BottomUp
```

### 键盘配置

keychron keyboard

[apple keyboard](https://wiki.archlinux.org/title/Apple_Keyboard)

```bash
sudo vim /etc/modprobe.d/hid_apple.conf

options hid_apple fnmode=2
```

这一步目的是让我的 k8 键盘默认是 fn 模式
