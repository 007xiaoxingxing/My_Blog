---
title: Nginx配合fcgiwrap部署bugzilla
categories:
  - Linux
tags:
  - nginx
date: 2019-02-13 14:28:27
---

由于测试需要，需要使用Nginx来部署运行bugzilla。而bugzilla是使用perl编写的cgi程序，然后便在网上找了很多关于Nginx部署bugzilla的文章，大多数也是抄来抄去的。于是乎只有自己摸索了。

<!--more-->

我的系统环境是Ubuntu18.04，安装配置fcgiwrap需要以下几步：

1. 安装fcgiwrap

   ```shell
   apt-get install fcgiwrap
   ```

2. 启动fcgiwrap服务

   ```shell
   systemctl start fcgiwrap
   ```

3. 配置Nginx，指向bugzilla目录。

   ```shell
   location /bugzilla/ {
           root /var/www;
       }
   
       location ~\.cgi$ {
               fastcgi_param SCRIPT_FILENAME /var/www$fastcgi_script_name;
               #fastcgi_param PATH_INFO $uri;
               #fastcgi_param BZ_CACHE_CONTROL 1;
               #return 200 $uri;
               include fastcgi_params;
               fastcgi_pass unix://run/fcgiwrap.socket;
           }
   ```

   

至于安装bugzilla的其它步骤，可以参考bugzilla官网的有关说明。额外就需要安装mysql，更改localconfig文件，执行perl install-module.pl --all和checksetup.pl两个文件。