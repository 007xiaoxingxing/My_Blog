---
title: 树莓派搭建iPXE服务
categories:
  - Linux
tags:
  - iPXE netboot
date: 2018-03-13 11:39:55
---

最近我对网络启动产生了浓厚的兴趣，很想亲自搭建一个环境来尝试一下，于是就有了一系列的折腾过程。本篇记录的是使用iPXE来网络启动，自动安装CentOS系统。

<!--more-->

#### 基本技术介绍

PXE:

​    PXE(preboot execute environment，预启动执行环境)是由Intel公司开发的最新技术，工作于Client/Server的网络模式，支持工作站通过网络从远端服务器下载映像，并由此支持通过网络启动操作系统，在启动过程中，终端要求服务器分配IP地址，再用TFTP（trivial file transfer protocol）或MTFTP(multicast trivial file transfer protocol)协议下载一个启动软件包到本机内存中执行，由这个启动软件包完成终端（客户端）基本软件设置，从而引导预先安装在服务器中的终端操作系统。PXE可以引导多种操作系统，如:Windows95/98/2000/windows2003/windows2008/winXP/win7/win8,linux系列系统等。

iPXE:

​    iPXE is the leading open source network boot firmware. It provides a full PXE implementation enhanced with additional features such as:

boot from a web server via HTTP
boot from an iSCSI SAN
boot from a Fibre Channel SAN via FCoE
boot from an AoE SAN
boot from a wireless network
boot from a wide-area network
boot from an Infiniband network
control the boot process with a script
You can use iPXE to replace the existing PXE ROM on your network card, or you can chainload into iPXE to obtain the features of iPXE without the hassle of reflashing.

iPXE is free, open-source software licensed under the GNU GPL (with some portions under GPL-compatible licences), and is included in products from several network card manufacturers and OEMs.

iPXE是一个开源的网络启动固件，它使用了诸如http,ftp等协议来传输，相较于普通的PXE固件使用TFTP来进行文件传输速度更快，而且还支持烧录到网卡芯片，CD-ROM，U盘上，有了它，只要有网络的地方就可以启动机器进行系统维护或者系统安装了。

iPXE官网地址：http://ipxe.org   该项目开源，可以自己克隆代码进行编译。

#### 软硬件环境准备

硬件：树莓派3，提供系统支持，用作启动服务器。当然，如果有其它Linux系统的电脑也是可以的。

软件：nginx，作为web服务器，提供系统镜像，本地软件源。dnsmasq，dnsmasq自身带有tftp服务器，支持pxe，支持dhcpPROXY模式，有了它就不用再去安装单独的dhcp服务器和tftp服务器了。

安装软件：

```shell
pi@raspberrypi:~ $ sudo apt-get instal nginx
pi@raspberrypi:~ $ sudo apt-get instal dnsmasq
```

#### 准备所需的系统镜像和pxe文件

下载CentOS7系统镜像，并复制相应文件到web目录：

```shell
pi@raspberrypi:~ $ wget https://mirrors.aliyun.com/centos/7.4.1708/isos/x86_64/CentOS-7-x86_64-Minimal-1708.iso
pi@raspberrypi:~ $ mount CentOS-7-x86_64-Minimal-1708.iso /mnt
pi@raspberrypi:~ $ sudo mkdir -p /var/www/html/centos/7/x86_64/
pi@raspberrypi:~ $ sudo cp -r /mnt/* /var/www/html/centos/7/x86_64/
```

下载iPXE相关的启动文件到web目录

```shell
pi@raspberrypi:/var/www/html $ sudo wget http://boot.ipxe.org/undionly.kpxe
```

现在我的web根目录的文件内容：

```shell
pi@raspberrypi:/var/www/html $ ls -la
总用量 811112
drwxr-xr-x 3 root root      4096 3月  12 21:53 .
drwxr-xr-x 3 root root      4096 10月 21 09:31 ..
-rw-r--r-- 1 root root        43 3月  12 21:52 boot.txt
drwxr-xr-x 3 pi   pi        4096 3月  12 15:23 centos
-rw-r--r-- 1 pi   pi   830472192 9月   5  2017 CentOS-7-x86_64-Minimal-1708.iso
-rw-r--r-- 1 root root      1191 3月  12 20:13 ks.cfg
-rw-r--r-- 1 pi   pi         705 3月  12 19:56 menu.conf
-rw-r--r-- 1 root root     67369 3月  12 21:00 undionly.kpxe
```



编写启动菜单menu.conf，内容如下：

```config
#!ipxe
    set menu-timeout 20000
    set munu-default winpe
    set protocal:string http
    isset ${ip} || dhcp
:start
    menu iPXE Boot Menu --${ip}
    item  winpe         Boot WindowsPE
    item centos7        Boot CentOS
    item shell          Boot iPXE Shell
    choose --timeout ${menu-timeout} --default ${menu-default} selected
    goto ${selected}
:centos7
    #set base http://mirrors.aliyun.com/centos/7.4.1708/os/x86_64
    set base http://192.168.2.189/centos/7/x86_64
    kernel ${base}/images/pxeboot/vmlinuz initrd=initrd.img ks=http://192.168.2.189/ks.cfg repo=${base}
    initrd ${base}/images/pxeboot/initrd.img
    boot
:winpe
    kernel ${protocal}://${next-server}/boot/wimboot
```

编辑dnsmasq配置文件，使其支持pxe，让它能够根据网卡不同的请求发送不同的boot file。由于我的电脑和树莓派都接在一台openwrt路由器下边，路由器已经开启了DHCP功能，为了不对路由器本身的DHCP功能造成影响，我的pxe服务器上的dnsmasq配置为proxy模式，只对pxe请求作出响应，普通的DHCP请求分配IP地址不作响应。话说为了这个配置文件我真是搜索了好多资料，也经过了无数次的尝试，总是在iPXE固件的第二次DHCP请求时无法发送菜单内容过去，卡住了很久，最终在谷歌的帮助下，搜索到一篇文章，用他的配置终于成功了。

```shell
no-daemon  #不作为守护进程，保持在前台，便于调试，正常使用时可去掉
dhcp-range=192.168.2.0,proxy #自己所在的需要代理的网段，我的是192.168.2.0
dhcp-boot=tag:!ipxe,undionly.kpxe,192.168.2.189 #如果不是ipxe固件发起的，则发送undionly.kpxe
dhcp-match=set:ipxe,175 # gPXE/iPXE sends a 175 option.
dhcp-boot=tag:!ipxe,undionly.kpxe 
dhcp-boot=http://192.168.2.189/menu.conf,192.168.2.189,192.168.2.189 #如果是ipxe发起的请求，则发送菜单链接，后边两个ip为pxe服务器ip
pxe-service=tag:!ipxe,x86PC,"splash",undionly.kpxe #对所有的pxe请求做出响应，发送undionly.kcpxe
enable-tftp #开启tftp
tftp-root=/var/www/html/  #设置tftp的根目录
log-queries  #打印所有dhcp请求，用于调试
conf-dir=/etc/dnsmasq.d  #包含的其他配置
```

#### 服务测试

开启dnsmasq服务：

```shell
pi@raspberrypi:~/pxe_start $ sudo dnsmasq -C proxy_dhcp 
dnsmasq: started, version 2.76 cachesize 150
dnsmasq: compile time options: IPv6 GNU-getopt DBus i18n IDN DHCP DHCPv6 no-Lua TFTP conntrack ipset auth DNSSEC loop-detect inotify
dnsmasq-dhcp: DHCP, proxy on subnet 192.168.2.0
dnsmasq-tftp: TFTP root is /var/www/html/ 
dnsmasq: reading /etc/resolv.conf
dnsmasq: using nameserver 192.168.2.1#53
dnsmasq: read /etc/hosts - 5 addresses

```

开启nginx服务：

```shell
pi@raspberrypi:~/pxe_start $ sudo nginx
```

建立虚拟机，设置网络模式为桥接。

vmare的启动效果：

![image](/blogimg/vmware_ipxe.gif)

vbox启动效果图：

![imange](/blogimg/vbox_ipxe.gif)

#### CentOS无人职守自动安装

在一台有图形界面的机器上安装kickstart，然后按照提示生成ks.cfg文件。我的ks.cfg是从一台已经安装好的centos上提取的，内容如下：

```config
#platform=86, AMD64, or Intel EM64T

#version=DEVEL
# System authorization information
auth --useshadow  --passalgo=sha512
# Install OS instead of upgrade
install
# Use network installation
url --url="http://192.168.2.189/centos/7/x86_64/"
# Use text mode install
text
# Firewall configuration
firewall --disabled
firstboot --disable
ignoredisk --only-use=sda
# Keyboard layouts
# old format: keyboard us
# new format:
keyboard --vckeymap=us --xlayouts=''
# System language
lang en_US.UTF-8

# Network information
network  --bootproto=dhcp --device=eth0 --activate
network  --hostname=localhost.localdomain
# Reboot after installation
reboot
# Root password
rootpw --iscrypted $1$.iNc/OT7$5cYmijQ4G5pBjxC8sgAIJ/
# SELinux configuration
selinux --enforcing
# System services
services --enabled="chronyd"
# Do not configure the X Window System
skipx
# System timezone
timezone Asia/Shanghai
# System bootloader configuration
bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda
autopart --type=lvm
# Partition clearing information
clearpart --all --initlabel --drives=sda

%packages
@core
chrony
kexec-tools

%end

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end

```

要实现自动安装，在启动菜单中给个内核参数即可

```config
kernel ${base}/images/pxeboot/vmlinuz initrd=initrd.img ks=http://192.168.2.189/ks.cfg 
```

自此，pxe启动自动安装centos搭建完成，接下来再尝试启动其它系统和winpe。

参考链接：

https://dev.to/arachan/ipxe-chainloading-to-use-dnsmasq-and-proxydhcp-4he