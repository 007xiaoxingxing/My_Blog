---
title: DjangoRESTframework-基于类的视图（class-based views）
categories:
  - Program
tags:
  - djangorestframework
date: 2017-10-10 17:13:09
---

### DjangoRESTframework    基于类的视图（Class-based Views）

我们也能够使用基于类的视图来替代基于方法的视图。让我们来看看这个强大的模式，让我们重用许多基本方法，帮助我们保持代码的DRY(Don't Repeat Yourself)。

<!--more-->

#### 利用基于类的视图重写我们的API

我们将使用基于类的视图来重写咱们的视图，这个需要重构一下views.py了

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from django.http import Http404
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status


class SnippetList(APIView):
    """
    List all snippets, or create a new snippet.
    """
    def get(self, request, format=None):
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    def post(self, request, format=None):
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

到目前为止，它看起来和我们之前写的非常相似，但是我们已经很好的分离了不同的HTTP请求方法，继续改。

```python
class SnippetDetail(APIView):
    """
    Retrieve, update or delete a snippet instance.
    """
    def get_object(self, pk):
        try:
            return Snippet.objects.get(pk=pk)
        except Snippet.DoesNotExist:
            raise Http404

    def get(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    def put(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk, format=None):
        snippet = self.get_object(pk)
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

这看起来很好嘛。但是看起来还是和之前差不太多，继续重构urls.py,更好的支持基于类的视图。

```python
from django.conf.urls import url
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

urlpatterns = [
    url(r'^snippets/$', views.SnippetList.as_view()),
    url(r'^snippets/(?P<pk>[0-9]+)/$', views.SnippetDetail.as_view()),
]

urlpatterns = format_suffix_patterns(urlpatterns)
```

哇，好了。打开开发服务器，一切都将和之前一样的运转。

#### 使用mixins（Use mixins）

使用基于类的视图的一大好处就是允许我们能够容易的组织可重用的行为。

create/retrieve/updae/delete操作，在我们创建的支持模型的视图中都区别不大。这些行为都已经在REST framework中通过mixins很好的实现了。

来看看使用mixin类如何组织我们的视图，同样是更改views.py

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import mixins
from rest_framework import generics

class SnippetList(mixins.ListModelMixin,
                  mixins.CreateModelMixin,
                  generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
```

我们花点时间来仔细的研究一下这里边到底都发生了什么。我们使用GenericAPIView,添加进了ListModelMixin和CreateModelMixin。

基类提供了核心的方法，mixin类提供了.list()和.create()行为。我们可以明确的绑定get和post方法在不同的行为上。非常简单，还够用。

```python
class SnippetDetail(mixins.RetrieveModelMixin,
                    mixins.UpdateModelMixin,
                    mixins.DestroyModelMixin,
                    generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)

    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)

    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)
```

很相似，我们再次用GenericAPIView类提供关键方法，minxins提供了.retrieve(),.update(),.destroy()操作。

#### 使用基于generic class的视图

使用基于mixins的视图已经帮我们减少了代码，还可以再省一点。REST framework提供了一整套mixed-in geric的视图。我们再次为views.py瘦身一下。

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import generics


class SnippetList(generics.ListCreateAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer


class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```

哇塞，这也太简洁了。这很Django！