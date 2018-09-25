---
title: DjangoRESTframework-序列化（Serializetion）
categories:
  - Program
tags:
  - Django djangorestframework
date: 2017-10-10 14:55:21
---

### DjangoRESTframework --------序列化（Serializetion）

#### 简介

这个教程将建立一个简单的Web API代码片段。通过这个例子将想你介绍众多的REST framework的特性，以及让你深入理解每个组件是怎么组合在一起运行的。

那就开始吧！

<!--more-->

#### 建立一个新的环境

载我们开始干活之前我们需要使用virtualenv建立一个新的环境，它能够确保我们的包的配置能够很好的和我们正在开发的其他项目保持很好的隔离。

```shell
virtualenv env
source env/bin/activeate  #windows:env/scripts/activate.bat
```

之后，我们进入到virtualenv环境，我们安装我们需要的package包。

```shell
pip install django
pip install djangorestframework
pip install pygments #这个用来实现代码高亮
```

小纸条：如果需要退出virtualenv环境，只需要输入deactivate即可。

#### 开始编码

现在可以开始编码了，在开始工作之前，首先建立一个新的工程先。

```shell
cd ~
django-admin startproject tutorial
cd tutorial
```

工程建立好之后，我们就可以创建一个新的app，我们将使用这个app来建立所需的Web API。

```shell
python manage.py startapp snippets
```

我们需要添加新的的snippets app，将rest_framework注册到INSTALLED_APPS。这些操作需要编辑tutorial/settings.py这个文件：

```python
INSTALLED_APPS = (
    ...
    'rest_framework',
    'snippets.apps.SnippetsConfig',
)
```

请注意，如果你使用Django的版本小于1.9，你需要将snippets.app.SnippetsConfig替换为snippets。Let's go on.

#### 创建一个下一步工作所需的model

为了这个教程能够继续下去，我们需要创建一个简单的Snippet model，它用来存储snippets的代码。编辑文件snippets/models.py。注意：添加注释是一个良好的编程习惯。

```python
from django.db import models
from pygments.lexers import get_all_lexers
from pygments.styles import get_all_styles

LEXERS = [item for item in get_all_lexers() if item[1]]
LANGUAGE_CHOICES = sorted([(item[1][0], item[0]) for item in LEXERS])
STYLE_CHOICES = sorted((item, item) for item in get_all_styles())


class Snippet(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    title = models.CharField(max_length=100, blank=True, default='')
    code = models.TextField()
    linenos = models.BooleanField(default=False)
    language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
    style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)

    class Meta:
        ordering = ('created',)
```

我们也需要创建一个用于初始化迁移我们的model，同时在第一次需要同步数据库。

```python
python manage.py makemigrations snippets
python manage.py migrate
```

#### 创建Serializer类

要实现我们的Web API，第一件需要做的事情是提供一个可以序列化和反序列化的snippets实例，比如用json来表示。我们可以定义serializers，它和Django的forms非常类似。在snippets目录中新建一个名为serializers.py的文件，文件内容如下：

```python
from rest_framework import serializers
from snippets.models import Snippet, LANGUAGE_CHOICES, STYLE_CHOICES


class SnippetSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(required=False, allow_blank=True, max_length=100)
    code = serializers.CharField(style={'base_template': 'textarea.html'})
    linenos = serializers.BooleanField(required=False)
    language = serializers.ChoiceField(choices=LANGUAGE_CHOICES, default='python')
    style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')

    def create(self, validated_data):
        """
        Create and return a new `Snippet` instance, given the validated data.
        """
        return Snippet.objects.create(**validated_data)

    def update(self, instance, validated_data):
        """
        Update and return an existing `Snippet` instance, given the validated data.
        """
        instance.title = validated_data.get('title', instance.title)
        instance.code = validated_data.get('code', instance.code)
        instance.linenos = validated_data.get('linenos', instance.linenos)
        instance.language = validated_data.get('language', instance.language)
        instance.style = validated_data.get('style', instance.style)
        instance.save()
        return instance
```

serializer类的第一部分定义了需要序列化和反序列化的成员变量，create()方法和update()方法定义了如何在调用serializer.save()方法时如何创建和修改实例。

serializer类和Django的form类非常类似，都包含了类似的数据有效性验证机制和丰富的方法，比如required，max_length,default。

数据域的标志能够控制serializer如何在特定的情况下显示，比如当在html页面中渲染{'base_template': ‘textarea.html’}，以上的标签等价于在Django的form类中的widget=widget.Textarea。这个在控制如何展示可视化API的过程中非常有用。接下来的的教程中我们将会体会到。

实际上我们可以使用ModelSerializer来节约时间，这个之后会用到，当然我们需要确保我们的serializer的定义是明确的。

#### 使用Serializers

在继续接下来的工作之前，我们需要进入到Django shell中。

```shell
python manage.py shell
```

好了，我们需要引入一些内容，并创建一些代码片段去处理它。

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser

snippet = Snippet(code='foo = "bar"\n')
snippet.save()

snippet = Snippet(code='print "hello, world"\n')
snippet.save()
```

我们已经有了一些snippets实例，让我们看看将它们序列化之后的样子吧。

```python
serializer = SnippetSerializer(snippet)
serializer.data
# {'id': 2, 'title': u'', 'code': u'print "hello, world"\n', 'linenos': False, 'language': u'python', 'style': u'friendly'}
```

这里，我们已经将model实例转换成了Python的原生数据类型，为了最终序列化结果，我们将数据转换成json格式。

```python
content = JSONRenderer().render(serializer.data)
content
# '{"id": 2, "title": "", "code": "print \\"hello, world\\"\\n", "linenos": false, "language": "python", "style": "friendly"}'
```

反序列化也是类似的，首先我们将Python原生数据类型转换成一个stream

```python
from django.utils.six import BytesIO

stream = BytesIO(content)
data = JSONParser().parse(stream)
```

然后我们将装换成原生数据类型的model实例重新序列化为一个Python object实例。

```python
serializer = SnippetSerializer(data=data)
serializer.is_valid()
# True
serializer.validated_data
# OrderedDict([('title', ''), ('code', 'print "hello, world"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])
serializer.save()
# <Snippet: Snippet object>
```

注意：这个API和Django forms非常类似。它们的相似性在我们开始写视图的时候将变得更加明显。

我们也可以序列化查询的数据集，只需要在序列化方法中添加many=True参数即可。

```python
serializer = SnippetSerializer(Snippet.objects.all(), many=True)
serializer.data
# [OrderedDict([('id', 1), ('title', u''), ('code', u'foo = "bar"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')]), OrderedDict([('id', 2), ('title', u''), ('code', u'print "hello, world"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')]), OrderedDict([('id', 3), ('title', u''), ('code', u'print "hello, world"'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])]
```

#### 使用ModelSerializers

我们的SnippetSerializer类已经包含了Snippet这个model中的非常多的信息。它能够让我们的代码显得更加的简洁。

同样的，Django也提供了form类和ModelForm类，REST framework提供了Serializer类和ModelSerializer类。

接下来让我们用ModelSerializer类来重构一下我们的API，打开文件snippets/serializers.py，将SnippetSerializer类替换成下面的样子：

```python
class SnippetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Snippet
        fields = ('id', 'title', 'code', 'linenos', 'language', 'style')
```

一个好的serializer的特点就是你能够在serializer实例中检查所有的数据成员，如果要打印数据表现，使用Django shell  :python manage.py shell

```python
from snippets.serializers import SnippetSerializer
serializer = SnippetSerializer()
print(repr(serializer))
# SnippetSerializer():
#    id = IntegerField(label='ID', read_only=True)
#    title = CharField(allow_blank=True, max_length=100, required=False)
#    code = CharField(style={'base_template': 'textarea.html'})
#    linenos = BooleanField(required=False)
#    language = ChoiceField(choices=[('Clipper', 'FoxPro'), ('Cucumber', 'Gherkin'), ('RobotFramework', 'RobotFramework'), ('abap', 'ABAP'), ('ada', 'Ada')...
#    style = ChoiceField(choices=[('autumn', 'autumn'), ('borland', 'borland'), ('bw', 'bw'), ('colorful', 'colorful')...
```

使用ModelSerializer类需要记住的重要的一点就是不要做特别的操作，它知识简单的创建一个序列化类：

- 自动识别字段

- 简单的实现默认的create()方法和update()方法。

#### 使用Serializer来编写Django views

让我们来尝试用新的serializer类来写一些API视图。这里，我们不会使用任何REST framework的其他特性，我们仅仅写点views来作为传统的Django views。编辑文件snippets/views.py，文件内容如下：

```python
from django.http import HttpResponse, JsonResponse
from django.views.decorators.csrf import csrf_exempt
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
```

访问我们的API的根路径能够列出当前存在的snippets或者创建一个新的snippets。

```python
@csrf_exempt
def snippet_list(request):
    """
    List all code snippets, or create a new snippet.
    """
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return JsonResponse(serializer.data, safe=False)

    elif request.method == 'POST':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data, status=201)
        return JsonResponse(serializer.errors, status=400)
```

注意，因为我们希望允许POST视图中的from表单，但是客户端不存在csrf token，我们需要添加注解csrf_exempt。这不是你平常需要做的，REST framework 的views能够控制敏感操作，这里的目的不是研究它。

我们也需要一个view来确保snippet的独立性，并且用来获取、更新和删除snippet。

```python
@csrf_exempt
def snippet_detail(request, pk):
    """
    Retrieve, update or delete a code snippet.
    """
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return HttpResponse(status=404)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return JsonResponse(serializer.data)

    elif request.method == 'PUT':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(snippet, data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data)
        return JsonResponse(serializer.errors, status=400)

    elif request.method == 'DELETE':
        snippet.delete()
        return HttpResponse(status=204)
```

最后，我们需要将这些view展示出来，创建一个文件snippets/urls.py:

```python
from django.conf.urls import url
from snippets import views

urlpatterns = [
    url(r'^snippets/$', views.snippet_list),
    url(r'^snippets/(?P<pk>[0-9]+)/$', views.snippet_detail),
]
```

我们还需要编辑应用根路径下的urls.py文件，用以配置app所需的url

```python
from django.conf.urls import url, include

urlpatterns = [
    url(r'^', include('snippets.urls')),
]
```

需要注意的是，这里还有一些小问题我们没有进行处理。如果我们发送的是json格式的数据，或者请求没有被适当的view处理，服务器将会反悔500“server error”错误。

#### 测试我们的第一个Web API

首先要开启测试服务器，先退出Django shell

```python
quit()
```

开启测试服务器：

```python
python manage.py runserver

Validating models...

0 errors found
Django version 1.11, using settings 'tutorial.settings'
Development server is running at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```



测试API我们可以使用curl或者httpie。httpie是使用python编写的工具，当然安装它也很简单：

```shell
pip install httpie
```

如果pip安装不上的话，还可以上使用easy_intall尝试一下：

```shell
easy_install httpie
```

使用httppie测试结果如下：

```python
PS E:\Study\Serializer_tutorial\tutorial> http http://127.0.0.1:8000/snippets/
HTTP/1.0 200 OK
Content-Length: 354
Content-Type: application/json
Date: Tue, 10 Oct 2017 06:46:43 GMT
Server: WSGIServer/0.1 Python/2.7.13
X-Frame-Options: SAMEORIGIN

[
    {
        "code": "foo = \"bar\"\n",
        "id": 1,
        "language": "python",
        "linenos": false,
        "style": "friendly",
        "title": ""
    },
    {
        "code": "print \"hello, world\n\"",
        "id": 2,
        "language": "python",
        "linenos": false,
        "style": "friendly",
        "title": ""
    },
    {
        "code": "print \"hello, world\n\"",
        "id": 3,
        "language": "python",
        "linenos": false,
        "style": "friendly",
        "title": ""
    }
]

```



当然，使用浏览器直接查看这个API也是可以的。浏览器的效果：

![image](/blogimg/rest-ser-1.png)
![image](/blogimg/rest-ser-2.png)

翻译自：http://www.django-rest-framework.org/tutorial/1-serialization/