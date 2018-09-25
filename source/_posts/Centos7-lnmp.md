---
title: CentOS 7.2 搭建lnmp环境
date: 2017.3.11 11:08:20
categories: Linux
tags:
- CentOS
- PHP
- Nignx
- MariaDB
- LNMP
---

### Centos7 安装php7
由于centos自己的软件仓库中的php版本为5.4，而yii2框架需要的php版本必须是5.4及以上。索性直接一步到位到7.0。为图方便就不使用编译安装的方式，而是采用yum安装方式.
#### 首先rpm安装php7.0的yum源
```bash
[root@vultr ~]# rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
```
<!-- more -->
#### yum安装php7
```bash
[root@vultr ~]# yum install php70w
```
### yum安装php-fpm
```bash
[root@vultr ~]# yum install php70w-fpm
```
####  查看php版本
```bash
[root@vultr ~]# php -v
PHP 7.0.16 (cli) (built: Feb 18 2017 10:25:02) ( NTS )
Copyright (c) 1997-2017 The PHP Group
Zend Engine v3.0.0, Copyright (c) 1998-2017 Zend Technologies
```

### 安装MariaDB

```bash
[root@vultr ~]# yum install mariadb-server mariadb
```
启动MariaDB

```bash
[root@vultr ~]# systemctl start mariadb
```
刚刚安装好数据库，是没有设置root用户密码的，这样子显然是不安全的。在初次安装时，可以使用MariaDB自带的安全安装命令设置root用户密码。

```bash
[root@vultr ~]# mysql_secure_installation 

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] Y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!

```
根据提示设置好root帐号的密码，移除测试数据库等等，MariaDB就安装完成了。

### Nginx+php-fpm，让Nginx能够解析php请求

在安装好Nginx和php-fpm之后，需要配置一下Nginx。
```
server {

		listen 80;
		server_name www.xxx.com;
		
		root /var/www/html;
		index index.html index.htm index.php;
		
		location ~ \.php$ {
			
			include fastcgi_params;
			fastcgi_pass 127.0.0.1:9000;
			fastcgi_index index.php;
			fastcgi_param SCRIPT_FILENAME /var/www/html$fastcgi_script_name;
		
		}

	}
```

开启php-fpm
```bash
[root@vultr lgsmb]# systemctl start php-fpm
```
在web根目录写个phpinfo
```php
<?php
	phpinfo();
?>
```
访问一下，如果看到了phpinfo的页面，证明Nginx已经能够正常解析php文件了。
