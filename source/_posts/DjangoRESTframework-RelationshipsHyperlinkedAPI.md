---
title: DjangoRESTframework-关系与超链接API
categories:
  - Program
tags:
  - djangorestframework
date: 2017-10-11 12:24:04
---

### DjangoRESTframework 关系和超链接API(Relationships & Hyperlinked APIs)

目前，我们的API内部的关系是通过使用主键来表示的。在本教程的这一部分中，我们将通过使用超链接关系来改善我们的API的内聚性和可发现性。

<!--more-->

#### 为我们的API创建一个入口

目前为止我们已经有了snippets和users两个接口，但是我们没有一个单独的API的入口点。我们使用基于方法的视图来创建一个视图，并用上之前介绍过的@api_view装饰器。需要编辑的文件是snippets/views.py,添加上如下的代码：

```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework.reverse import reverse


@api_view(['GET'])
def api_root(request, format=None):
    return Response({
        'users': reverse('user-list', request=request, format=format),
        'snippets': reverse('snippet-list', request=request, format=format)
    })
```

这里需要注意两点。第一，我们使用了REST framework框架的reverse方法来返回完整限定(fully-qualified)的URLs.第二,URL规则使用名称来方便辨识的，这在之后编辑snippets/urls.py的时候将被提到。

#### 创建一个代码高亮的接口

我们的api还缺少一个关键点--代码高亮接口。

不像其他的API接口，我们不希望使用JSON,我们更愿意使用HTML来作为渲染输出。REST framework提供了两种风格的HTML渲染器，第一种先使用模版来处理预渲染HTMl(pre-rendered HTML)，第二种渲染器就像之前我们所用到的。

另外一件事情我们需要考虑的是创建代码高亮视图的时候不能使用现有的generic view，我们并没有返回一个对象实例，而是对象实例中的某个属性。

除了使用内建的generic view，我们将使用一个base class来展示实例，创建爱你我们自己的.get()方法。需要编辑的文件是snippets/views.py。

```python
from rest_framework import renderers
from rest_framework.response import Response

class SnippetHighlight(generics.GenericAPIView):
    queryset = Snippet.objects.all()
    renderer_classes = (renderers.StaticHTMLRenderer,)

    def get(self, request, *args, **kwargs):
        snippet = self.get_object()
        return Response(snippet.highlighted)
```

通常我们需要把新添加的视图增添到url配置中，编辑snippets/urls.py文件，添加我们的API:

```python
url(r'^$', views.api_root),
```

添加代码高亮接口的url：

```python
url(r'^snippets/(?P<pk>[0-9]+)/highlight/$', views.SnippetHighlight.as_view()),
```

#### 让我们的API变得超链接一点（Hyperlinking our API）

处理实体之间的关系是Web API设计中更具挑战性的方面之一。我们有很多不同的方式来表示一段关系:

1.使用主键

2.在不同的实体之间使用超链接（hyperlinking）

3.使用唯一标识

4.使用相关实体的默认字符串

5.在父表中嵌套相关的实体

6.一些其他的自定义的实现

REST framework支持以上全部的策略，可以将它们应用于转发或反向关系，或者将它们应用到通用的管理器，例如通用的外键。

在本例中，我们希望在实体之间使用超链接风格。为了这样做,我们会修改我们的序列化器来扩展HyperlinkedModelSerializer代替现有ModelSerializer。

ModelSerializer HyperlinkedModelSerializer有以下区别:

1.它默认不包含id字段

2.它使用HyperlinkedIdentityField,包含url字段。

3.使用HyperlinkedRelatedField来替代PrimaryKeyRelatedField表示关系

编辑snippets/serializers.py文件，使用超链接来改写序列化器：

```python
class SnippetSerializer(serializers.HyperlinkedModelSerializer):
    owner = serializers.ReadOnlyField(source='owner.username')
    highlight = serializers.HyperlinkedIdentityField(view_name='snippet-highlight', format='html')

    class Meta:
        model = Snippet
        fields = ('url', 'id', 'highlight', 'owner',
                  'title', 'code', 'linenos', 'language', 'style')


class UserSerializer(serializers.HyperlinkedModelSerializer):
    snippets = serializers.HyperlinkedRelatedField(many=True, view_name='snippet-detail', read_only=True)

    class Meta:
        model = User
        fields = ('url', 'id', 'username', 'snippets')
```

需要注意，我们已经添加了新的hightlight字段。这个字段和url字段是相同的类型。它指向的是snippet-hightlight这个url配置，而不是snippet-detail这个url配置。

因为我们包含了format suffixed URLs,支持类似于.json这个url，我们还需要在hightlight字段中指出，任何格式的后缀超链接都应该使用“。”html的后缀。

#### 让我们的URL配置已被命名（Make sure our URL patterns are named）

如果我们要有一个超链接的API，我们需要确定我们的URL路由。让我们看一下需要命名的UR路由。

1.我们的API的根路由，比如user-list和snippet-list。

2.我们的snippet序列化器包含的snippet-hightlight字段。

3.我们的序列化器包含的snippet-detail字段。

4.我们的snippet和user序列化器包含的url字段，它们默认指向{model_name}-detail，达到这样的效果snippet-detail和user-detail。

当我们把以上这些名称都添加到了URL配置中后，snippets/urls.py这个文件看起来应该是这个样子的。

```python
from django.conf.urls import url, include
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

# API endpoints
urlpatterns = format_suffix_patterns([
    url(r'^$', views.api_root),
    url(r'^snippets/$',
        views.SnippetList.as_view(),
        name='snippet-list'),
    url(r'^snippets/(?P<pk>[0-9]+)/$',
        views.SnippetDetail.as_view(),
        name='snippet-detail'),
    url(r'^snippets/(?P<pk>[0-9]+)/highlight/$',
        views.SnippetHighlight.as_view(),
        name='snippet-highlight'),
    url(r'^users/$',
        views.UserList.as_view(),
        name='user-list'),
    url(r'^users/(?P<pk>[0-9]+)/$',
        views.UserDetail.as_view(),
        name='user-detail')
])

# Login and logout views for the browsable API
urlpatterns += [
    url(r'^api-auth/', include('rest_framework.urls',
                               namespace='rest_framework')),
]
```

#### 添加分页功能

我们的user或者code snippet可能会返回大量的实例，所以我们最好能够让它们分页展示，也许API客户端能够获取到独立的页面。

我们可以使用分页来更改默认的列表风格，需要更改tutorial/settings.py文件：

```python
REST_FRAMEWORK = {
    'PAGE_SIZE': 10
}
```

注意：REST framework中的设置都被命名为一个名为“REST_FRAMEWORK”的单一字典，这有助于使它们与其他项目设置保持良好的分离。

如果需要的话，我们也可以自定义分页样式，但在本例中，我们只使用缺省值。

#### 浏览更改之后的API

如果我们打开一个浏览器打开API链接，您会发现现在可以通过跟踪超链接来使用这个API。

您还可以看到snippet上的“hightlight”链接，点击浏览能够看到代码高亮的HTML表示。

![image](/blogimg/api_root.png)

![image](/blogimg/highlight.png)

原文链接：http://www.django-rest-framework.org/tutorial/5-relationships-and-hyperlinked-apis/