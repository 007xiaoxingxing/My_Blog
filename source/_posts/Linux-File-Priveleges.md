---
title: Linux学习笔记之文件权限
categories:
  - Linux
tags:
  - 文件权限
  - 学习笔记
date: 2017-05-23 17:32:43
---

在Linux系统中，一切皆为文件。当然文件权限的管理也相对复杂一些了。 Linux继续沿用了Unix管理文件权限的办法，其允许用户和组基于每个文件和目录的一组安全性设置来访问文件。但是对于使用“ls”命令查看Linux系统上的文件、目录和设备的权限时，输出的信息也没有弄得十分清楚，特地又重新学习了一下，免得遗忘，还是做下笔记。

<!-- more -->

### 理解ls命令的输出

```bash
[root@vultr etc]# ls -la
total 1660
drwxr-xr-x 85 root  root   4096 May 22 07:14 .
dr-xr-xr-x 18 root  root   4096 Dec 16 14:24 ..
-rw-r--r--  1 root  root     16 Nov 14  2016 adjtime
-rw-r--r--  1 root  root   1518 Jun  7  2013 aliases
-rw-r--r--  1 root  root  12288 Dec 27 09:14 aliases.db
drwxr-xr-x  2 root  root   4096 Mar 11 04:11 alternatives
-rw-r--r--  1 root  root     55 Nov  4  2016 asound.conf
drwxr-xr-x  2 root  root   4096 Dec 16 14:26 avahi
drwxr-xr-x  2 root  root   4096 May 22 07:31 bash_completion.d
-rw-r--r--  1 root  root   2853 Nov  5  2016 bashrc
drwxr-xr-x  2 root  root   4096 Mar  3 03:23 binfmt.d
-rw-r--r--  1 root  root     38 Nov 29 18:12 centos-release
-rw-r--r--  1 root  root     51 Nov 29 18:12 centos-release-upstream
```

上面是我查看/etc目录下的文件时的部分输出。其输出结果有七个字段，下面来具体解释一下具体含义：

#### 第一个字段：drwxr-xr-x，这个字段描述的是文件和目录权限码。

这个字段的第一个字符代表的含义有以下几种情况：

- -代表文件；例子：-rw-r--r--  1 root  root   2853 Nov  5  2016 bashrc，表示bashrc是一个文件
- d代表目录；例子：drwxr-xr-x  2 root  root   4096 Mar  3 03:23 binfmt.d，表示binfmt.d是一个目录
- l代表链接；例子：lrwxrwxrwx  1 root root          25 Jan  7 13:32 initctl -> /run/systemd/initctl/fifo，表示inictl是一个链接到/run/systemd/initctl/fifo的链接
- c代表字符设备；例子：crw--w----  1 root tty       4,   6 Jan  7 13:32 tty6，表示tty6是一个字符设备
- b代表块设备；例子：brw-rw----   1 root disk    253,   0 4月  13 19:25 vda，表示vda是一个块设备，它其实这台云服务器的硬盘。
- s代表socket文件；例子：srwxrwxrwx   1 mysql mysql        0 4月  13 19:25 mysql.sock，mysql.sock就是一个socket文件。
- p代表管道文件；

之后的3组三字符的编码：rwxr-xr-x，表示了该文件的三重访问权限

- r代表对象是可读的；
- w代表对象是可写的；
- x代表对象是可执行的;

如果没有某种权限，在该权限的位置会出现单破折线。这3组权限分别对应3个安全级别：

- 对象的属主，也就是文件的所有者；
- 对象的属组，也就是文件所有者所在的用户组；
- 系统其他用户；

理论上就是这样子了，我们去找个实际的例子来分析一下：

> 例子：drwxr-xr-x  2 root  root   4096 Mar 11 04:11 alternatives

![image](/blogimg/ls_detail.png)

但是有时候我们更改文件权限是由三位数字来表示的，例如"chmod 755 file"，这又是怎么回事呢？这里有一张表可以参考一下：

![image](/blogimg/linux_umask.png)

把权限类型弄明白了，然后学习一下怎么更改Linux的文件权限的命令吧。

chmod:用于改变文件的权限，SUID,SGID,SBIT等等

1. chmod命令的格式

   > chmod options mode file

   mode参数后边可以跟上八进制模式或符号模式来设置文件权限

   ```bash
   [root@VM_0_49_centos ~]# ls -la file_test #原本file_test的权限为644
   -rw-r--r-- 1 root apache 0 5月  23 21:55 file_test
   [root@VM_0_49_centos ~]# chmod 760 file_test 
   [root@VM_0_49_centos ~]# ls -la file_test #更改后为760
   -rwxrw---- 1 root apache 0 5月  23 21:55 file_test
   ```

chmod还有另外一种表示方式，较为复杂一些，命令格式如下：
> chmod [ugoa...][+-=][rwxXstugo...]

这样虽然复杂一些，但是更容易理解不是吗？
- u  代表用户
- g  代表组
- o  代表拥有者
- a  代表上述所有

接下来的符号表示你是想在现有权限的基础上增加权限（+），还是移除权限（-），还是将权限设置为后面的值（=）

最后的符号代表你想赋予的权限。详细的有：

- X  如果对象是目录或者它已经有执行权限，则赋予执行权限
- s  运行时重新设置UID或GID
- t  保留文件或目录
- u  将权限设置为属主一样
- g  将权限设置为跟属组一样
- o  将权限设置为跟其他用户一样

举个栗子：

```bash
$ chmod o+r newfile #给文件所有者加上读权限
```

#### 第二个字段表示该文件的链接数，包括硬链接和软链接

创建软链接：ln -s

创建硬链接：ln

#### 第三个字段表示该文件的所有者

文件所有者可以通过chown命令来更改。命令格式：

```bash
$  chown [OPTION]... [OWNER][:[GROUP]] FILE...
```

当然，chown也可以同时更改文件所属的组。-R参数可以递归的更改子目录和文件的所属关系，-h参数可以改变文件的所有符号链接文件的所属关系。

#### 第四个字段表示该文件所在的用户组

文件所属的用户组可通过chgrp来进行更改。命令格式：

```bash
$   chgrp [OPTION]... GROUP FILE...
```

#### 第五个字段表示该文件的文件大小，单位字节

#### 第六个字段表示文件的修改时间

#### 第七个字段表示文件名

