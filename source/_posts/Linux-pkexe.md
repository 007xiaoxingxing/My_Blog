---
title: Linux实用命令之pkexec
categories:
  - Linux
tags:
  - linux
date: 2018-05-28 16:54:39
---

今天在帮一个群友解决关于www-data用户实用sudo问题时，由于错误的用vim修改了/etc/sudoers文件导致sudo命令无法使用的问题。经过一番搜索，学习到了一个非常有用的命令工具`pkexec`。

<!--more-->

先问问man，pkexec是一个什么工具：

```shell
PKEXEC(1)                                              pkexec                                              PKEXEC(1)

NAME
       pkexec - Execute a command as another user

SYNOPSIS
       pkexec [--version] [--disable-internal-agent] [--help]

       pkexec [--user username] PROGRAM [ARGUMENTS...]

DESCRIPTION
       pkexec allows an authorized user to execute PROGRAM as another user. If username is not specified, then the
       program will be executed as the administrative super user, root.
```

可以看出，pkexec可以使用其它用户身份来执行一些命令，如果--user留空，则默认使用root用户来执行相关命令。然后我使用如下方式修改/etc/sudoers文件为正确配置：

```shell
user@localhost:~$ pkexec chmod 0777 /etc/sudoers #修改文件权限为777，使普通用户也可以修改
user@localhost:~$ vim /etc/sudoers #重新编辑配置文件为正确的
user@localhost:~$ pkexec chmod 0440 /etc/sudoers  #记住一定将该配置文件权限修改回0440
```

此次经历得到了一个教训，千万不要直接编辑/etc/sudoers文件，一定要使用`visudo`命令，这样子如果配置文件写错了，还有检查机制。