---
title: Django快速上手笔记之HelloWorld
categories:
  - Program
tags:
  - python web Django
date: 2017-10-02 13:09:39
---

在上篇博文中，成功的安装好了Django框架，在这篇博文中，先建个helloworl工程试试水。再根据helloworld工程进一步介绍Django框架中的各个模块怎么配置和使用。废话不多讲，上图。

<!--more-->

在Linux环境下用命令行也可以很好的进行Django项目的开发，为了学习的方便我在windowns环境下使用Pycharm集成开发环境作示范。

### 创建项目

1.点击file->new project,选择Django项目，输入你想创建的项目名称和所在路径，选择python解释器版本，点击create即可生成一个Django项目。

![image](/blogimg/django-step-1.png)

生成的项目的目录结构如下：

![image](/blogimg/django-step-2.png)

### 创建Django APP

在一个Django项目中，可以包含多个APP，一个APP可以作为分系统、子模块、功能部件等待。创建方法为在pycharm的terminal中输入命令

```cmd
python manage.py startapp hello
```

![imange](/blogimg/django-step-3.png)

创建成功的APP目录结构如下：

![imange](/blogimg/django-step-4.png)

### 编写路由

在project目录下的urls.py文件中添加需要的路由

```python
from django.conf.urls import url
from django.contrib import admin

from Hello import views  #导入对应的app的views文件

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'index/',views.index)  #自己需要的路由，第一个参数是url匹配的正则表达式，第二个参数是对应的views的业务逻辑方法
]
```

### 编写对应的业务逻辑

业务逻辑代码需要在APP中views.py文件中编写：

```python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.shortcuts import render
from django.shortcuts import HttpResponse #导入HttpResponse模块

# Create your views here.
def index(request):
    return  HttpResponse("Hello Django!")  #直接返回一个字符串
```

完成了以上两步，我们就将index这个url路径指向了views中的index()方法，它完成的动作便是给访问/index/这个url的浏览器返回“Hello Django!”字符串。

### 让服务跑起来

1. 在pycharm的terminal中执行：

```cmd
python manange.py runserver 127.0.0.1:8000
```

2. 直接点击pycharm工具栏中的绿色run图标即可。

### 看看效果

![image](/blogimg/django-step-5.png)

服务开启正常，用浏览器访问一下。

![imange](/blogimg/django-step-6.png)

OK,服务运行正常，说明这个工程创建是成功的。