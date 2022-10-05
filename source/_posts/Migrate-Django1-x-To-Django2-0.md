---
title: 从Django1.x迁移至2.0需要注意的几个点
categories:
  - Program
tags:
  - django
date: 2018-03-21 13:54:37
---

最近在把项目从django1.x往django2.x迁移，过程遇到了不少问题，大都根据报错信息进行了处理，现将过程中遇到的几个典型的问题记录如下：

<!--more-->

##### URL的匹配规则

不再支持patterns，下边这种写法是不行的了。

```python
from django.conf.urls import patterns, url
from django.contrib.auth import views as auth_views

import views

urlpatterns = patterns('',
    url(r'^logout/$', views.logout, name="logout"),
    url(r'^login/$', views.LoginView.as_view(), name="login")
)
```

如果想保留以上的正则路由匹配的方式，可以微调成以下的样子：

```shell
from django.conf.urls import url
from django.contrib.auth import views as auth_views

from . import views

urlpatterns = [
    url(r'^logout/$', views.logout, name="logout"),
    url(r'^login/$', views.LoginView.as_view(), name="login")
]
```

对比一下可以看出，url函数一致，只是把patterns函数直接换为[]，去掉''这个规则。

另外一种使用正则路由的方式如下：

```python
from django.urls import include, re_path

urlpatterns = [
    re_path(r'^index/$', views.index, name='index'),
    re_path(r'^bio/(?P<username>\w+)/$', views.bio, name='bio'),
    re_path(r'^weblog/', include('blog.urls')),
    ...
]
```

此外，Django2.0提供了新的path语法来定义路由：

```python
from django.urls import path

from . import views

urlpatterns = [
    path('articles/2003/', views.special_case_2003),
    path('articles/<int:year>/', views.year_archive),
    path('articles/<int:year>/<int:month>/', views.month_archive),
    path('articles/<int:year>/<int:month>/<slug:slug>/', views.article_detail),
]
```

具体说明和使用方法请参见Django官方文档。

##### MIDDLEWARE的变化

`settings.py`文件，`MIDDLEWARE_CLASSES`更改成了`MIDDLEWARE`，这个一定要注意。

```python
MIDDLEWARE = (
    'webapp.lib.acl.AclMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
)

```

为了将之前的1.x的middleware迁移至2.0的middleware，可以这么做：

```python
from .globals import set_thread_globals
from django.utils.deprecation import MiddlewareMixin  #引入此包

class Global(MiddlewareMixin):  #继承MiddlewareMixin
    def process_request(self, request):
        set_thread_globals('request', request)
        set_thread_globals('user', getattr(request, 'user', None))
        set_thread_globals('engine_exceptions', [])
```

这样做之后，MiddlewareMixin的 `__call__`方法会去调用process_request等方法。具体详情可参见官方文档说明：https://docs.djangoproject.com/en/2.0/topics/http/middleware/#view-middleware

当然有更简单的编写midlleware的方法：

```python
class SimpleMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        # One-time configuration and initialization.

    def __call__(self, request):
        # Code to be executed for each request before
        # the view (and later middleware) are called.

        response = self.get_response(request)

        # Code to be executed for each request/response after
        # the view is called.

        return response
```

​

##### User.is_authenticated 和User.is_anonymous

之前的`User.is_authenticated()`和`User.is_anonymous()`方法更改成了属性：`User.is_authenticated`和`User.is_anonymous`。

##### 外键的on_delete属性

外键的on_delete关系必须指明，不可省略。

##### django-filters的变化

之前项目中使用的django-filter是0.x版本，对于现在的drf 3.7.x版本支持不是那么好，所以更换为了1.x版本。它有两点发生了变化。

MethodFilter在1.x中被去掉了，所以下面的引用方式应当做响应改变

```python
from django_filters import (FilterSet, MethodFilter)  #之前的引用方式
from django_filters import (FilterSet, Filter)   #现在应将MethodFilter改为Filter即可
```

MethodFilter的写法从：

```python
import django_filters
class CustomerAccountFilter(django_filters.FilterSet):
    is_staff = django_filters.MethodFilter(name="is_staff", action="is_staff_filter")
```

变为了：

```python
import django_filters
class CustomerAccountFilter(django_filters.FilterSet):
    is_staff = django_filters.Filter(name="is_staff", method="is_staff_filter")
```

​

这几点是我在迁移的时候遇到的几个问题，可能还有其它需要注意点的点，遇到之后再做补充。