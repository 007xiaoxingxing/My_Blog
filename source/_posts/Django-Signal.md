---
title: Django-信号
categories:
  - Program
tags:
  - Django
date: 2017-10-12 11:47:16
---

### Django的信号机制（Signals）

Django自带一个信号调度程序允许*receiver*函数在某个动作出现的时候去获取通知。信号非常有用，当你需要你的代码去执行某些事件的时候同时正在发生其他事件。你还能够创建你自己的信号这样一来其他人可以在某个事件发生的时候获得通知。

<!--more-->

其中Django提供了一组内建信号，比如下面这些：

1. django.db.models.signals.pre_save

   在模型 save()方法调用之前或之后发送。

2. django.db.models.signals.pre_delete

   在模型delete() 方法调用之前或之后发送。

3. django.db.models.signals.m2m_changed

   模型上的 ManyToManyField 修改时发送。

4. django.core.signals.request_started

   Django建立或关闭HTTP 请求时发送。

​

上面的每个信号的完整使用方法可以参考Django官方的详细文档。

#### 如何使用Django中的信号呢？

首先了解一下Django Signal的处理流程：

![image](/blogimg/signal_process.png)

那么我就用个小例子来演示一下如何使用signal

##### 应用目的

之前根据DjangoRESTframework官方教程做了一个简单的管理代码片段的api，我想当有用户创建代码片段的时候打印点日志，这可以通过signal来完成。

#### 使用receiver装饰器处理signal

在应用下新建一个signals.py文件，内容如下：

```python
# -*- coding:utf-8 -*-
from django.db.models.signals import post_save
from django.dispatch import receiver
from snippets.models import Snippet
import logging

#使用了receiver装饰器来将on_snippet_create注册成监听函数，用以监听Snippet这个model的post_save事件
@receiver(post_save, sender=Snippet)
#这个就是post_save事件对应的回调函数
def on_snippet_create(sender,instance, **kwargs):
    logging.debug("New Snippet")
    print "New snippet created!"
    print "The code = %s" % instance.code
```

然后需要在app引入一下signals.py这个文件，不然它是不会被执行的。一个可以的位置是在apps.py中，在ready()方法中注册。apps.py文件内容如下：

```python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.apps import AppConfig
```

```python
 class SnippetsConfig(AppConfig):
    name = 'snippets'

    def ready(self):
        # import signal handlers
        import snippets.signals
```


  接下来可以开启测试服务器，看一下是否能够监听到model的post_save事件。

  ![image](/blogimg/receiver_result.png)

  可以看到控制台已经打印出了log信息，说明监听成功。

  #### 使用connect,send来处理自定义信号

除了使用Django内建信号，我们还可以自定义所需的信号，那么具体的步骤如下：

##### 定义信号

信号的定义是这样的：

class Signal([providing_args=list])

所有信号都是 django.dispatch.Signal 的实例。`providing_args`是一个列表，由信号将提供给监听者的参数名称组成。理论上是这样，但是实际上并没有任何检查来保证向监听者提供了这些参数。

那么我们可以这样来定义一个信号：

```python
import django.dispatch

snippet_saved = django.dispatch.Signal(providing_args=['obj'])
```

这段代码中我们声明了一个叫做snippet_saved的信号，它可以接收一个叫做obj的参数。

##### 发送信号

Django中可以有两种方法去发送信号

- Signal.send(sender, **kwargs)


- Signal.send_robust(sender, **kwargs)

调用 Signal.send()来发送信号。你必须提供`sender` 参数（大多数情况下它是一个类），并且可以提供尽可能多的关键字参数。

我们可以这样子来发送信号：

```python
    def save(self, *args, **kwargs):
        """
        把snippet使用pygments来生成高亮的html代码
        :param args:
        :param kwargs:
        :return:
        """
        lexer = get_lexer_by_name(self.language)
        linenos = self.linenos and 'table' or False
        options = self.title and {'title': self.title} or {}
        formatter = HtmlFormatter(style=self.style, linenos=linenos,
                                  full=True, **options)
        self.highlighted = highlight(self.code, lexer, formatter)
        super(Snippet, self).save(*args, **kwargs)
		#在保存model的时候发送该signal
        signals.snippet_saved.send(sender=self.__class__, obj=self, instance=self)
```

##### 把signal和receiver连接起来

最后别忘了在最后加上一句:

```python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.db import models
from pygments.lexers import get_all_lexers
from pygments.lexers import get_lexer_by_name
from pygments.formatters.html import HtmlFormatter
from pygments import highlight
from pygments.styles import get_all_styles

import signals


LEXERS = [item for item in get_all_lexers() if item[1]]
LANGUAGE_CHOICES = sorted([item[1][0], item[0]] for item in LEXERS)
STYLE_CHOICES = sorted((item, item) for item in get_all_styles())


class Snippet(models.Model):

    created = models.DateTimeField(auto_now_add=True)
    title = models.CharField(max_length=100, blank=True, default='')
    code = models.TextField()
    linenos = models.BooleanField(default=False)
    language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
    style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)

    owner = models.ForeignKey('auth.user', related_name='snippet', on_delete=models.CASCADE)
    highlighted = models.TextField()

    class Meta:
        ordering = ('created', )

    def save(self, *args, **kwargs):
        """
        把snippet使用pygments来生成高亮的html代码
        :param args:
        :param kwargs:
        :return:
        """
        lexer = get_lexer_by_name(self.language)
        linenos = self.linenos and 'table' or False
        options = self.title and {'title': self.title} or {}
        formatter = HtmlFormatter(style=self.style, linenos=linenos,
                                  full=True, **options)
        self.highlighted = highlight(self.code, lexer, formatter)
        super(Snippet, self).save(*args, **kwargs)

        signals.snippet_saved.send(sender=self.__class__, obj=self, instance=self)


signals.snippet_saved.connect(receiver=signals.on_snippet_create, sender=Snippet)
```

当然除了使用connect函数，使用之前用到的receiver装饰器也是可以的。

完成之后，开启测试服务器，put一段新的snippet，同样的能在控制台查看到打印的log。



参考资料：

http://www.jianshu.com/p/7ad5db9a1b69

http://www.cnblogs.com/mindsbook/archive/2009/10/27/django_signal.html

http://python.usyiyi.cn/translate/django_182/topics/signals.html