---
title: DjangoRESTframework-快速开始
categories:
  - Program
tags:
  - python djangorestframework
date: 2017-10-10 11:28:45
---

### DjangoRESTframework  快速开始

我们将开始建立一个简单的API，让admin用户去查看和编辑系统中用户和用户组。

<!--more-->

#### 建立工程

```shell
# 为工程建立存储目录
mkdir tutorial
cd tutorial

# 创建一个virtualenv，便于隔离我们的包的本地依赖
virtualenv env
source env/bin/activate  # 在windows环境使用命令 `env\Scripts\activate`

# 在virtualenv中安装django和djangorestframework框架
pip install django
pip install djangorestframework

# 建立一个新的project和一个新的app
django-admin.py startproject tutorial .  # 注意结尾的 '.' 号
cd tutorial
django-admin.py startapp quickstart
cd ..
```

工程的目录结构应该如下：

```shell
$ pwd
<some path>/tutorial
$ find .
.
./manage.py
./tutorial
./tutorial/__init__.py
./tutorial/quickstart
./tutorial/quickstart/__init__.py
./tutorial/quickstart/admin.py
./tutorial/quickstart/apps.py
./tutorial/quickstart/migrations
./tutorial/quickstart/migrations/__init__.py
./tutorial/quickstart/models.py
./tutorial/quickstart/tests.py
./tutorial/quickstart/views.py
./tutorial/settings.py
./tutorial/urls.py
./tutorial/wsgi.py
```

在引用外部模块的时候使用工程的命名空间来避免名称的冲突。接下来可以首先同步你的数据库：

```shell
python manage.py migrate
```

同时我们也需要建立和初始化admin用户，并设置 admin用户的密码为password123。稍后我们将在我们的实例应用中认证其他用户。

```shell
python manage.py createsuperuser
```

当你同步好数据库并初始化用户之后，打开工程的app目录（这里是quickstart），开始编码吧。

#### 序列化器（Serializers）

首先，我们需要定义一些序列化器（serializers），让我们创建一个新的模块（module），路径是tutorial/quickstart/serializers.py，这个将用来作为数据展示。

```python
from django.contrib.auth.models import User, Group
from rest_framework import serializers


class UserSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = User
        fields = ('url', 'username', 'email', 'groups')


class GroupSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Group
        fields = ('url', 'name')
```

注意：这里我们继承关系的是HyperlinkedSerializers,你也可以使用主键或者其他众多的关系，但是hyperlinking是很好的RESTful设计。

#### 视图（Views）

我们最好再写点视图（views）吧，打开文件tutorial/quickstart/views.py，然后把下面的内容敲进去。

```python
from django.contrib.auth.models import User, Group
from rest_framework import viewsets
from tutorial.quickstart.serializers import UserSerializer, GroupSerializer


class UserViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows users to be viewed or edited.
    """
    queryset = User.objects.all().order_by('-date_joined')
    serializer_class = UserSerializer


class GroupViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows groups to be viewed or edited.
    """
    queryset = Group.objects.all()
    serializer_class = GroupSerializer
```

比起写非常多的视图，我们将所有视图的许多共同行为（common behavior）放进了ViewSets类中。如果有需要，我们可以非常容易的将他们拆成单独的视图，但是使用viewsets可以保持良好的视图逻辑和简洁的代码。

#### URL配置（URLs)

接下来我们可以定义API的url路径了，文件是tutorial/urls.py

```python
from django.conf.urls import url, include
from rest_framework import routers
from tutorial.quickstart import views

router = routers.DefaultRouter()
router.register(r'users', views.UserViewSet)
router.register(r'groups', views.GroupViewSet)

# Wire up our API using automatic URL routing.
# Additionally, we include login URLs for the browsable API.
urlpatterns = [
    url(r'^', include(router.urls)),
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework'))
]
```

由于我们使用了viewsets代替了views，我们的API可以自动成成URL配置，只需要将viewsets注册到router class中即可。

同样，如果我们需要对API的URL获得更多的控制，也可以使用传统的class-based视图，然后自己配置URL。

最后，我们可以包含登录和登出视图来浏览我们的API。这是可选的，但是如果你的API需要登录认证和需要浏览API，这两个视图也非常有用。

#### 设置（Settings）

我们需要一点点全局设置，比如需要分页或者想让我们的API只能由admin用户使用，这些配置都需要在tutorial/settings.py中。

```python
INSTALLED_APPS = (
    ...
    'rest_framework',
)

REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAdminUser',
    ],
    'PAGE_SIZE': 10
}
```

好了，代码我们就写完了。

#### 测试我们的API

首先把测试服务器打开，让应用跑起来。

```shell
python manage.py runserver
```

服务开启正常后，我们就可以使用curl或者浏览器来测试api了。

```shell
PS E:\Study\tutorial\tutorial> http -a admin:password123 http://127.0.0.1:8000/users/
HTTP/1.0 200 OK
Allow: GET, POST, HEAD, OPTIONS
Content-Length: 103
Content-Type: application/json
Date: Tue, 10 Oct 2017 03:05:10 GMT
Server: WSGIServer/0.1 Python/2.7.13
Vary: Accept, Cookie
X-Frame-Options: SAMEORIGIN

[
    {
        "email": "admin@star-chen.com",
        "groups": [],
        "url": "http://127.0.0.1:8000/users/1/",
        "username": "admin"
    }
]
```

浏览器中是这个画风，当然你得先登录。

![image](/blogimg/quick_start.png)

ok,成功运行。