# 显卡直通
此教程绝大多数来源于是archlinux wiki, 只是记录其要点作为速通教程.

使用 systemd-network 设置网络，因为虚拟机需要用到桥接方式，那么直接创建桥接网络
[archwiki 的相关章节](https://wiki.archlinux.org/title/Systemd-networkd#Network_bridge_with_DHCP)

具体步骤：

1.  通过netdev 单元文件创建桥接网络

```bash
/etc/systemd/network/mybridge.netdev
[NetDev]
Name=br0
Kind=bridge
```

2. 绑定网卡到桥接网络

```bash
/etc/systemd/network/10_bind.network
[Match]
Name=en*

[Network]
Bridge=br0
```

3. 桥接网络

```bash
/etc/systemd/network/mybridge.network
[Match]
Name=br0

# 此处是动态分配
[Network]
DHCP=ipv4
# 建议通过分配静态IP的方式
[Network]
DNS=192.168.1.254 # 8.8.8.8 114.114.114.114 这里的设置待判断
Address=192.168.1.123/24
Gateway=192.168.1.254
```

archwiki 中提到 `systemd-resolved is required if DNS entries are specified in .network files.`

因此要同时启用 systemd-networkd 和 systemd-resolved
