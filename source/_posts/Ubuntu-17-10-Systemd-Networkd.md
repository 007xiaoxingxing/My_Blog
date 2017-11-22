---
title: Ubuntu-17.10-使用Systemd-Networkd接管网络配置
categories:
  - Linux
tags:
  - linux
date: 2017-11-22 15:03:28
---

最近在使用Ubuntu17.10的时候，被设置ip折磨的挺惨。我使用的台式机是戴尔的optiplex 9020，装的是win10和Ubuntu17.10双系统。最开始是从win10重启到Ubuntu后，Ubuntu无法上网。经过一番搜索设置也无果，在一次偶然将win10关机，进到bios设置了一下网卡后，开机到Ubuntu竟然能够正常上网了。
<!--more-->
最终解决该问题的办法是：将win10关机，再开机启动到Ubuntu。
第二个问题是因为公司使用的是静态ip上网的方式，按照往常的经验，编辑/etc/network/interface配置文件，设置静态ip即可。但是ip设置的问题是解决了，但是DNS是被systemd-resolved接管的，每次手动更改了/etc/resolv.conf后重启，该文件总是被覆盖。索性将系统的网络全部由systemd-networkd接管好了。
### 步骤
1.在/etc/systemd/network目录新建一个网卡配置文件，我的叫static-eno1.network，注意，一定得是.network结尾的文件名。内容如下：
```shell
[Match]
    Name=eno1
[Network]
    Address=10.70.27.35/19
    Gateway=10.70.31.254
    DNS=223.5.5.5
    DNS=223.6.6.6

```
这里由两个地方需要注意，Match段，需要指定配置的网卡Name，Name需要注意首字母大写。Address是网卡的ip地址，由ip和掩码长度组成。需要设置多个DNS的话，请写多行。
2.设置systemd-networkd开机启动
```shell
$sudo systemctl enable systemd-networkd
```
稳妥起见的话，还可以将/etc/resolv.conf删除，然后建立/run/systemd/resolv/resolv.conf的软连接
```shell
$sudo ln -s /run/systemd/resolv/resolv.conf /etc/resolv.conf
```
然后重启机器即可。
