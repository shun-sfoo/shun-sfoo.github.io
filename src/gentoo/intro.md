# 前言

![larry](../static/gentoo/Larry-hi.png)

前置说明:

1. 不考虑老旧的 mbr 格式 一律选择 gpt + uefi 的组合
2. 文件的主要格式是 xfs
3. 使用 wm 而不是 de (de 有崩溃风险，且编译太耗时)
4. systemd 的手越来越长越管越多, 选择使用 openrc。
5. 网络方案选择 netifrc 和 iwd, networkmanage 和 systemd 都是 red hat 弄出来的，弃之。
6. gentoo 放弃了 consolekit ，选择 elogind
7. lto pgo 优化,可以提升程序运行速度并且减小程序体积
8. bootloader 中暂时先选择 grub2, 后续改成直接用 efibootmgr
9. jack2 低延迟音频组件 cmus 播放器 mpv 播放器
10. nftables 作为防火墙
11. TODO: 使用 doas 作为权限管理 取代 sudo
12. TODO: X11 桌面的选择 bspwm ... , sway 的配置, iwd

关于显卡驱动:

- sway 不支持 nvidia 驱动, 所以 n 卡选择 `xserver + dwm`
- 集成显卡和 amd 显卡选择 `wayland + sway`
- intel 显卡驱动 xf86-video-intel 年久失修 ,现在推荐的是的 `x11-base/xorg-drivers` 和 `media-lib/mesa`
- 笔记本可选择双显卡驱动， 台式机只用单显卡。

上述组合拳下来 选择 profile 方案就是默认，也即服务器 openrc 版本,

由此也可初步选择一些全局 USE:

```bash
# 括号表示根据情况选装
# vdpau vaapi 安装视频播放器需要
NEO_VIDEO="(nvidia) vulkan vdpau vaapi"

# pluseaudio与alsa基本一致， oss太过古老
NEO_AUDIO="jack libsamplerate alsa -pulseaudio -oss"

NEO_COMPILE="lto pgo ccache (sudo) (dosa) minizip openmp"

# nftables防火墙可以取代了iptables
NEO_NET="-iptables nftables netifrc (-)wifi -networkmanage -dhcpcd"

# elogind 取代了 consolekit
NEO_DESKTOP="elogind -bindist -consolekit -gnome-shell -gnome -gnome-keyring -kde -systemd (-)X (-)wayland (-)bluetooth cjk"

USE="${NEO_VIDEO} ${NEO_AUDIO} ${NOE_COMPLE} ${NEO_NET} ${NEO_DESKTOP}"
```

使用 LiveUSB （推荐 Fedora）安装, 好处是可以直接从磁盘分区开始
而且可以在终端复制粘贴此处的 code。

教程跟着官网走就行，以下步骤是为了安装时能无脑复制。(最好还是对照官网)

# 文章列表

[安装](./gentoo.md)

[portage](./portage.md)

[内核配置](./core.md)

[fstab 设置](./fstab.md)

[驱动](./driver.md)

[笔记](./note.md)
