# 记录一些设置

## 调整 tmpfs 大小

[tmpfs](https://wiki.gentoo.org/wiki/Portage_TMPDIR_on_tmpfs#Resizing_tmpfs)

编译一些大型软件(chromium)的时候会出现临时空间不足的情况，通过调整大小来解决这个问题。

1. 直接调整 fstab 中设置的大小

`tmpfs /tmp tmpfs rw,nosuid,noatime,nodev,relatime,mode=1777,size=10G 0 0`

内存 tmpfs(/tmp 目录)的大小，2G 内存设为 1G、4G 内存设为 2G、8G 内存可设为 4-6G、16G 内存可设为 10-13G

调整完成之后要再次挂载指定的挂载点

`mount /tmp`

2. 重新设置大小

`mount -o remount,size=N /var/tmp/portage`

## nginx

精简编译安装 nginx 作为你的 http 和 webdav 文件服务器，方便与其他电脑和手机通过网络传输文件，

```bash
sudo USE="http -http-cache -http2 ipv6 pcre -ssl aio -debug -libatomic -libressl -luajit -pcre-jit -rtmp (-selinux) threads -vim-syntax" NGINX_MODULES_HTTP="-access auth_basic autoindex -browser charset -empty_gif -fastcgi -geo -grpc -gzip -limit_conn -limit_req -map -memcached -mirror -proxy -referer rewrite -scgi -split_clients -ssi -upstream_hash -upstream_ip_hash -upstream_keepalive -upstream_least_conn -upstream_zone -userid -uwsgi -addition -auth_ldap -auth_pam -auth_request -brotli -cache_purge dav dav_ext -degradation -echo -fancyindex -flv -geoip -geoip2 -gunzip -gzip_static -headers_more -image_filter -javascript -lua -memc -metrics -mogilefs -mp4 -naxsi -perl -push_stream -random_index -realip -secure_link -security -slice -slowfs_cache -spdy -sticky -stub_status -sub -upload_progress -upstream_check -vhost_traffic_status xslt" NGINX_MODULES_MAIL="-imap -pop3 -smtp" NGINX_MODULES_STREAM="-access -geo -geoip -geoip2 -javascript -limit_conn -map -realip -return -split_clients -ssl_preread -upstream_hash -upstream_least_conn -upstream_zone" emerge www-servers/nginx

sudo groupadd netshare
sudo useradd netshare -g netshare -s /sbin/nologin -d /dev/null   #创建一个只有最低权限的安全的网络分享用户(中介/中间用户)。此用户没有自己的家目录，无法登陆桌面，无法用bash执行命令，以防止黑客利用这些权限破解我的gentoo系统。网络安全从小事做起，恩
sudo mkdir /home/netshare    #创建用于网络分享的目录，此目录里的文件将在http浏览器或webdav客户端里见到
sudo chown netshare /home/netshare -R
sudo chgrp netshare /home/netshare -R
sudo chmod 700 /home/netshare -R
sudo mkdir /var/tmp/nginx
sudo mkdir /var/tmp/nginx/client
sudo chown netshare /var/tmp/nginx -R
sudo chgrp netshare /var/tmp/nginx -R
```

```bash
sudo vim /etc/nginx/nginx.conf：   #完整配置如下
user netshare netshare;
worker_processes auto;
worker_rlimit_nofile 65535;
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
#pid        logs/nginx.pid;
events {
    worker_connections  65535;
    use epoll;
    multi_accept on;
    accept_mutex off;
}
http {
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
    #access_log  logs/access.log  main;
    aio threads;
    charset utf-8;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    server_tokens off;
    types_hash_max_size 2048;
    keepalive_timeout  65;
    server_names_hash_max_size 512;
    server_names_hash_bucket_size 1024;
    client_max_body_size 0;
    dav_ext_lock_zone zone=foo:10m;
    server {
       listen 8888 default_server;
       listen [::]:8888 default_server;
       server_name _;
       location / {
         autoindex on;
         auth_basic "What's your passwd?";
         auth_basic_user_file /etc/nginx/passwd;
         root /home/netshare/;
         dav_methods PUT DELETE MKCOL COPY MOVE;
         dav_ext_methods PROPFIND OPTIONS LOCK UNLOCK;
         dav_ext_lock zone=foo;
         create_full_put_path on;
         # enable creating directories without trailing slash
         set $x $uri$request_method;
         if ($x ~ [^/]MKCOL$) {
             rewrite ^(.*)$ $1/;
         }
         client_body_temp_path /var/tmp/nginx/client;
         dav_access user:rw group:r all:r;
       }
    }
}
```

在 ufw-GUI 中打开 8888 端口(TCP)，设置为 allow in

```bash
sudo vim /etc/nginx/passwd：
netshare:ZAKqshAPTy6TM    #用户名netshare，密码kof97

sudo rc-service nginx start
sudo rc-update add nginx default
```

重启后通过浏览器或 webdav 客户端访问 http://你的 ip:8888

```bash
sudo vim /etc/portage/package.mask:
www-servers/nginx
```

避免 nginx 在后续的滚动更新中被重新编译

option: 调整 linux 内核 TCP 网络参数以提高 nginx 上传下载速度

```bash
sudo vim /etc/sysctl.d/99-sysctl.conf：     #重启后生效
fs.inotify.max_user_watches = 600000
dev.i915.perf_stream_paranoid = 0
vm.swappiness = 1
net.ipv6.conf.all.accept_ra = 2
fs.file-max = 6553560
net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
net.core.rmem_default = 65536
net.core.wmem_default = 65536
net.core.optmem_max = 10000000
net.core.netdev_max_backlog = 8096
net.core.somaxconn = 8096
net.ipv4.ip_default_ttl = 128
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.route.gc_timeout = 100
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl = 15
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_ecn = 1
net.ipv4.tcp_sack = 1
net.ipv4.tcp_fack = 1
net.ipv4.tcp_low_latency = 0
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_frto = 2
net.ipv4.tcp_frto_response = 0
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.ipv4.tcp_mtu_probing = 1
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_max_syn_backlog = 30000
net.ipv4.tcp_max_orphans = 262114
net.ipv4.netfilter.ip_conntrack_max = 204800
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
# for high-latency network Google BBR    #个人PC不建议开启BBR，对网速不会有提升，还会降低wifi吞吐量；还是等BBR2正式版出了再开启BBR2吧，BBR2对wifi就没有影响了
#net.ipv4.tcp_congestion_control = bbr
net.core.default_qdisc = fq_codel
net.ipv4.tcp_congestion_control = cubic
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
net.ipv6.ip_default_ttl = 128
```

```bash
sudo vim /etc/security/limits.conf：
* soft nofile 65536
* hard nofile 65536
* soft noproc 65536
* hard noproc 65536
```

## TODO: dolphin
