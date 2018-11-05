---
title: 让Django在WSGI状态下返回静态文件
categories:
  - Program
tags:
  - django
date: 2018-11-05 16:08:47
---

最近在写一个api的时候想用浏览器在django服务部署后也能访问到静态文件，于是乎阅读了一下django的源码，发现在runserver的时候，静态文件是由django.contrib.staticfiles.views.serve来提供服务的。

<!--more-->

那么处理这个问题就简单了，只需要在urls.py中将serve视图引入，然后配置好自己需要的url就可以了。

```python
from django.contrib.staticfiles.views import serve
rlpatterns = [

    path('api/', include(app_url_patterns)),
    path('api/static/<path:path>', serve)

]
```

因为我的Nginx的反代设置的是/api到uWSGI，原本的settings里边的STATIC_URL也需要改一下。

```python
STATIC_URL = '/api/static/'
```

这样一来，无论是runserver还是通过Nginx+uWSGI部署后都能直接访问到静态文件。当然，不推荐这么做。