---
title: OpenWRT中Wget报Invalid SSL certificate
categories:
  - Linux
tags:
  - openwrt
date: 2018-07-15 17:22:58
---

 在折腾OpenWRT的时候遇到个报错：

```shell
Connection error: Invalid SSL certificate
```

wget的时候无法下载https的资源，猜想是系统中没有相应的证书吧，经过一番搜索，果不其然，于是乎将证书安装上，问题解决。

```shell
opkg install ca-bundle
opkg install ca-certificates
```

由于我插了优盘在路由器上，将软件默认安装到了优盘中，所以还需要将/etc/ssl做个软连接到/etc。

```shell
ln -s /mnt/usb/etc/ssl/ /etc/ssl
```

