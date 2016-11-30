---
title:  搭建OpenWRT编译开发环境 
date: 2016-11-28 19:25:18
categories: Linux
tags:
- Linux
- openwrt
- enviroment
---
本次搭建编译环境所需的准备工作：
1. 操作系统：物理机或者虚拟机安装----Ubuntu 16.10
2. 预备知识：了解一点点Linux终端的基本操作

Let's go!  
首先还是更新一下系统：

``` bash

  star-chen@starchen-op:~$ sudo apt-get update && apt-get dist-upgrade 

```
接下来参考OpenWRT官方网站给出的搭建编译环境所需要进行的软件包的安装步骤进行安装，官网步骤网址：[OpenWRT官方步骤](https://wiki.openwrt.org/doc/howto/buildroot.exigence)
由于我使用的是Ubuntu16.10 64bit系统，所以需要执行的安装命令为：
``` bash

star-chen@starchen-op:~$ sudo apt-get install git-core mercurial build-essential subversion libncurses5-dev zlib1g-dev gawk gcc-multilib flex git-core gettext libssl-dev

```
软件包安装完毕之后，测试一下git命令是否可用：
``` bash
star-chen@starchen-op:~$ git
usage: git [--version] [--help] [-C <path>] [-c name=value]
           [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
           [-p | --paginate | --no-pager] [--no-replace-objects] [--bare]
           [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
           <command> [<args>]
           此处省略一万字.......

```

经测试，git命令可用，接下来创建一个新的目录用来存放OpenWRT源码：
``` bash
star-chen@starchen-op:~$ mkdir OP1505

```
进入OP1505目录并从git仓库克隆OpenWRT 15.05(chaos_calmer)分支：
``` bash
star-chen@starchen-op:~$ cd OP1505/
star-chen@starchen-op:~$ git clone -b chaos_calmer git://github.com/openwrt/openwrt.git

```
更新并安装软件包：
``` bash
 cd openwrt
./scripts/feeds update -a
./scripts/feeds install -a

```
根据自己的需要设置编译条件，生成makefile（另一篇专门讲讲这部分）：
``` bash
make menuconfig

```
OK,make menuconfig通过，接下来咱们编译一个x86的固件来爽爽：

![image](/blogimg/op-makemenu.png)

就以默认参数吧,我给虚拟机分了四个核，4GB内存，这里用两个线程来跑吧。
``` bash

make j=2 V=99

```
编译过程中可能需要下载很多文件，所以挂个代理什么的很有必要，不然很多依赖无法下载，造成编译失败，耐心等待吧。编译完成之后，固件将会在openwrt/bin目录下躺好了。