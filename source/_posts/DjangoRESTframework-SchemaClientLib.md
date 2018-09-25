---
title: DjangoRESTframework-Schema和客户端类库
categories:
  - Program
tags:
  - djangorestframework
date: 2017-10-11 12:28:03
---

### DjangoRESTframework    Schemas & client libraries

shema是一种机器可读的文档，描述了可用的API端点、它们的url以及它们支持的操作。

shema可以成为自动生成文档的有用工具，还可以用于驱动与API交互的动态客户端库。

<!--more-->

#### Core API

为了提供shema支持REST框架使用了Core API。

Core API是用于描述API的文档规范。它用于提供可用接口的内部表示格式和API公开的可能交互。它既可以使用服务器端，也可以使用客户端。

在使用服务器端时，Core API允许API支持对多种schema或hypermedia格式的呈现。

在使用客户端时，Core API允许动态驱动的客户端库，可以与公开支持的模式或hypermedia格式的任何API交互。

#### 添加schema

REST framework支持显式定义的模式视图或自动生成的模式。由于我们使用的是viewset和路由器，所以我们可以简单地使用自动模式生成。

为了包含API schema，您需要安装coreapi 包。

```shell
pip install coreapi
```

现在，我们可以通过在URL配置中包含一个自动生成的模式视图来包含API的模式。

```python
from rest_framework.schemas import get_schema_view

schema_view = get_schema_view(title='Pastebin API')

urlpatterns = [
    url(r'^schema/$', schema_view),
    ...
]
```

如果您在浏览器中访问API接口，您现在应该可以看到corejson表示作为一个选项可用。

我们还可以从命令行请求schema，通过在Accept标头中指定所需的内容类型。

```ptyhon
$ http http://127.0.0.1:8000/schema/ Accept:application/coreapi+json
HTTP/1.0 200 OK
Allow: GET, HEAD, OPTIONS
Content-Type: application/coreapi+json

{
    "_meta": {
        "title": "Pastebin API"
    },
    "_type": "document",
    ...
```

默认的输出样式是使用核心的JSON编码。

其他的模式格式，如Open API也得到了支持。

#### 使用命令行的客户端

既然我们的API公开了一个schema 接口，我们就可以使用动态客户端库来与API交互。为了演示这一点，让我们使用核心API命令行客户端。

命令行客户端可以作为coreapi-cli包:

```python
 pip install coreapi-cli
```

现在检查命令行上是否有可用的命令。

```shell
$ coreapi
Usage: coreapi [OPTIONS] COMMAND [ARGS]...

  Command line client for interacting with CoreAPI services.

  Visit http://www.coreapi.org for more information.

Options:
  --version  Display the package version number.
  --help     Show this message and exit.

Commands:
...
```

首先，我们将使用命令行客户端加载API schema。

```python
$ coreapi get http://127.0.0.1:8000/schema/
<Pastebin API "http://127.0.0.1:8000/schema/">
    snippets: {
        highlight(id)
        list()
        read(id)
    }
    users: {
        list()
        read(id)
    }
```

我们还没有经过身份验证，所以现在我们只能看到只读的接口，这与我们如何设置API的权限是一致的。

让我们尝试使用命令行客户端来列出已存在的代码片段:

```shell
$ coreapi action snippets list
[
    {
        "url": "http://127.0.0.1:8000/snippets/1/",
        "id": 1,
        "highlight": "http://127.0.0.1:8000/snippets/1/highlight/",
        "owner": "lucy",
        "title": "Example",
        "code": "print('hello, world!')",
        "linenos": true,
        "language": "python",
        "style": "friendly"
    },
    ...
```

一些API接口需要指定的参数。例如，为了获取特定代码片段的突出显示HTML，我们需要提供一个id。

```shell
$ coreapi action snippets highlight --param id=1
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">

<html>
<head>
  <title>Example</title>
  ...
```

#### 客户端登录认证

如果我们想要创建、编辑和删除代码片段，我们需要作为一个有效的用户进行身份验证。在本例中，我们将使用基本的身份验证。

使用您的实际用户名和密码来替换下面的用户名和密码。

```python
$ coreapi credentials add 127.0.0.1 <username>:<password> --auth basic
Added credentials
127.0.0.1 "Basic <...>"
```

现在，如果我们再次获取shema，我们应该能够看到完整的可用交互集。

```shell
$ coreapi reload
Pastebin API "http://127.0.0.1:8000/schema/">
    snippets: {
        create(code, [title], [linenos], [language], [style])
        delete(id)
        highlight(id)
        list()
        partial_update(id, [title], [code], [linenos], [language], [style])
        read(id)
        update(id, code, [title], [linenos], [language], [style])
    }
    users: {
        list()
        read(id)
    }
```

我们现在可以与这些接口进行交互。例如，要创建一个新的代码片段:

```shell
$ coreapi action snippets create --param title="Example" --param code="print('hello, world')"
{
    "url": "http://127.0.0.1:8000/snippets/7/",
    "id": 7,
    "highlight": "http://127.0.0.1:8000/snippets/7/highlight/",
    "owner": "lucy",
    "title": "Example",
    "code": "print('hello, world')",
    "linenos": false,
    "language": "python",
    "style": "friendly"
}
```

然后删除一个片段:

```shell
$ coreapi action snippets delete --param id=7
```

除了命令行客户端，开发人员还可以使用客户端库与您的API进行交互。Python客户机库是第一个可用的，并且计划很快发布一个Javascript客户机库。

关于定制模式生成和使用核心API客户端库的更多细节，您需要参考完整的文档。

#### 回顾一下我们的工作

有了非常少的代码，我们现在得到了一个完整的粘贴本Web API，它是完全Web浏览的，包括一个由模式驱动的客户端库，并且完成了身份验证、每个对象的权限和多个渲染器格式。

我们已经经历了设计过程的每一步，并看到如果我们需要定制任何东西，我们就可以逐步地使用普通的Django视图。

您可以在GitHub上查看最终的教程代码，或者在沙箱中尝试这个示例。

原文链接:http://www.django-rest-framework.org/tutorial/7-schemas-and-client-libraries/