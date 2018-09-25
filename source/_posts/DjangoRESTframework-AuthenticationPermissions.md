---
title: DjangoRESTframework-认证与权限
categories:
  - Program
tags:
  - djangorestframework
date: 2017-10-11 12:22:18
---

### DjangoRESTframework  认证和权限（Authentication & Permissions）

直到目前，我们的API都没有包含任何的限制条件，任何人都能够编辑或者删除code snippet。我们应该添加一点高级的行为和规则以确保：

1.code snippet 总是和创建者保持相关

2.只有通过认证的用户能够创建snippet

3.只有创建者能够更新和删除snippet

4.未认证的用户的请求应该被限制为只读权限

<!--more-->

#### 给我们的model添加一点额外的信息

我们将为Snippet这个model进行一点改变。首先，添加一些成员变量，其中一个成员变量表示Snippet的创建者，另一个成员变量用来存储高亮的html之后的Snippet。在models.py文件中，对Snippet进行更改如下：

```python
owner = models.ForeignKey('auth.User', related_name='snippets', on_delete=models.CASCADE)
highlighted = models.TextField()
```

我们需要确保model被正确存储，我们再使用pygments生成高亮的html。我们需要再引入一点其他的东西：

```python
from pygments.lexers import get_lexer_by_name
from pygments.formatters.html import HtmlFormatter
from pygments import highlight
```

然后，我们添加一个.save()方法

```python
def save(self, *args, **kwargs):
    """
    Use the `pygments` library to create a highlighted HTML
    representation of the code snippet.
    """
    lexer = get_lexer_by_name(self.language)
    linenos = self.linenos and 'table' or False
    options = self.title and {'title': self.title} or {}
    formatter = HtmlFormatter(style=self.style, linenos=linenos,
                              full=True, **options)
    self.highlighted = highlight(self.code, lexer, formatter)
    super(Snippet, self).save(*args, **kwargs)
```

当以上工作完成之后，我们需要更新一下数据库中的表，通常的做法是创建一个migration来做这个事儿，但是在这个教程中，我们把它删掉，建个新的来用。

```shell
rm -f tmp.db db.sqlite3
rm -r snippets/migrations
python manage.py makemigrations snippets
python manage.py migrate
```

你可能需要创建一些不同的用户来测试这个API,最方便的办法是利用createsuperuser命令

```shell
python manage.py createsuperuser
```

#### 给咱们的用户model添加一些方法

现在我们已经有了几个用户，我们最好能够在我们的API中展示这些用户，那么创建给serializer吧。在serializers.py中添加如下代码：

```python
from django.contrib.auth.models import User

class UserSerializer(serializers.ModelSerializer):
    snippets = serializers.PrimaryKeyRelatedField(many=True, queryset=Snippet.objects.all())

    class Meta:
        model = User
        fields = ('id', 'username', 'snippets')
```

由于snippets和user 是一个相对的关系，所以在使用ModelSerializer类时，它不会被包含在内，所以我们需要为它添加一个显式的字段。

我们还会向views.py添加一些视图。我们希望仅对用户表示使用只读视图，因此我们将使用基于ListAPIView类和RetrieveAPIView类的类视图。

```python
from django.contrib.auth.models import User


class UserList(generics.ListAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer


class UserDetail(generics.RetrieveAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
```

确保引入了UserSerializer类

```python
from snippets.serializers import UserSerializer
```

最后，我们需要将这些视图添加到API中，并从URL conf中引用它们，并将以下内容添加到urls.py中的规则中。

```python
url(r'^users/$', views.UserList.as_view()),
url(r'^users/(?P<pk>[0-9]+)/$', views.UserDetail.as_view()),
```

#### 将Snippets和Users进行关联

现在，如果我们创建了一个code snippet，那么就没有办法将创建code snippet的用户与snippet实例关联起来。用户不是作为序列化表示的一部分发送的，而是作为传入请求的一个属性。

我们处理这个问题的方法是在代码片段视图上覆盖一个.performcreate()方法，它允许我们修改实例保存的方式，并处理传入请求或请求URL中隐式的任何信息。

在SnippetList视图类中，添加以下方法:

```python
def perform_create(self, serializer):
    serializer.save(owner=self.request.user)
```

我们的serializer的create()方法现在将通过一个附加的“ower”字段，以及来自请求的验证数据。

#### 更新serializer

现在这些代码片段与创建它们的用户相关联，让我们更新我们的SnippetSerializer来反映这一点。在serializers.py中添加以下字段到序列化器定义。

```python
owner = serializers.ReadOnlyField(source='owner.username')
```

注意:确保你还添加了“owner”，到内部元类的字段列表中。

这个领域正在做一些非常有趣的事情。源参数控制使用哪个属性来填充字段，并且可以指向序列化实例的任何属性。它还可以使用上面所示的虚线符号，在这种情况下，它将遍历给定的属性，与Django的模板语言使用的方式类似。

我们添加的字段是非类型化的read只读字段类，这与其他类型的字段形成对比，如CharField、布尔菲尔德等。未类型化的readonly字段始终是只读的，并且将用于序列化表示，但在反序列化时不会用于更新模型实例。我们也可以在这里使用CharField(readonly=True)。

#### 为视图添加所需的权限

现在，code snippet与用户相关，我们希望确保只有经过身份验证的用户能够创建、更新和删除代码片段。

REST framework包含了许多权限类，我们可以使用这些类来限制谁可以访问给定的视图。在这种情况下,我们正在寻找IsAuthenticatedOrReadOnly,这将确保经过身份验证的请求得到读写访问,和未经身份验证的请求得到只读访问。

先在视图模块中添加以下导入:

```python
from rest_framework import permissions
```

然后，将以下属性添加到SnippetList和SnippetDetail视图类中。

```python
permission_classes = (permissions.IsAuthenticatedOrReadOnly,)
```

#### 为可视化API添加登录功能

如果您现在打开浏览器并导航到可浏览的API，您将发现您不再能够创建新的代码片段。为了实现这一目的，我们需要能够以用户身份登录。

我们可以通过在我们的项目级url中编辑URLconf，添加一个用于可浏览的API的登录视图。urls.py文件。

```python
from django.conf.urls import include
```

在文件的最后，添加一个模式来包含可浏览的API的登录和注销视图。

```python
urlpatterns += [
    url(r'^api-auth/', include('rest_framework.urls',
                               namespace='rest_framework')),
]
```

r'^api-auth/'部分模式实际上可以是你想要使用的任何URL。惟一的限制是所包含的url必须使用“restframework”名称空间。在Django 1.9+中，REST框架将设置名称空间，因此您可以将其保留。

现在，如果您再次打开浏览器并刷新页面，您将看到页面右上角的“登录”链接。如果您以您之前创建的用户的身份登录，那么您将能够再次创建代码片段。

一旦您创建了一些代码片段，导航到'/users/'端点，并注意到该表示包含了一个列表.

#### 对象的等级权限

实际上，我们希望所有代码片段都能被任何人看到，但也要确保只有创建了代码片段的用户能够更新或删除它。

要做到这一点，我们需要创建一个自定义权限。

在代码片段应用程序中，创建一个新的文件，permissions.py。

```python
from rest_framework import permissions


class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    Custom permission to only allow owners of an object to edit it.
    """

    def has_object_permission(self, request, view, obj):
        # Read permissions are allowed to any request,
        # so we'll always allow GET, HEAD or OPTIONS requests.
        if request.method in permissions.SAFE_METHODS:
            return True

        # Write permissions are only allowed to the owner of the snippet.
        return obj.owner == request.user
```

现在，我们可以通过在SnippetDetail视图类上编辑permissionclass属性，将这个定制权限添加到我们的代码片段实例端点。

请确保导入IsOwnerOrReadOnly类。

```python
from snippets.permissions import IsOwnerOrReadOnly
```

现在，如果您再次打开浏览器，您会发现“DELETE”和“PUT”操作只出现在代码片段实例端点上，如果您登录的是创建代码片段的相同用户。

#### 为API添加认证功能

因为我们现在有了API的一组权限，所以如果我们想要编辑任何代码片段，就需要对请求进行身份验证。我们还没有设置任何身份验证类，因此目前应用了缺省值，即SessionAuthentication和BasicAuthentication。

当我们通过web浏览器与API进行交互时，我们可以登录，然后浏览器会话将为请求提供所需的身份验证。

如果我们以编程方式与API交互，我们需要在每个请求上显式地提供身份验证凭证。

如果我们尝试创建一个不进行身份验证的代码片段，我们会得到一个错误:

```shell
http POST http://127.0.0.1:8000/snippets/ code="print 123"

{
    "detail": "Authentication credentials were not provided."
}
```

我们可以通过包括我们先前创建的一个用户的用户名和密码来获得成功的请求。

```shell
http -a tom:password123 POST http://127.0.0.1:8000/snippets/ code="print 789"

{
    "id": 1,
    "owner": "tom",
    "title": "foo",
    "code": "print 789",
    "linenos": false,
    "language": "python",
    "style": "friendly"
}
```

#### 总结

现在，我们已经获得了一组相当细粒度的Web API权限，以及系统用户和他们创建的代码片段的端点。

原文：http://www.django-rest-framework.org/tutorial/4-authentication-and-permissions/

