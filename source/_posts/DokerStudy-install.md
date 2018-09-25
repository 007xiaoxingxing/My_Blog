---
title: 动手学Docker系列---在操作系统上安装Docker
categories:
  - Linux
tags:
  - docker
date: 2017-06-02 18:48:13
---

### What is Docker?

  Docker是当前相当流行的一种应用容器引擎，通过Docker容器，可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化。Docker容器完全使用沙箱机制，相互之间不会有任何的接口，可以让应用得到高等级的隔离，相比于虚拟机，Docker容器更加轻量级，性能开销更低。

### Docker的一些应用场景

- web应用的自动化打包和发布；
- 自动化测试和持续集成、发布；
- 在服务性环境中部署和调整数据库或其他的后台应用；
- 从头编译或者扩展现有的OpenShift或cloud Foundry平台来搭建自己的PaaS环境；

### Docker所解决的问题

- 更有序的组织
- 更高的可移植性
- 保护你的机器，提高安全性

既然Docker容器有这么多的优点，那怎么才能使用上它呢？这篇文章就来记录一下怎么去安装Docker。

<!-- more -->

### 安装Docker

我所使用的操作系统是Ubuntu 14.04 LTS

Docker官方的安装说明地址： https://docs.docker.com/engine/installation/linux/ubuntu/  先仔细阅读一下官方的安装说明。

> Recommended extra packages for Trusty 14.04
>  Unless you have a strong reason not to, install the `linux-image-extra-*` packages, which allow Docker to use the `aufs` storage drivers.
>

看起来我使用的14.04还需要安装一下内核扩展，那安装一下先

```bash
xing@ubuntu-compile:~$ sudo apt-get install linux-image-extra-$(uname -r) linux-image-extra-virtual
正在读取软件包列表... 完成
正在分析软件包的依赖关系树       
正在读取状态信息... 完成       
linux-image-extra-3.19.0-33-generic 已经是最新的版本了。
linux-image-extra-3.19.0-33-generic 被设置为手动安装。
下列软件包是自动安装的并且现在不需要了：
  linux-headers-3.19.0-25 linux-headers-3.19.0-25-generic
  linux-image-3.19.0-25-generic linux-image-extra-3.19.0-25-generic
Use 'apt-get autoremove' to remove them.
将会安装下列额外的软件包：
  linux-image-3.13.0-119-generic linux-image-extra-3.13.0-119-generic
  linux-image-generic
建议安装的软件包：
  fdutils linux-doc-3.13.0 linux-source-3.13.0 linux-tools
  linux-headers-3.13.0-119-generic
下列【新】软件包将被安装：
  linux-image-3.13.0-119-generic linux-image-extra-3.13.0-119-generic
  linux-image-extra-virtual linux-image-generic
升级了 0 个软件包，新安装了 4 个软件包，要卸载 0 个软件包，有 89 个软件包未被升级。
需要下载 52.0 MB 的软件包。
解压缩后会消耗掉 195 MB 的额外空间。
您希望继续执行吗？ [Y/n] 
```

官方网站上介绍了两种安装方式，一是从软件仓库安装，而是下载deb安装包，手动安装。

- 从软件仓库安装

```bash
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

```bash
$ sudo apt-key fingerprint 0EBFCD88

pub   4096R/0EBFCD88 2017-02-22
      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22
```

```bash
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

```bash
$ sudo apt-get update
$ sudo apt-get install docker-ce
```



- 下载deb包，手动安装

浏览器打开  [https://download.docker.com/linux/ubuntu/dists/](https://download.docker.com/linux/ubuntu/dists/)  选择适合系统版本的deb包下载。

我选择手动下载deb包进行安装。

```bash
xing@ubuntu-compile:~$ wget https://download.docker.com/linux/ubuntu/dists/trusty/pool/stable/amd64/docker-ce_17.03.1~ce-0~ubuntu-trusty_amd64.deb
```

吐槽一句，下载的速度这是太慢了------------终于下载好了

```bash
xing@ubuntu-compile:~$ sudo dpkg -i docker-ce_17.03.1~ce-0~ubuntu-trusty_amd64.deb 
[sudo] password for xing: 
正在选中未选择的软件包 docker-ce。
(正在读取数据库 ... 系统当前共安装有 144928 个文件和目录。)
正准备解包 docker-ce_17.03.1~ce-0~ubuntu-trusty_amd64.deb  ...
正在解包 docker-ce (17.03.1~ce-0~ubuntu-trusty) ...
dpkg: 依赖关系问题使得 docker-ce 的配置工作不能继续：
 docker-ce 依赖于 libltdl7 (>= 2.4.2)；然而：
  未安装软件包 libltdl7。
 docker-ce 依赖于 libsystemd-journal0 (>= 201)；然而：
  未安装软件包 libsystemd-journal0。

dpkg: 处理软件包 docker-ce (--install)时出错：
 依赖关系问题 - 仍未被配置
正在处理用于 man-db (2.6.7.1-1ubuntu1) 的触发器 ...
正在处理用于 ureadahead (0.100.0-16) 的触发器 ...
在处理时有错误发生：
 docker-ce

```

好像出了点问题，需要依赖，那么安装一下依赖吧。

```bash
xing@ubuntu-compile:~$ sudo apt-get -f install
xing@ubuntu-compile:~$ sudo dpkg -i docker-ce_17.03.1~ce-0~ubuntu-trusty_amd64.deb 
(正在读取数据库 ... 系统当前共安装有 145148 个文件和目录。)
正准备解包 docker-ce_17.03.1~ce-0~ubuntu-trusty_amd64.deb  ...
docker stop/waiting
正在将 docker-ce (17.03.1~ce-0~ubuntu-trusty) 解包到 (17.03.1~ce-0~ubuntu-trusty) 上 ...
正在设置 docker-ce (17.03.1~ce-0~ubuntu-trusty) ...
docker start/running, process 2629
正在处理用于 man-db (2.6.7.1-1ubuntu1) 的触发器 ...
正在处理用于 ureadahead (0.100.0-16) 的触发器 ...

```

这些安装好了！！加载一个Hello World镜像尝试一下

### Docker Hello World

```bash
xing@ubuntu-compile:~$ sudo docker run dockerinaction/hello_world
Unable to find image 'dockerinaction/hello_world:latest' locally
latest: Pulling from dockerinaction/hello_world
a3ed95caeb02: Pull complete 
1db09adb5ddd: Pull complete 
Digest: sha256:cfebf86139a3b21797765a3960e13dee000bcf332be0be529858fca840c00d7f
Status: Downloaded newer image for dockerinaction/hello_world:latest
hello world

```

可以看到，docker是先检查本地是否有hello_world镜像，发现不存在，再去镜像市场拉取到本地，再进行运行的。最后已经打印出了hello world，说明我们的docker已经安装成功了！

再来运行hello_world看看有什么不一样

```bash
xing@ubuntu-compile:~$ sudo docker run dockerinaction/hello_world
hello world

```

直接打印出了hello world ，说明镜像已经存在，docker直接是运行的本地的镜像。

-----------安装篇完

参考资料：

 https://docs.docker.com/engine/installation/linux

《Docker实战》

http://www.runoob.com/docker/ubuntu-docker-install.html

http://www.docker.org.cn/book/docker/what-is-docker-16.html