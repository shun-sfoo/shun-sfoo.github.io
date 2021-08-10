# make.conf

## default

```bash
COMMON_FLAGS="-march=native -O2 -pipe"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
FCFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"
MAKEOPTS="-j4"

# CPU_FLAGS_X86="aes avx avx2 f16c fma3 mmx mmxext pclmul popcnt sse sse2 sse3 sse4_1 sse4_2 ssse3"
# cpu参数用cpuid2cpuflags命令看，这里先不用管，暂时注释掉，后面再来配置

# 注意核心数
EMERGE_DEFAULT_OPTS="--with-bdeps=y --ask --verbose=y --load-average --keep-going --deep"
PORTDIR="/var/db/repos/gentoo"
DISTDIR="/var/cache/distfiles"
PKGDIR="/var/cache/binpkgs"


PORTAGE_TMPDIR="/tmp"
# 大内存(8G、16G) 设置 小于 4G内存不用设置。
LC_MESSAGES=C

ACCEPT_LICENSE="*"
ACCEPT_KEYWORDS="amd64"
# "amd64"是使用稳定版的较旧的软件，"~amd64"是使用不稳定版的更新的软件


USE="-bindist -consolekit -gnome-shell -gnome -gnome-keyring -kde -systemd elogind lto pgo netifrc (-)X (-)wayland (-)wifi (-)bluetooth -networkmanager -dhcpcd (nvidia) vulkan ccache (sudo) (dosa) minizip openmp"

L10N="en-US zh-CN en zh"
LINGUAS="en-US zh-CN en zh"
AUTO_CLEAN="yes"

ALSA_CARDS="hda-intel"
# intel HD声卡
# INPUT_DEVICES="libinput synaptics"
# 笔记本电脑的触控板

VIDEO_CARDS="nvidia"
# VIDEO_CARDS="intel i965 iris"
# VIDEO_CARDS="intel i965 iris nvidia"

LLVM_TARGETS="X86"
GENTOO_MIRRORS="https://mirrors.ustc.edu.cn/gentoo/"

GRUB_PLATFORMS="efi-64"
# UEFI 64位系统引导必须项

# QEMU_SOFTMMU_TARGETS="alpha aarch64 arm i386 mips mips64 mips64el mipsel ppc ppc64 s390x sh4 sh4eb sparc sparc64 x86_64"
# QEMU_USER_TARGETS="alpha aarch64 arm armeb i386 mips mipsel ppc ppc64 ppc64abi32 s390x sh4 sh4eb sparc sparc32plus sparc64 x86_64"
# 虚拟机

#FEATURES="ccache"
#CCACHE_DIR="/var/cache/ccache"
#此处先注释掉,配置完ccache后再去掉注释
```

## compelete

`/etc/portage/make.conf`
定义整个系统在构建的过程中按照怎样的规则来进行编译，具有一定的全局属性。
比方说编译器的选择，系统语言的设定，使用什么特性来编译，系统架构，显卡类型等等

一个例子：

```bash

# GCC
# Please consult /usr/share/portage/config/make.conf.example for a more
COMMON_FLAGS="-march=native -O2 -pipe"
# 无论何种intel、amd的CPU，且无论何种新老CPU架构，均建议-march=native，CPU指令集自动识别全面
# 程序员的用户注意了，“-fomit-frame-pointer"这一项会导致你编译出来的程序无法debug；
# 不做程序开发或debug的普通用户可以放心开启。

CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
FCFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"
MAKEOPTS="-j4"
# 推荐值为CPU中核心/逻辑处理器的数量，可用lscpu命令查看，结果为“CPU(s):”后面的数字

# CPU_FLAGS_X86="aes avx avx2 f16c fma3 mmx mmxext pclmul popcnt sse sse2 sse3 sse4_1 sse4_2 ssse3"
# cpu参数用cpuid2cpuflags命令看，这里先不用管，暂时注释掉，后面再来配置

EMERGE_DEFAULT_OPTS="--with-bdeps=y --ask --verbose=y --load-average --keep-going --deep"
# 这个设置的目的是在遇到编译错误的时候不要停止，而是继续编译下去

# NOTE: This stage was built with the bindist Use flag enabled
PORTDIR="/var/db/repos/gentoo"
DISTDIR="/var/cache/distfiles"
PKGDIR="/var/cache/binpkgs"
PORTAGE_TMPDIR="/tmp"
# 大内存(8G、16G) 设置 小于 4G内存不用设置。

# This sets the language of build output to English.
# Please keep this setting intact when reporting bugs.
LC_MESSAGES=C

MINUS="-bindist -mdev -systemd -consolekit -bluetooth -gtk -netifrc -oss -gpm -iptables"
DESKTOP="-gnome-shell -gnome -gnome-keyring -wayland X cjk"
AUDIO="-pulseaudio alsa jack"
VIDEO="vulkan nvidia"
COMPILE="fortran lto pgo openmp minizip"
ELSE="sudo ccache aria2"
# USE="plugins ${MINUS} ${DESKTOP} ${AUDIO}  ${VIDEO} ${COMPILE} ${ELSE}"
# 建议不要在 make.conf 中定义 USE 去 /etc/portage/package.use/ 中定义。

ACCEPT_LICENSE="*"
ACCEPT_KEYWORDS="amd64"
# "amd64"是使用稳定版的较旧的软件，"~amd64"是使用不稳定版的更新的软件

L10N="en-US zh-CN en zh"
LINGUAS="en-US zh-CN en zh"
AUTO_CLEAN="yes"

GRUB_PLATFORMS="efi-64"
# UEFI 64位系统引导必须项

VIDEO_CARDS="nvidia"
# VIDEO_CARDS="intel i965 iris"
# VIDEO_CARDS="intel i965 iris nvidia"

ALSA_CARDS="hda-intel"
# intel HD声卡
# INPUT_DEVICES="libinput synaptics"
# 笔记本电脑的触控板
MICROCODE_SIGNATURES="-S"
# 如果想把CPU的microcode直接编译进内核，则需要设置为“-S”；否则注释掉

# QEMU_SOFTMMU_TARGETS="alpha aarch64 arm i386 mips mips64 mips64el mipsel ppc ppc64 s390x sh4 sh4eb sparc sparc64 x86_64"
# QEMU_USER_TARGETS="alpha aarch64 arm armeb i386 mips mipsel ppc ppc64 ppc64abi32 s390x sh4 sh4eb sparc sparc32plus sparc64 x86_64"
# 虚拟机

LLVM_TARGETS="X86"

GENTOO_MIRRORS="https://mirrors.ustc.edu.cn/gentoo/"

#FEATURES="ccache"
#CCACHE_DIR="/var/cache/ccache"
#此处先注释掉,配置完ccache后再去掉注释
```
