---
title: Ubuntu上安装使用KVM
categories:
  - Linux
tags:
  - kvm
date: 2018-03-05 14:18:38
---
KVM(kernel-based virtual machine),它是基于内核的完全虚拟化技术，有着较高的效率和完全的guest系统支持，可以安装各种guest系统。
<!--more-->
它具体的原理暂时不去探究，先安装一下软件，跑起来体验一把。
我的系统环境：Ubuntu 17.10
1. 安装需要的kvm软件、网络桥接工具、管理工具
```shell
sudo apt-get install qemu-kvm libvirt-bin bridge-utils qemu-system virt-manager virt-viewer qemu
```
耐心等待软件安装完成之后，先使用图形化的virt-manager来创建一台虚拟机。
```shell
sudo virt-manager
```
安装步骤依次设置，设置的步骤看起来和vm，vbox差不多，完了给它启动起来，看起来还不错。
![image](/blogimg/virtmanager.png)
![image](/blogimg/virtm-1.png)
![image](/blogimg/virtm-2.png)
![image](/blogimg/virtm-3.png)
![image](/blogimg/virtm-4.png)
![image](/blogimg/virtm-5.png)
![image](/blogimg/virtm-start.png)

除了使用图形化工具来创建虚拟机，还可以使用命令行工具来创建和管理kvm虚拟机，另外还有libvirt工具包提供了接口，可以使用其它编程语言来编写自定义的管理工具和云平台。