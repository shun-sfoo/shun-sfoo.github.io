# 使用 openwrt 做旁路由

[参考](https://xmanyou.com/vmware-esxi-install-openwrt/)

## 虚拟机方案

以 mac 平台为例

下载 openwrt 镜像，使用已经编译好的固件

`https://github.com/coolsnowwolf/lede/releases`

将 img 固件转换成 vmdk 格式

```bash
brew install qemu
qumu-img convert -f raw -O vmdk xxxx.img xxxx.vmdk
```

### vmware fusion 操作

新建， 创建自定虚拟机，linux 版本选择 5.0 内核以上 64 位，传统 bois, 使用现有虚拟磁盘，其他默认。

**注意** 虚拟机，网络适配器，选择**桥接模式**

启动后，出现一些加载信息，然后点击回车（之前的某些镜像没有点击回车还以为下载出现问题）

### openwrt network 设置

```bash
vim /etc/config/network
option ipaddr '192.168.1.1'
```

这里的 ipaddr 设置取决于本地路由，
本地路由器如果是 `192.168.1.1`
这里就改成 `192.168.1.!1` 最后以为修改成想要设置的地址比如 `192.168.1.115`

### 代理设置

打开 openwrt 管理页面

浏览器打开`192.168.1.115`

用户名默认：root

密码默认： password

#### 接口修改

`网络->接口->点击修改`

ipv4 地址填设置的 115

ipv4 网关填 本地路由器地址

ipv4 广播设置为 255 (这个是否需要设定？)

自定 dns 服务器地址填本地路由地址

保存，在网络诊断中点击 ping 测试是否网络联通。

#### 其他端使用代理

代理设置完成后
位于统一网关下的机器将路由器地址设置为 openwrt 的 ipaddr
本地静态 ip 设置成不同于 openwrt 的 ip
dns 设置为 `114.114.114.114`

以 linux 的 netifrc 设置为例

```bash
vim /etc/conf.d/net
config_eth0="192.168.1.7 netmask 255.255.255.0"
routes_eth0="default via 192.168.1.1"
dns_servers_eth0="192.168.1.1 114.114.114.114"
```
