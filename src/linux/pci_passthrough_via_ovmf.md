# 显卡直通

此教程绝大多数来源于是archlinux wiki, 只是记录其要点作为速通教程.

## bios 设置

1. 开启虚拟化， amd 中是设置 svm mode
2. 开启 IOMMU, 这个选项在 settings 中，另外为了保险起见关闭了安全启动
3. 打开 SR-IOV 支持这个是用来虚拟网卡的

通过如下命令确认开启了 IOMMU

`dmesg | grep -i -e DMAR -e IOMMU`

通过如下脚本查看 IOMMU 分组情况

```bash
#!/bin/bash
shopt -s nullglob
for g in $(find /sys/kernel/iommu_groups/* -maxdepth 0 -type d | sort -V); do
    echo "IOMMU Group ${g##*/}:"
    for d in $g/devices/*; do
        echo -e "\t$(lspci -nns ${d##*/})"
    done;
done;
```

注意，一个组是直通的最小单位，我显卡组里面的两个设备都要直通出来。

```txt
IOMMU Group 12:
	01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GM206 [GeForce GTX 960] [10de:1401] (rev a1)
	01:00.1 Audio device [0403]: NVIDIA Corporation GM206 High Definition Audio Controller [10de:0fba] (rev a1)
```

有些主板上把cpu 和 gpu放在同一分组了，这是因为在该主板上，gpu插在了cpu直通pcie上，会识别成一个组，
遇到这种情况把gpu换到里cpu较远的插槽。我的主板没有这种情况。

## 显卡分离

把设备 id 绑定到 vfio-pci

1. 把id添加到内核参数

我用的是systemd-boot, 因此在 `/boot/loader/entries/2023-11-12_13-33-25_linux-zen.conf`
的 options 后面添加

`vfio-pci.ids=10de:1401,10de:0fba`

2. 在 modprobe.d 中添加

```bash
nvim /etc/modprobe.d/vfio.conf
options vfio-pci ids=10de:1401,10de:0fba
```

3. 提前加载 vfio-pci

wiki中指出需要在显卡驱动绑定到显卡之前把vfio-pci模块注册上。
有两个方式 修改 modprobe.d 和 initramsfs

modprobe.d

```
nvim /etc/modprobe.d/vfio.conf
options vfio-pci ids=10de:1401,10de:0fba
softdep drm pre: vfio-pci
```

这种方式在我电脑上不生效，wiki中指出这步如果出现问题，要验证initramsfs方式是否可以。
总结就是直接使用initramsfs方式

initramsfs

把 `vfio_pci vfio vfio_iommu_type1`

放到 mkinitcpio 中

```bash
nvim /etc/mkinitcpio.conf
MODULES=(... vfio_pci vfio vfio_iommu_type1 ...)
```

还要确认下 HOOKS 包含在 mkinitcpio.conf 中

```bash
/etc/mkinitcpio.conf
HOOKS=(... modconf ...)
```

重新生成 initramsfs

`sudo mkinitcpio -p linux-zen` 因为我用的是 zen 内核

## 软件方案

`sudo pacman -S qemu-desktop libvirt edk2-ovmf virt-manager dnsmasq dmidecode`

加入 libvirt 组

`sudo usermod -aG libvirt neo`

`id $(whoami)` 查看是否加入到组内

开启 libvirt服务

```bash
sudo systemctl enable libvirtd
sudo systemctl start libvirtd
```

激活 libvirt 默认网络

```bash
sudo -s
virsh net-autostart default
virsh net-start default
```

## 安装虚拟机

我选择的镜像是 win10 最新版

1. bios 选择 uefi 主板
2. cpu socket 1 ,core 6, thread 2.
3. 硬盘选择 virio 方式提升性能
4. 下载[virto-win.iso](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/)驱动镜像作为CD-ROM设备
5. 安装时会找不到硬盘因为是virio方式，加载第四步的驱动就可以接着往下
6. 检查设备管理器的驱动，用驱动盘去更新没有识别的驱动

装完系统直接是可以联网的，如果出现断网标志，网络未连接不一定是断网了，可以通过浏览器试试网络。

## Looking-Glass

[教程参考](https://looking-glass.io/docs/B6/install/)

设置 shmem 容量

```bash
sudo -s
EDITOR=nvim virsh edit win10
# 在 devices 节点中加入 128M 是为了4k
<shmem name='looking-glass'>
  <model type='ivshmem-plain'/>
  <size unit='M'>128</size>
</shmem>

```

开启 tmpfiles

```bash
# Type Path               Mode UID  GID Age Argument

f /dev/shm/looking-glass 0660 user kvm -
```

其中user 设置为用户名

```bash
sudo systemd-tmpfiles --create /etc/tmpfiles.d/10-looking-glass.conf
```

建立 tmpfs

开启虚拟机后在虚拟中下载最新版的 `VC_redist.x64.exe` 和对应的 looking-glass window host版本，比如都是最新的b6版.
然后在虚拟机把**内置的显卡驱动关闭**，只用独显。

键鼠剪切板增强

确认有 `<grapics type=spice>` 设备

- Find your `<video>` device, and set `<model type='vga'/>`

- If you can’t find it, make sure you have a `<graphics>` device, save and edit again.

- Remove the `<input type='tablet'/>` device, if you have one.

- Create an `<input type='mouse' bus='virtio'/>` device, if you don’t already have one.

- Create an `<input type='keyboard' bus='virtio'/>` device to improve keyboard usage.

重启虚拟机后把未识别的驱动用驱动盘更新一下

虚拟机与主系统共剪切板可以通过安装 [spice-guest-tools](https://www.spice-space.org/download.html#windows-binaries)

## 问题

1. 使用 looking-glass-client 后发现屏幕分辨率达不到4k，首先把独显也接到显示器上，通过屏幕切换到独显的视频口输出，设置为4k后
   切回主系统就可以设置成4k了，但目前只有30hz，不知道是显卡的原因还是设置的问题。

2. 鼠标偏移，可以先多接一个鼠标，后续删除。

3. looking-glass-client 使用出现服务未启动，确认下是不是把内置显示驱动禁用了。

4. 可能网络设置的原因，lol创建游戏后进不去。steam下载古墓丽影游戏都是没问题的，但由于4k只有30帧，换成1080p才能正常玩。

## 优化

hupage

cpu 设置

## 参考

[looking-glass](https://looking-glass.io/docs/B6/install/)

[别人的文章](https://ivonblog.com/posts/qemu-kvm-vfio-gaming/#6-%E8%A8%AD%E5%AE%9Alooking-glass%E4%BD%8E%E5%BB%B6%E9%81%B2%E5%AD%98%E5%8F%96windows%E6%A1%8C%E9%9D%A2)

[spice](https://www.spice-space.org/download.html#windows-binaries)
