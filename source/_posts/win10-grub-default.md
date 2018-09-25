---
title: Win10与Linux双系统时，使用GRUB作为默认引导
categories:
  - Linux
tags:
  - grub
date: 2018-03-26 20:20:53
---

我的电脑是Win10与Linux双系统，最近由于升级了一次Windows导致Windows的bootmgr取代了Linux的grub启动菜单成了默认启动项，导致无法直接启动到Linux系统，虽然可以在UEFI shell通过手动执行grubx64.efi进入到grub菜单，实在是麻烦。于是乎在网上搜索一下解决方案，以下这种方式我试验成功，特此记录，以免忘记。

<!--more-->

1.启动到Win10,以管理员身份运行命令提示符

2.将EFI分区挂载到一个目前未被使用的盘符

```shell
C:\WINDOWS\system32>mountvol z: /s
```

3.查看一下EFI分区内的文件内容

```shell
Z:\>dir
 驱动器 Z 中的卷没有标签。
 卷的序列号是 50E3-51B8

 Z:\ 的目录

2016/04/14  17:25    <DIR>          EFI
               0 个文件              0 字节
               1 个目录    283,672,576 可用字节

Z:\>dir EFI
 驱动器 Z 中的卷没有标签。
 卷的序列号是 50E3-51B8

 Z:\EFI 的目录

2015/10/30  10:40    <DIR>          .
2015/10/29  09:48    <DIR>          ..
2015/10/29  09:48    <DIR>          Microsoft
2015/10/29  09:51    <DIR>          Boot
2015/11/19  12:47    <DIR>          kali
               0 个文件              0 字节
               5 个目录    283,672,576 可用字节

Z:\>dir EFI\kali
 驱动器 Z 中的卷没有标签。
 卷的序列号是 50E3-51B8

 Z:\EFI\kali 的目录

2015/11/19  12:47    <DIR>          .
2015/11/19  12:47    <DIR>          ..
2017/03/06  03:39           121,856 grubx64.efi
               1 个文件        121,856 字节
               2 个目录    283,672,576 可用字节
```

4.执行命令将grubx64.efi设置为默认启动项

```shell
Z:\>bcdedit /set {bootmgr} path \EFI\kali\grubx64.efi
```

5.重启电脑查看效果

重启之后正常的话就能够看到grub启动菜单了。