---
title: Django快速上手笔记之安装Django
categories:
  - Program
tags:
  - python web Django
date: 2017-10-02 12:33:17
---

### Django简介

Django是一款开源的web框架，由python编写。采用了MVC的框架模式。Django最初是被用来开发用于管理劳伦斯出版集团旗下的一些以新闻内容为主的网站（CMS）,于2005年以BSD许可证发布。

<!--more -->

Django的设计之美：

- 对象关系映射（ORM）:以Python类形式定义数据模型，ORM将模型和关系数据库连接起来，得到一个十分容易使用的数据库API,同时在Django中也可以使用原生SQL语句。
- URL路由：使用正则表达式匹配URL,可以设计任意的URL，没有框架的特定限定，十分灵活。
- 模版系统：使用Django强大而可扩展的模版语言，可以分隔设计、内容和Python代码。具有可继承性。
- 表单处理：可以方便的生成各种表单模型，实现表单的有效性检验。可以方便的从你定义的模型实例生成对应的表单。
- Cache系统：可以挂载内存缓冲或其它的框架实现超级缓冲。
- 会话（session），用户登录与权限检查，快速开发用户会话功能。
- 国际化：内置国际化系统，方便开发出多种语言的网站。
- 自动化的管理界面：不需要你花大量的工作来创建人员管理和更新内容。Django自带一个Admin site，类似于内容管理系统。

总的来说，Django是一个大而全的框架，你所需要的它都有。那么怎么来使用它呢？怎么去一步一步的学习使用它呢？从本篇博文开始，我就要开始探索Django框架了。
### 安装Django
首先从安装开始，安装方式有两种：使用pip安装或源码安装。
pip安装很简单，在配置好python环境后，在命令提示符下或终端输入pip install django就行了。下面是我在自己的windowns电脑上的安装记录：
```powershell
PS C:\Users\star-chen> pip install django
Collecting django
  Downloading Django-1.11.5-py2.py3-none-any.whl (6.9MB)
    100% |████████████████████████████████| 7.0MB 149kB/s
Collecting pytz (from django)
  Downloading pytz-2017.2-py2.py3-none-any.whl (484kB)
    100% |████████████████████████████████| 491kB 377kB/s
Installing collected packages: pytz, django
Successfully installed django-1.11.5 pytz-2017.2
You are using pip version 8.1.2, however version 9.0.1 is available.
You should consider upgrading via the 'python -m pip install --upgrade pip' command.
```

源码安装：

从github上的Django项目上下载相关的release压缩包 
https://github.com/django/django/releases，
选择你所需要的版本。这里我下载1.11.5版本的zip格式压缩包。将下载到的压缩包解压到任何目录，在该目录按住shift键再鼠标右击，打开命令提示符，输入python选择你所需要的版本。这里我下载1.11.5版本的zip格式压缩包。将下载到的压缩包解压到任何目录，在该目录按住shift键再鼠标右击，打开命令提示符，输入python setup.py install即可。

```powershell
PS F:\一些源码\django-1.11.5> python .\setup.py install
```

为了日后使用方便，可以将Django的相关可执行文件添加到系统的可执行目录。

```
C:\Python27\Scripts
```

在我的机器上，没进行添加环境变量也可以正常调用django命令了，就省去这个步骤了。

```pow
PS C:\Users\star-chen> django-admin

Type 'django-admin-script.py help <subcommand>' for help on a specific subcommand.

Available subcommands:

[django]
    check
    compilemessages
    createcachetable
    dbshell
    diffsettings
    dumpdata
    flush
    inspectdb
    loaddata
    makemessages
    makemigrations
    migrate
    runserver
    sendtestemail
    shell
    showmigrations
    sqlflush
    sqlmigrate
    sqlsequencereset
    squashmigrations
    startapp
    startproject
    test
    testserver
```



OK,安装过程就是这么简单，接下来就是实际体验时刻了，下篇博文见。