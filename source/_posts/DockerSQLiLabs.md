---
title: 制作SQLiLabs的Docker镜像
categories:
  - Linux
tags:
  - sqli docker
date: 2018-06-06 10:09:32
---

最近想系统的来学习一下有关sql注入的基础知识。github上有一个印度大牛总结的sqli-labs项目，里边包含了常见的sql注入漏洞的程序，用来夯实基础还是够用的。但是搭建漏洞环境比较麻烦，需要LAMP或者WAMP环境，下软件都需要一大堆，放在自己的机器上也不方便跨设备访问，将其制作成docker镜像是一个不错的选择。

<!--more-->

##### 准备工作

在服务器上安装好docker环境。

##### 编写Dockerfile

Dockerfile是用来构建镜像的文本文件，包含了自定义的指令和格式。下面总结一下我这第一次次编写Dockerfile学到的点。

1. Dockerfile描述了组装镜像的步骤，其中的每条指令是单独执行的。除了FROM指令，其它指令都会在上一条指令生成的镜像的基础上执行，执行完之后生成一个新的镜像层，最终的镜像就是这样一层一层的叠加起来的。

2. Dockerfile的指令格式：

   ```shell
   #Comment 注释
   INSTRUCTION arguments #指令是不区分大小写的，但是为了和参数区分，习惯上大写。
   ```

   Dockerfile的第一条指令必须是FROM指令，用于指定构建镜像的基础镜像。

   指令包含有：FROM，MAINTAINER，RUN，CMD,EXPOSE, ENV,ADD,COPY,ENTRYPOINT,VOLUME,USER,WORKDIR,ONBUILD

3. 常见指令功能介绍

   ENV:声明环境变量

   FROM:指定基础镜像

   COPY:复制指定的文件到镜像中

   ADD:和COPY类似，但ADD支持使用URL添加，添加本地压缩归档文件到镜像自动解压

   RUN:在前一条指令创建的镜像的基础上创建一个容器，并在容器中执行命令，在命令结束后提交新的镜像，新镜像被下一条指令继续使用。

   CMD:提供容器运行时的默认值，一般用这个指令来设置容器开始运行时需要运行的脚本，比如开启服务。但会被docker run传递过来的参数覆盖

   ENTRYPOINT：和CMD类似，可以让容器在每次启动时执行相同的命令。

下边是我写的Dockerfile：

```shell
#base image
FROM centos:latest
# MAINTAINER
MAINTAINER chanxing9@gmail.com

RUN yum update && yum -y install httpd && yum -y \
    install mariadb-server mariadb-client && yum -y install git\
    yum -y install vim && yum -y install curl && yum -y install php\
    yum -y install php-mysql
RUN rm -rf /var/www/html/*
RUN git clone https://github.com/Audi-1/sqli-labs.git "/var/www/html/"
ADD ./sqli_lab.sh /etc/init.d/sqli_lab.sh
RUN chmod 755 /etc/init.d/sqli_lab.sh
CMD ["/etc/init.d/sqli_lab.sh"]
```

sqli_lab.sh

```shell
#!/bin/bash
/usr/sbin/httpd -DFOREGROUND &
/usr/libexec/mariadb-prepare-db-dir %n && /usr/bin/mysqld_safe --basedir=/usr &
/bin/bash
```

##### 构建镜像

将上述两个文件放入同一文件夹中，然后执行：

```shell
root@iZ3pwm54lxe41mZ:~/sqlilabs_docker# docker build -t sqli-labs .
```

等待构建完成，然后就可以查看到构建好的docker镜像：

```shell
root@iZ3pwm54lxe41mZ:~/sqlilabs_docker# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
sqli-labs           latest              fed060c2e56b        36 minutes ago      519MB
```

##### 运行镜像，尝试访问

使用docker run启动镜像即可：

```shell
root@iZ3pwm54lxe41mZ:~/sqlilabs_docker# docker run --privileged -p 82:80 -itd --name sqlilab sqli-labs
```

我这里将容器的80端口映射到了我服务器的82端口，给这个容器取了名字叫sqlilab，可以查看容器的运行情况：

```shell
root@iZ3pwm54lxe41mZ:~/sqlilabs_docker# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
fdf8d5664996        sqli-labs           "/etc/init.d/sqli_la…"   37 minutes ago      Up 37 minutes       0.0.0.0:82->80/tcp       sqlilab
```

访问一下82端口,并初始化一下数据库看看。

![img](/blogimg/dockersqlilab.png)