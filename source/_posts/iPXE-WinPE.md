---
title: iPXE启动Windows PE
categories:
  - Linux
tags:
  - iPXE winpe
date: 2018-03-14 10:16:54
---

接上回，上次我们已经实现了iPXE服务的搭建和自动安装centos 7。那这次来尝试启动一下Windows PE。iPXE官网上的启动boot.wim的方式可以作为参考，但是默认的纯粹的PE只有个cmd窗口，显然不堪大用，最好是使用一个支持网络的PE,那么从网络启动之后，还可以上上网，多么舒服。

<!--more-->

于是我找到了无忧论坛上一位大佬制作的win10 pe，自带网络功能和网络启动功能，我仔细一看，他用的还是TinyPXE server，正好这个网络启动软件也支持iPXE，于是乎就可以直接拿来用了嘛。

http://bbs.wuyou.net/forum.php?mod=viewthread&tid=378234  ，于是乎我花了一晚上从百度网盘将该PE的iso下载回来，然后把该iso内的全部文件都弄到树莓派的/var/www/winpe目录下。

```shell
pi@raspberrypi:/var/www/html/winpe $ ls -la
总用量 1484
drwxr-xr-x 5 root root   4096 3月  14 09:55 .
drwxr-xr-x 5 root root   4096 3月  14 09:51 ..
dr-xr-xr-x 4 root root   4096 3月  14 09:58 boot
-r--r--r-- 1 root root 397858 10月  7 17:34 bootmgr
-r--r--r-- 1 root root 648088 10月  7 17:09 bootmgr.exe
-r--r--r-- 1 root root    490 2月  18  2016 config.ini
dr-xr-xr-x 4 root root   4096 3月  14 09:32 efi
-r--r--r-- 1 root root 332583 10月  7 17:28 ipxe.pxe
-r--r--r-- 1 root root    555 9月  13  2015 pxeautorun.txt
-r--r--r-- 1 root root  25358 9月  29 21:37 pxeboot.n12
-r--r--r-- 1 root root  67227 10月  7 17:28 undionly.kpxe
dr-xr-xr-x 2 root root   4096 3月  14 09:32 wifidriver
-r--r--r-- 1 root root   1903 3月  22  2017 网启说明.txt

```

接下来照猫画虎，更改一下咱们自己的menu菜单

```config
#!ipxe
    set menu-timeout 20000
    set munu-default winpe
    set protocal:string http
    isset ${ip} || dhcp
:start
    menu iPXE Boot Menu --${ip}
    item  winpe         Boot WindowsPE
    item centos7        Boot CentOS 7.4
    item ubuntu         Boot Ubuntu 17.10
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
    set base http://192.168.2.189/winpe
    kernel ${base}/boot/wimboot gui || goto retry
    initrd ${base}/pxeautorun.txt ${next-server}.cmd || goto nextg
    :nextg
    initrd ${base}/boot/boot.sdi   boot.sdi  || goto retry
    iseq ${platform} pcbios || goto winefi
    initrd ${base}/bootmgr.exe bootmgr.exe || goto retry
    initrd ${base}/boot/bxe bxd || goto retry
    initrd ${base}/boot/PE64.WIM  boot.WIM  || goto retry
    boot || goto retry
    goto start
:winefi
    set base http://192.168.2.189/winpe
    initrd ${base}/EFI/Boot/bootx64.efi || goto retry
    initrd ${base}/boot/bcd  || goto retry
    initrd ${base}/boot/PE64.WIM || goto retry
    boot || goto retry
    goto start
:ubuntu
    set base http://192.168.2.189/ubuntu/
    kernel ${base}/install/netboot/ubuntu-installer/amd64/linux ks=http://192.168.2.189/ks_ubuntu.cfg
    initrd ${base}/install/netboot/ubuntu-installer/amd64/initrd.gz
    boot
```

嗯，就这样了，再创建一台虚拟机来启动一下试试看效果。

![image](/blogimg/ipxe_winpe.gif)

由于我的网络环境是百兆，加载500多M的wim需要一段时间，如果是千兆环境就更快了。启动成功，看起来效果不错。难道说我还要去学习一下如果制作PE么，这又要入坑了。

最后还是要感谢制作该PE镜像的大佬，你的PE还是蛮好使的。