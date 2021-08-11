# fstab

先了解下 fstab，就像一张表，在 Linux 开机的告诉 mount 应该把哪个分区以什么文件系统，
以什么方式挂载到系统对应位置。这里提供一份使用 btrfs 的 fstab 书写方式。

其次，就是挂载时需要注意的挂载选项，无论你是使用 ext4 文件系统，
还是使用 btrfs 文件系统，使用合适的挂载选项有助于最大性能的发挥你的文件系统的性能，
一方面实现快速读取和写入，另一方面最小化文件丢失

## fstab

分别记录 btrfs 和 xpf 的 fstab 方案

```bash
blkid >> /etc/fstab    #将输出结果追加到fstab配置文件末尾，然后根据追加的内容进行下述修改
```

### btrfs

如果使用 btrfs 文件系统的话，非常推荐如下的挂载选项

`defaults,noatime,space_cache,space_cache=v2,autodefrag,discard=async,ssd,compress=zstd:1`

使用 discard=async 与 fstrim 是不冲突的；另外透明压缩不需要启动等级 3，第一等级就足够了，
如果有需求使用 btrfs 下的 swap ，那不能启用透明压缩功能；并且启用了自动碎片整理功能。这样对文件系统是比较好的。

整个 fstab 的书写就是这样的 UUID 可以使用 blkid 命令查看
`nona /etc/fstab`

```conf
UUID=boot-uuid  /boot       vfat  defaults 0 0
UUID=root-uuid  /           btrfs subvol=@,defaults,noatime,space_cache,space_cache=v2,autodefrag,discard=async,ssd,compress=zstd:1 0 1
UUID=home-uuid  /home       btrfs defaults,noatime,space_cache,space_cache=v2,autodefrag,discard=async,ssd,compress=zstd:1 0 2
UUID=opt-uuid   /opt        btrfs defaults,noatime,space_cache,space_cache=v2,autodefrag,discard=async,ssd,compress=zstd:1,commit=120 0 2
tmpfs           /tmp        tmpfs size=8G,notaime 0 0
tmpfs           /var/tmp    tmpfs size=8G,notaime 0 0
```

最后加了 tmpfs 的内容，建议所有不论你安装什么桌面环境，不论用于什么生产环境，都加上

### xpf

```bash
nano -w /etc/fstab：      #参考配置，建议使用uuid的形式设置（不建议使用/dev/sdX？的形式）(是uuid，而不是partuuid)
UUID=......      /boot/efi      vfat      noauto,defaults,noatime,umask=0077                               0 2
UUID=......      /boot          ext4      defaults,noatime,discard                                         0 2
UUID=......      /              xfs       defaults,noatime                                                 0 1
UUID=......      /home          xfs       noatime,discard                                                  0 2
UUID=......      none           swap      sw,noatime,discard                                               0 0
tmpfs            /tmp           tmpfs     rw,nosuid,noatime,nodev,relatime,mode=1777,size=10G               0 0

# 内存tmpfs(/tmp目录)的大小，2G内存设为1G、4G内存设为2G、8G内存可设为4-6G、16G内存可设为10-13G
# 根分区/不建议设置discard参数，你得记得每个星期定期执行一遍"sudo fstrim -v /"命令来优化根分区/
# discard和fstrim都是专门针对SSD固态硬盘的优化，并且你的SSD必须确保支持TRIM；否则在不支持TRIM的SSD上盲目使用discard和fstrim优化很可能会有数据丢失的风险，2017年以后的SSD基本上都支持TRIM了。
```

我的台式机的 ssd 在 2017 年前生产，考虑去掉 discard 和 fstrim
