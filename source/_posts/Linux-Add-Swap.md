---
title: Linux学习笔记之添加SWAP分区
categories:
  - Linux
tags:
  - Linux
  - SWAP
date: 2017-05-23 22:53:19
---

这台博客所在的服务器只有740M内存，云服务器默认也是没有给SWAP分区的。刚刚我想更新一下服务器的软件包，但是报了这么一个错：

```bash

Error downloading packages:
  php70w-xml-7.0.19-1.w7.x86_64: [Errno 5] [Errno 12] Cannot allocate memory
  libmount-2.23.2-33.el7_3.2.x86_64: [Errno 5] [Errno 12] Cannot allocate memory
  grubby-8.28-21.el7_3.x86_64: [Errno 5] [Errno 12] Cannot allocate memory
  httpd-tools-2.4.6-45.el7.centos.4.x86_64: [Errno 5] [Errno 12] Cannot allocate memory
  libtool-ltdl-2.4.2-22.el7_3.x86_64: [Errno 5] [Errno 12] Cannot allocate memory
  php70w-intl-7.0.19-1.w7.x86_64: [Errno 5] [Errno 12] Cannot allocate memory
  1:NetworkManager-team-1.4.0-19.el7_3.x86_64: [Errno 5] [Errno 12] Cannot allocate memory
  util-linux-2.23.2-33.el7_3.2.x86_64: [Errno 5] [Errno 12] Cannot allocate memory
  php70w-7.0.19-1.w7.x86_64: [Errno 5] [Errno 12] Cannot allocate memory
  httpd-2.4.6-45.el7.centos.4.x86_64: [Errno 5] [Errno 12] Cannot allocate memory
  kernel-tools-3.10.0-514.16.1.el7.x86_64: [Errno 5] [Errno 12] Cannot allocate memory
  1:NetworkManager-libnm-1.4.0-19.el7_3.x86_64: [Errno 5] [Errno 12] Cannot allocate memory
  php70w-pdo-7.0.19-1.w7.x86_64: [Errno 5] [Errno 12] Cannot allocate memory
  php70w-pecl-apcu-5.1.8-1.w7.x86_64: [Errno 5] [Errno 12] Cannot allocate memory
  ntp-4.2.6p5-25.el7.centos.2.x86_64: [Errno 5] [Errno 12] Cannot allocate memory
  7:device-mapper-libs-1.02.135-1.el7_3.4.x86_64: [Errno 5] [Errno 12] Cannot allocate memory

```

<!-- more -->

看来是内存不够了，看一下内存占用情况呢：

```bash
[root@vultr ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:            740         454         146          37         139         130
Swap:             0           0           0
```

唉，空间确实不太充裕了，那就给它加个swap分区，让一些内存交换到硬盘上吧。

#### dd分配出一块空间

```bash
[root@vultr ~]# dd if=/dev/zero of=/opt/swap bs=1024 count=1048576
1048576+0 records in
1048576+0 records out
1073741824 bytes (1.1 GB) copied, 2.27196 s, 473 MB/s
```

bs=块的大小，单位bit,我这里给的1KB

count=块的数量，我这里给了1024*1024个，总计就是1G

#### 将分配的空间格式化为SWAP格式

```bash
[root@vultr ~]# mkswap /opt/swap 
Setting up swapspace version 1, size = 1048572 KiB
no label, UUID=f42645fc-5a4b-4cfd-99c5-caf6fa6070f7
```

可以看到已经为它分配了UUID。

#### 将格式化好的空间挂载上

```bash
[root@vultr ~]# swapon /opt/swap 
swapon: /opt/swap: insecure permissions 0644, 0600 suggested.
```

咦，它提示说权限不太安全，那就chmod成0600吧。

#### 开机自动挂载SWAP分区

开机自动挂载的话，需要在/etc/fstab中添加挂载选项。

```bash
 echo "/opt/swap swap swap defaults  0 0" >> /etc/fstab
```

好了，这样给服务器成功地加上了个SWAP分区了。再检查一下

```bash
[root@vultr ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:            740         454          62          37         222         117
Swap:          1023           0        1023 
```

Swap分区已经不为0了，确认已经分配完成。