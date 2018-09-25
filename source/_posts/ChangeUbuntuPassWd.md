---
title: 忘记Ubuntu用户密码，怎么办?
categories:
  - Linux
tags:
  - liux
date: 2017-06-19 20:13:50
---

在使用Linux系统的时候，有时候难免会忘掉登录的用户名和密码，这个时候该怎么办呢？<!--more-->

#### 开机，在grub启动菜单选择"ubuntu 高级选项"

![image](/blogimg/grub_advance.png)

#### 选择恢复模式，按e，进入编辑模式

![image](/blogimg/grub_recovery.png)

找到第一个内核参数配置的地方，把“ro recovery nomodeset”替换为“quiet splash rw init=/bin/bash”，如下图所示：

![image](/blogimg/grub_kernel_param.png)

改好参数后，F10保存，系统会重新启动。

#### 重启后，命令行更改用户密码

![image](/blogimg/grub_reboot.png)

重启后，已经得到一个root权限的shell，可以查看当前的用户。现在就可以用

```bash
root@(none)/# passwd root
```

更改root用户的密码了。

