---
title: Ubuntu 17.04 安装PostgreSQL
categories:
  - Linux
tags:
  - postgres
date: 2017-10-12 13:44:11
---

### 在Ubuntu17.04 Server中安装PostgreSQL与基本使用方法

PostgreSQL是自由的对象-关系型数据库服务器（数据库管理系统），在灵活的BSD-风格许可证下发行。它在其他开放源代码数据库系统（比如MySQL和Firebird），和专有系统比如Oracle、Sybase、IBM的DB2和Microsoft SQL Server之外，为用户又提供了一种选择。

<!--more-->

#### 安装

在ubuntu的软件仓库中已经包含了Postgre软件包 ，使用apt可以很方便的安装。

```shell
xing@ubuntu-server:~$ sudo apt-get install postgresql postgresql-contrib
```

然后坐等安装完成。安装完成之后，会在系统中创建一个名为postgres的用户，查看/etc/passwd可以看到：

```shell
postgres:x:113:119:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
```

这个Linux用户默认和Postgres管理员身份进行了关联，接下来我们就可以使用这个用户来对Postgres进行管理

#### 使用Postgre角色和创建数据库



首先切换到postgres用户:

```shell
xing@ubuntu-server:~$ sudo -i -u postgres 
#出现这个提示符表示已经切换到了postgres用户
postgres@ubuntu-server:~$ 
```

使用psql命令登录PostgreSQL控制台：

```shell
postgres@ubuntu-server:~$ psql
psql (9.6.5)
Type "help" for help.

postgres=# 
```

然后可以在PostgreSQL控制台为postgres用户先设置给密码：

```shell
postgres=# \password postgres
```

再创建一个名为db_test数据库用户，并设置它的密码为db_test：

```shell
postgres=# create user db_test with password 'db_test';
```

创建数据库并将数据库的权限的所有者指定为db_test用户:

```shell
postgres=# create database testdb owner db_test;
```

然后再把testdb的权限给db_test用户：

```shell
postgres=# grant all privileges on database testdb to db_test;
```

退出psql控制台的方法是:

```shell
postgres=# \q
```

除了使用psql控制台管理数据库，还可以使用shell命令行来进行配置:

在postgres用户的命令提示符下创建数据库用户：

```shell
postgres@ubuntu-server:~$ createuser --interactive
```

在其他用户的命令终端下可以这样:

```shell
xing@ubuntu-server:~$ sudo -u postgres createuser --interactive
```

在命令终端创建数据库：

​	在postgres用户的终端：

```shell
postgres@ubuntu-server:~$ createdb testdb
```

​	在其他用户的终端；

```shell
sudo -u postgres createdb testdb
```

#### 登录数据库

添加了新的用户和数据库之后，就可以使用新的用户登录到数据库了：

```shell
xing@ubuntu-server:~$ psql -U db_test -d testdb -h localhost -p 5432
Password for user db_test: 
psql (9.6.5)
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

testdb=> 
```

#### psql控制台的一些命令

- \h：查看SQL命令的解释，比如\h select。
- \?：查看psql命令列表。
- \l：列出所有数据库。
- \c [database_name]：连接其他数据库。
- \d：列出当前数据库的所有表格。
- \d [table_name]：列出某一张表格的结构。
- \du：列出所有用户。
- \e：打开文本编辑器。
- \conninfo：列出当前数据库和连接的信息。

参考资料:

http://www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html

https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04