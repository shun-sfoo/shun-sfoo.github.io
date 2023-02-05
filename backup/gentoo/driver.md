# 驱动

本文参考知乎专栏

[医学生折腾 Gentoo Linux 记](https://www.zhihu.com/column/c_1271625347856310272)

## 视频

### intel 显卡

**注意** `x11-drivers/xf86-video-intel` 驱动已经年久失修不要再安装。

```bash
sudo emerge x11-base/xorg-drivers
sudo emerge media-libs/mesa
```

### nvidia

```bash
sudo emerge x11-drivers/nvidia-drivers
```

**注意**：如果安装后提示 warning（红色"\*"号提示）当前内核配置的“CONFIG_I2C_NVIDIA_GPU=y”这一项不符合要求，
与 nvidia-drivers 冲突，需要按照提示将这一项内核配置禁用，并重新编译安装内核

**注意** ：以后每次重新编译安装内核 kernel 后，均须要运行一遍“emerge @module-rebuild”，重新编译安装 nvidia 驱动模块加载到内核之中，否则 nvidia 驱动无法加载！！！

```bash
lsmod | grep nvidia

sudo rmmod nvidia
sudo  modprobe nvidia

lsmod | grep nvidia


sudo vim /etc/modules-load.d/nvidia.conf:
nvidia

sudo vim /etc/modprobe.d/nvidia-drm.conf：
options nvidia-drm modeset=1


sudo rc-update add modules boot

sudo reboot   #重启系统
```

### xorg-server

```bash
sudo emerge x11-base/xorg-server
```

TODO: xorg-server config

## 音频

系统使用 jack 低延迟音频组件播放无损音乐（PC-HiFi），播放音乐时由播放器独占声卡

系统的音频驱动只需要 alsa 和 jack 就行了，二者组合在一起用。

```bash
sudo emerge libsamplerate #采样率转换必须依赖的组件
sudo emerge -C jack-audio-connection-kit #删除老的 jack1
sudo emerge media-sound/jack2 #换用新的 jack2

sudo emerge media-sound/cadence #安装强大的 jack2 图形管理界面 GUI
```

### cadence 使用

1. ` System-->JACK Brideges-->ALSA Audio-->Bridge Type:(None)` 不需要将 alsa 的音频重定向到 jack 中
2. ` System-->Configure-->Engine-->勾选“Realtime”`，并将“Realtime priority”的值设置为 99（最高优先级），其他保持默认即可
3. `System-->Configure-->Driver-->ALSA-->"Device/Interface"` 设置为你自己要用来听音乐的声卡，去掉“Duplex Mode”的勾选，勾选上“Monitor”和“Hardware Metering”，“Sample Rate”设置为你电脑的声卡支持的最高采样率，一般普通的板载声卡最高支持 48000,专业独立声卡能够支持更高，设置过高音乐不能正常播放。“Buffer Size”设置为你的电脑 CPU 能承受的最小值，值越小越好，播放音乐的延迟也越低，一般现代的 CPU 最低可以设置为 256；设置过低会导致播放音乐时出现爆音、卡顿（Xruns）。“Periods/Buffer”建议设置为最小值 2,越小的值带来越低的延迟，如果你的 CPU 承受不住这种负荷了（表现为播放时出现卡顿、爆音），再把值调大。其他保持默认即可。
4. `System-->点击“Start”`即可启动 jack2,此时由于 jack 将声卡独占了，所以系统里除了正在运行的支持 jack 的软件之外，其他的所有软件均不能播放声音，你的浏览器也不能正常播放视频。注意这不是 bug，想要取消声卡独占点击“stop”就可以了。

**注意** 以后但凡使用任何支持 jack 的软件之前，均要先打开 `cadence-->System-->点击“Start”`。

一是启动 jack2；二是应用你所作出的 jack 设置

### 音乐播放器

cmus，一个运行于命令行终端下的轻量级音乐播放器，操作方便，响应迅速，系统资源占用极少，而且音质不赖，支持 jack 音频组件

```bash
sudo emerge cmus

# 配置
vim ~/.config/cmus/rc:
set output_plugin=jack #使用 jack 作为音频输出
set dsp.jack.resampling_quality=2 #使用最高质量的音频输出
```

### 视频播放器

安装视频播放器（这里 make.conf 的 USE 标记需加入“vdpau vaapi”）

著名的轻量级高性能播放器 mpv

```bash
sudo emerge mpv
# 配置 ffmpeg
sudo vim /etc/portage/package.use/ffmpeg:
media-video/ffmpeg openssl network #使 ffmpeg 支持网络播放
sudo emerge ffmpeg #重新编译 ffmpeg 以支持网络播放
```

#### mpv 设置

```bash
vim ~/.config/mpv/mpv.conf：
hwdec=vdpau-copy #使用 nvidia 硬解，copy 方法能够更降低系统资源占用
vo=gpu #使用 GPU 硬件解码
gpu-api=vulkan #使用 vulkan
profile=gpu-hq
scale=ewa_lanczossharp
cscale=ewa_lanczossharp
#interpolation

#使用 nvidia-prime 双显卡设置后，若这一项开启会导致 mpv 播放时视频和音频不同步，视频速度过快的问题，目前这是个 bug
#video-sync=display-resample

tscale=oversample
vd-lavc-dr=yes
opengl-pbo=yes
ao=jack #使用 jack 独占声卡音频输出，使用后 mpv 本身无法调节音量。
audio-exclusive=yes #独占声卡
```

## 防火墙

使用现代的 nftables，淘汰老旧的 iptables

```bash
# 安装nftables，删除iptables
sudo emerge net-firewall/nftables
sudo emerge -C net-firewall/iptables

# 开始添加nftables规则
sudo nft flush ruleset
sudo nft add table inet filter
sudo nft add chain inet filter input { type filter hook input priority 0 \; policy drop \; }
sudo nft add chain inet filter forward { type filter hook forward priority 0 \; policy drop \; }
sudo nft add chain inet filter output { type filter hook output priority 0 \; policy accept \; }
sudo nft add rule inet filter input iif lo accept
sudo nft add rule inet filter input ct state related,established accept
sudo nft add rule inet filter input ct state invalid drop
sudo nft add rule inet filter input tcp dport 8080 accept      #开放本机8080/tcp端口
sudo nft add rule inet filter input udp dport 8080 accept      #开放本机8080/udp端口


# 保存以上创建的规则，并设置为开机自动加载
sudo /etc/init.d/nftables save
sudo /etc/init.d/nftables start
sudo /etc/init.d/nftables reload
sudo /etc/init.d/nftables list
sudo rc-update add nftables default
```

## 电源

[gentoo wiki Power_management](https://wiki.gentoo.org/wiki/Power_management)

[gentoo wiki acpi](https://wiki.gentoo.org/wiki/ACPI)

```bash
emerge --ask sys-power/acpid # Power Management
emerge sys-power/thermald # intel support acpi
rc-update add thermald default
```
