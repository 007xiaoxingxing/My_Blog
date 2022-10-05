---
title: 搭建用于激活Windows系统和Office的Docker
categories:
  - Linux
tags:
  - kms
date: 2018-08-07 13:31:37
---

今天一位同事在激活自己的windows 10专业版系统的时候遇到点问题，于是乎有了下文。本来可以使用`https://hub.docker.com/r/luodaoyi/kms-server/`提供的kms.luodaoyi.info这台kms服务器进行激活操作，但是telnet尝试了一下，发现目标服务器的1688端口没有开放，那只好自己拉一下镜像到自己的服务器上跑一个kms了。

<!--more-->

#### 拉取镜像

```shell
docker pull luodaoyi/kms-server
```

#### RUN镜像

```shell
docker run -itd -p 1688:1688 --name kms luodaoyi/kms-server
```

然后就可以使用这台服务器来进行系统的激活操作了。

#### 激活系统

```shell
C:\slmgr /skms xxx.xxx.com
C:\slmgr /ato
```

如果当前电脑使用的不是vlk版本专业版系统的key，可以先根据docker镜像的说明页面中的操作将key换掉，再进行kms激活操作即可。

#### 激活Office

激活office需要进入到ospp.vbs所在目录，然后执行如下两条命令即可。

```shell
C:\xxx\csscript ospp.vbs /sethst:xxx.xx.com
C:\xxx\csscript ospp.vbs /act
```

与系统一样，office需要使用专业版才可以用kms的方式激活。