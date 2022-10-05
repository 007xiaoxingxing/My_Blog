---
title: KVM命令行安装windows7
categories:
  - Linux
tags:
  - kvm
date: 2018-03-08 22:29:29
---

书接上回，上次我们在Ubuntu上安装好了KVM软件，并且使用图形化的virt-manager来创建了一台windows虚拟机，这次我们使用KVM的命令行工具来安装一台windows虚拟机。

<!--more-->

#### 步骤1：准备操作系统镜像

这里我使用的是老毛子精简版windows7镜像，整个镜像大小1.6G左右。

![image](/blogimg/win7_iso.png)

#### 步骤2：在宿主机上创建响应目录

接下来需要在宿主机上创建用来存放虚拟机安装镜像和虚拟机镜像的目录。

```shell
root@ubuntu-kvm:/# tree kvm_images/
kvm_images/
├── guest_os
│   └── mini_win7.img
└── iso
    └── 23403_GRMCULXFRER_ZH-CN_MICRO_v2.iso

2 directories, 2 files

```

这是我的目录结构。接下来将系统安装镜像传到宿主机中。

#### 步骤3：命令行安装

```shell
root@ubuntu-kvm:/kvm_images/iso# virt-install --name mini-win7 -hvm --ram 1024 --vcpus 1 --disk path=/kvm_images/guest_os/mini_win7.img,size=10 --network network:default --accelerate --vnc --vncport=5901 --vnclisten=0.0.0.0 --cdrom /kvm_images/iso/23403_GRMCULXFRER_ZH-CN_MICRO_v2.iso -d
```

部分参数解释：

--name:创建的虚拟机名称

-hvm：完全虚拟机

--ram:虚拟机内存大小，单位M,这里我给了1G

--vcpus:虚拟机cpu个数，我这里给了一个核

--disk path:虚拟机虚拟硬盘的存放目录，我放在了/kvm_images/guest_os/mini_win7.img，size为硬盘大小，我给了10G

--network:虚拟机的网络连接方式，这里我使用的default，即NAT方式

--vnc:远程连接方式使用VNC

--vnclisten:vnc监听地址，这里使用0.0.0.0，即监听所有接口，默认监听127.0.0.1，所以外部不能直接访问到。

--vncport:VNC的端口，这里我使用了5901端口

--cdrom:虚拟光驱挂载的镜像路径，这里接上安装镜像的路径

回车后，安装过程就开始了，可以使用VNC工具去连接相应的端口进行安装了。如果忘记加--vnclisten=0.0.0.0，可以通过virsh edit mini-win7去编辑配置文件，将graphics中的配置更改如下：

```xml
<graphics type='vnc' port='5901' autoport='no' listen='0.0.0.0'>       
  <listen type='address' address='0.0.0.0'/>        
</graphics>
```

使用VNC连接上之后就是很普通的安装过程了。