# Debug Kernel By Qemu And Debug

## requires

`sudo pacman --needed -S qemu-full bc gdb`

## Download Source Code

kernel

`git clone --depth 1 --branch v4.19 https://github.com/torvalds/linux.git`

[busybox](https://busybox.net/downloads/)

## Configuration and Build

### kernel

```bash
make menuconfig
```

kernel need to turn on

```bash
Kernel hacking -> Kernel debugging
Kernel hacking -> KGDB:kernel debugger
Kernel hacking -> Compile time checks and compiler options -> Provide GDB scripts for kernel debugging
```

need to turn off

```bash
Kernel hacking -> Compile time checks and compiler options -> Reduce debugging information
```

and then build

`make -jx`

generate `vmlinux` and `arch/x86_64/boot/bzImage`

## Make rootfs

### build an initrd-base rootfs

initrd is an in-memory root file system that loads the system before the hard disk is driven.

```c
#include <stdio>
int main() {
    printf("hello linux");
    printf("hello world");
    printf("hello linux");
    printf("hello world");
    fflush(stdout);
    while(1);
    return 0;
}
```

`gcc --static -o fakeinit fakeinit.c`

use the cpio program to package.

```bash
echo fakeinit | cpio -o --format=newc > initrd_rootfs.img
```

## Download and build busybox

make menuconfig

```bash
Settings -> Build Options -> Build static binary (no shared libs)
```

Creating rootfs with busybox

dd if=/dev/zero of=./busybox_rootfs.img bs=1M count=10
mkfs.ext3 ./busybox_rootfs.img

```bash
mkdir rootfs_mount
sudo mount -t ext3 -o loop ./busybox_rootfs.img ./rootfs_mount
```

make install CONFIG_PREFIX=/path/to/rootfs_mount/

```bash
mkdir /path/to/rootfs_mount/proc
mkdir /path/to/rootfs_mount/dev
mkdir /path/to/rootfs_mount/etc
cp busybox-source-code/examples/bootfloppy/* /path/to/rootfs_mount/etc/
sudo umount /path/to/rootfs_mount
```

```bash
qemu-system-x86_64 \
  -kernel ./linux/arch/x86/boot/bzImage \  # 指定编译好的内核镜像
  -initrd ./rootfs/initrd_rootfs.img \  # 指定rootfs
  -serial stdio \ #指定使用stdio作为输入输出
  -append "root=/dev/ram rdinit=/fakeinit console=ttyS0 nokaslr" \ # 内核参数，指定使用initrd作为rootfs，禁止地址空间布局随机化
  -s -S # 指定Qemu在启动时暂停并启动gdb server，等待gdb的连入（端口默认为1234）
```

```bash
qemu-system-x86_64 \
  -kernel ./linux/arch/x86/boot/bzImage \
  -hda ./rootfs/busybox_rootfs.img \ # 指定磁盘镜像
  -serial stdio \
  -append "root=/dev/sda console=ttyS0 nokaslr" \ # 内核参数，指定root磁盘，禁止地址空间布局随机化
  -s -S
```

```bash
gdb ./linux/vmlinux # 指定调试文件为包含调试信息的内核文件
target remote:1234
```
