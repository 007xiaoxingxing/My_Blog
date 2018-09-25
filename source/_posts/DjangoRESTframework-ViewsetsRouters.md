---
title: DjangoRESTframework-视图集合和路由
categories:
  - Program
tags:
  - djangorestframework
date: 2017-10-11 12:26:23
---

### DjangoRESTframework    视图集合和路由（ViewSets & Routers）

REST framework包含一个用于处理ViewSets的抽象，它允许开发人员专注于对状态和API的交互进行建模，并根据公共约定自动构建URL。

ViewSet类做的事情和View类做的事情差不多，除开它们都提供read或者update操作，而不是处理get或者put。

ViewSet类仅在最后时刻绑定到一组方法处理程序，当它被实例化为一组视图时，通常使用一个Router类来处理它，为你定义复杂的URL配置。

<!--more-->

#### 使用ViewSets重构

让我们来看看当前的视图，并将它们重构为view sets。

首先，让我们将UserList和UserDetail视图重构为一个单独的UserViewSet。我们可以删除这两个视图，并将它们替换为单个类:

```python
from rest_framework import viewsets

class UserViewSet(viewsets.ReadOnlyModelViewSet):
    """
    This viewset automatically provides `list` and `detail` actions.
    """
    queryset = User.objects.all()
    serializer_class = UserSerializer
```

这里我们使用了ReadOnlyModelViewSet类来自动提供默认的“只读”操作。我们仍然设置了queryset和serializer_class属性，就像我们使用常规视图时那样，但是我们不再需要向两个单独的类提供相同的信息。

接下来，我们将替换SnippetList、SnippetDetail和SnippetHighLight视图类。我们可以删除这三个视图，然后用一个类替换它们。

```python
from rest_framework.decorators import detail_route
from rest_framework.response import Response

class SnippetViewSet(viewsets.ModelViewSet):
    """
    This viewset automatically provides `list`, `create`, `retrieve`,
    `update` and `destroy` actions.

    Additionally we also provide an extra `highlight` action.
    """
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
    permission_classes = (permissions.IsAuthenticatedOrReadOnly,
                          IsOwnerOrReadOnly,)

    @detail_route(renderer_classes=[renderers.StaticHTMLRenderer])
    def highlight(self, request, *args, **kwargs):
        snippet = self.get_object()
        return Response(snippet.highlighted)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

这一次，我们使用了ModelViewSet类，以便获得完整的默认读和写操作集。

请注意，我们还使用@detail_route装饰器创建了一个自定义动作，名为highlight。这个装饰器可以用来添加任何不符合标准的create/updat/delete样式的自定义入口点。

使用@detail_route装饰器的自定义操作会在缺省情况下响应GET请求。如果我们想要一个响应POST请求的操作，我们可以使用methods参数来指定。

缺省情况下，自定义操作的url依赖于方法名本身。如果您想要更改url的构造方式，可以将url_path作为修饰器关键字参数。

#### 将Viewsets和URL分别进行绑定

当我们定义URL配置时，处理程序方法只能绑定到操作。为了查看底层的情况，让我们首先从我们的视图中创建一组视图。

在url.py文件将我们的ViewSet类绑定到一组具体的视图中。

```python
from snippets.views import SnippetViewSet, UserViewSet, api_root
from rest_framework import renderers

snippet_list = SnippetViewSet.as_view({
    'get': 'list',
    'post': 'create'
})
snippet_detail = SnippetViewSet.as_view({
    'get': 'retrieve',
    'put': 'update',
    'patch': 'partial_update',
    'delete': 'destroy'
})
snippet_highlight = SnippetViewSet.as_view({
    'get': 'highlight'
}, renderer_classes=[renderers.StaticHTMLRenderer])
user_list = UserViewSet.as_view({
    'get': 'list'
})
user_detail = UserViewSet.as_view({
    'get': 'retrieve'
})
```

注意，通过将http方法绑定到每个视图所需的操作，我们如何从每个ViewSet类创建多个视图。

既然我们已经将我们的资源绑定到具体的视图中，我们可以像往常一样使用URL conf注册视图。

```python
urlpatterns = format_suffix_patterns([
    url(r'^$', api_root),
    url(r'^snippets/$', snippet_list, name='snippet-list'),
    url(r'^snippets/(?P<pk>[0-9]+)/$', snippet_detail, name='snippet-detail'),
    url(r'^snippets/(?P<pk>[0-9]+)/highlight/$', snippet_highlight, name='snippet-highlight'),
    url(r'^users/$', user_list, name='user-list'),
    url(r'^users/(?P<pk>[0-9]+)/$', user_detail, name='user-detail')
])
```

#### URL路由

因为我们使用的是ViewSet类而不是视图类，所以我们实际上不需要自己设计URL。将资源连接到视图和url的约定可以通过路由器类自动处理。我们所要做的就是用一个路由器注册合适的视图集，然后让它完成剩下的工作。

现在的urls.py文件是这样的

```python
from django.conf.urls import url, include
from snippets import views
from rest_framework.routers import DefaultRouter

# Create a router and register our viewsets with it.
router = DefaultRouter()
router.register(r'snippets', views.SnippetViewSet)
router.register(r'users', views.UserViewSet)

# The API URLs are now determined automatically by the router.
# Additionally, we include the login URLs for the browsable API.
urlpatterns = [
    url(r'^', include(router.urls)),
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework'))
]
```

用路由器注册viewset类似于提供一个urlpattern。我们包括两个参数:视图的URL前缀和viewset本身。

我们使用的DefaultRouter类也会自动为我们创建API根视图，因此我们现在可以从views模块中删除api_root方法。

#### views和viewsets之间的权衡

使用viewset是一个非常有用的抽象。它有助于确保URL约定在您的API中是一致的，最小化您需要编写的代码量，并允许您集中精力于您的API所提供的交互和表示，而不是URL conf的细节。

这并不意味着它总是正确的方法。在使用基于类的视图而不是基于函数的视图时，还需要考虑类似的权衡。使用viewset比单独构建视图更显式。

原文链接：http://www.django-rest-framework.org/tutorial/6-viewsets-and-routers/