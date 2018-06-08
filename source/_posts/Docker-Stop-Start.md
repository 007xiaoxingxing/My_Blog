---
title: 从docker stop/start出发，实现自动启动容器内应用
categories:
  - Linux
tags:
  - docker
date: 2018-06-07 10:54:14
---

接上篇，sqli-labs镜像能够实现docker run的时候自动启动mysql和apache进程，但是一旦docker异常退出或者执行docker stop之后，再次执行docker start启动容器时，mysql和apache不会自动启动，而只有mysql会启动起来，于是乎有了如下的折腾过程。

<!--more-->

##### 执行docker stop会发生什么？

先来看看官方文档怎么说：

> ```
> 用法：docker stop [OPTIONS] CONTAINER [CONTAINER...]
> 参数：--time , -t	10	Seconds to wait for stop before killing it
> 描述：The main process inside the container will receive SIGTERM, and after a grace period, SIGKILL.
> ```

讲得明白一点就是执行docker stop命令之后，会给容器内pid=1的进程发送SIGTERM信号，然后等待-t时间，如果在-t指定的时间内，进程没有正常结束，会发送SIGKILL信号强制结束pid=1这个进程，然后退出容器。这里的-t指定的时间可以预留给容器内的程序做一些善后工作，比如继续处理接收的请求，保存数据，做一些清理工作等。

于是我进入容器，查看一下pid=1的进程是什么，没错，就是在Dockerfile中CMD指令指定的那个脚本。

```shell
[root@bd79479d225d /]# ps -aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  11680   152 pts/0    Ss   Jun06   0:00 /bin/bash /etc/init.d/sqli_lab.sh
```

##### 为什么docker stop后，再docker start之后，apache没有启动呢？

首先docker stop sqlilab，再docker start sqlilab,查看容器内进程

```shell
[root@bd79479d225d /]# ps -aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  11680  1332 pts/0    Ss   03:13   0:00 /bin/bash /etc/init.d/sqli_lab.sh
root         6  0.0  0.0  11680   584 pts/0    S    03:13   0:00 /bin/bash /etc/init.d/sqli_lab.sh
root         7  0.0  0.1  11820  1680 pts/0    S+   03:13   0:00 /bin/bash
root        48  0.0  0.1  11816  1624 pts/0    S    03:13   0:00 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
mysql      193  0.4  8.1 775012 82832 pts/0    Sl   03:13   0:00 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/pl
root       216  0.0  0.1  11820  1884 pts/1    Ss   03:13   0:00 /bin/bash
root       231  0.0  0.1  51708  1720 pts/1    R+   03:14   0:00 ps -aux
```

这里可以发现sqli_lab.sh被执行了两次，其中一个pid=1另外一个pid=6。并且mysql进程是存在的，那为什么apache进程没有启动起来呢？

继续来查看一下/etc/init.d/sqli_lab.sh

```shell
#!/bin/bash

/usr/sbin/httpd -DFOREGROUND &
/usr/libexec/mariadb-prepare-db-dir %n && /usr/bin/mysqld_safe --basedir=/usr &
/bin/bash
```

也许发现一些端倪了，应该是进程前后台设置混乱导致的。这里具体的细节原因限于我目前的知识水平无法解释。

##### 解决办法

要保持docker在后台运行不要退出，一定要保留一个前台进程，这里我是使用的/bin/bash，其实上边的apache以前台模式运行就可以替代/bin/bash了。然后我尝试了一下/usr/bin/mysqld_safe --basedir=/usr，发现这条命令也会保持在前台不退出，那么久先把mysql给放到后台去，然后令apache在前台保持就好了。更改后的脚本如下：

```shell
#!/bin/bash
/usr/libexec/mariadb-prepare-db-dir %n
/usr/bin/mysqld_safe --basedir=/usr > /dev/null 2>&1 &
/usr/sbin/httpd -DFOREGROUND
```

保存后，重新stop/start，mysql和apache均能正常启动。问题暂时解决。



2018.6.8 更新

​	由于晚上我将服务器关机了，今天再次启动容器的时候发现问题依然存在，真是捉急。于是乎只有换个方法了，还是需要使用supervisor来管理mysql和httpd进程。

于是更改Dockerfile，增加安装supervisor

```shell
RUN yum -y install epel-release && yum install supervisor -y
```

编写分别用于控制httpd和mysqld的配置文件：

```ini
[program:mysql]
command=/bin/bash -c "/usr/libexec/mariadb-prepare-db-dir && /usr/bin/mysqld_safe --basedir=/usr"
autostart=true
autorestart=true
user=root
numprocs=1
stdout_logfile=/tmp/supervisor-mysql.log
```

```ini
[program:httpd]
command=/usr/sbin/httpd -DFORGROUND
autostart=true
autorestart=true
user=root
numprocs=1
stdout_logfile=/tmp/supervisor-httpd.log
```

将配置文件加入到镜像中：

```shell
ADD ./mysql.ini /etc/supervisord.d/mysql.ini
ADD ./httpd.ini /etc/supervisord.d/httpd.ini
CMD ["/usr/bin/supervisord", "-n"]
```

这次问题终于得到解决了。