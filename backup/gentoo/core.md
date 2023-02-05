# 内核的配置

## 将微码编进内核

```bash
emerge --ask sys-firmware/intel-microcode sys-apps/iucode_tool
#下载速度慢 需要科学上网，微码的部分可先跳过
iucode_tool -S
iucode_tool -S -l /lib/firmware/intel-ucode/*
# 识别处理器签名并查找相应microcode文件名,其为“06-9e-0d”这样的编号格式，选择最末尾的那个（最新的）
# 建议把找到的结果用手机拍照下来，然后设置编译内核（make menuconfig）的时候
```

```bash
make menuconfig 中
Device Driver:
    Generic Driver Options:
         Firmware loader:
             Build name firmware blobs into the kernel binary

输入“intel-ucode/06-9e-0d”这样的编号格式，将微码文件直接编译进内核。
```

确保 emerge intel-microcode 之前就得在 已经在 make.conf 中设置好了
`MICROCODE_SIGNATURES="-S"`

配置内核时开启

```bash
CONFIG_MICROCODE=y
CONFIG_MICROCODE_INTEL=y
```

## 永久禁用 nouveau

```bash
vim  /etc/modprobe.d/blacklist.conf：
blacklist nouveau
blacklist lbm-nouveau
options nouveau modeset=0
```

即便在编译内核前就已经设置内核禁用 Nouveau 驱动了，

但是内核安装时还是会默认把 nouveau 驱动作为内核模块自动加载。

启用了 nouveau 驱动模块的内核会出现各式各样的莫名其妙的数不清的问题，所以为了避免以后出现这些问题，

从现在就开始永久禁用 nouveau 模块！这是很多教程包括 gentoo wiki 上都不曾提到过的大问题，

也是让很多人遭坑的关键地方。

禁用 Nouveau 驱动

`CONFIG_I2C_NVIDIA_GPU` 禁用

#### jack 低延迟实时音频组件

支持 jack 低延迟实时音频组件（Jack-Audio-Connection-Kit），需要修改`vim .config`，手动设置

```bash
# jack
CONFIG_CGROUPS=y
CONFIG_CGROUP_SCHED=y
CONFIG_RT_GROUP_SCHED=y
```
