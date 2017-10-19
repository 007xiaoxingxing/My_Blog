---
title: Python中实现单例模式
categories:
  - Program
tags:
  - python
date: 2017-10-19 16:33:44
---

#### 0x1 何谓单例模式？

单例模式简单来讲就是一个类只能存在一个对象实例。

那么在python中如何实现单例模式呢？

<!--more-->

#### 0x2 在python中实现单例模式

1.使用装饰器

```python
# -*- coding:utf-8 -*-


def singleton(cls, *args, **kwargs):
    instances = {}
    def _singleton():
        if cls  not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    return _singleton

@singleton
class MyClass(object):
    a = 1
    def __init__(self, x=0):
        self.x = x

c1 = MyClass()
c2 = MyClass()
print id(c1)
print id(c2)

#运行结果
>>> 
============= RESTART: E:\Study\PythonPractice\Singleton_demo.py =============
57740032
57740032
```

2.使用\_\_metaclass\_\_ 来实现

```python
class Singleton2(type):
    _instance = {}
    def __init__(cls, name, bases, dict):
        super(Singleton2, cls).__init__(name, bases, dict)
        cls._instance = None
    def __call__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super(Singleton2, cls).__call__(*args, **kwargs)

class MyClass2(object):
    __metaclass__ = Singleton2


c3 = MyClass2()
c4 = MyClass2()



print id(c3)
print id(c4)
```

以上就是python中实现单例模式的两种常用方法。

另外还有一种方式：

```python
class Singleton(object):
    def __new__(cls, *args, **kwargs):
        if not hasattr(cls, '_instance'):
            cls._instance = super(Singleton, cls).__new__(cls, *args, **kwargs)
        return cls._instance
 
class Foo(Singleton): #单例类
    a = 1
```

参考链接:

http://python.jobbole.com/87514/?repeat=w3tc

http://blog.csdn.net/karldoenitz/article/details/23467221

http://www.cnblogs.com/kakaliush/p/5228165.html

