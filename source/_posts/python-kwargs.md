---
title: python-**kwargs的用法
categories:
  - Program
tags:
  - python
date: 2017-10-12 12:55:27
---

### Python中的**kwargs的用法

**kwargs 允许你将不定长度的键值对作为参数传递给一个函数。如果你想要在一个函数里处理带名字的参数，使用\*\*kwargs是一个好选择。

<!--more-->

来看个例子感受一下：

```python
# -*- coding:utf-8 -*-
def foo(*args, **kwargs):
    print 'args = ', args
    print 'kwargs = ', kwargs
    print '---------------------------------------'

if __name__ == '__main__':
    foo(1,2,3,4)
    foo(a=1,b=2,c=3)
    foo(1,2,3,4, a=1,b=2,c=3)
    foo('a', 1, None, a=1, b='2', c=3)
```

运行结果：

```python
>>> 
====== RESTART: C:\Users\STAR_CHEN\Documents\工作记录\2017.10.12\kw.py ======
args =  (1, 2, 3, 4)
kwargs =  {}
---------------------------------------
args =  ()
kwargs =  {'a': 1, 'c': 3, 'b': 2}
---------------------------------------
args =  (1, 2, 3, 4)
kwargs =  {'a': 1, 'c': 3, 'b': 2}
---------------------------------------
args =  ('a', 1, None)
kwargs =  {'a': 1, 'c': 3, 'b': '2'}
---------------------------------------
```

\*args带入的参数是一个tuple类型，而\*\*kwargs带入的参数是一个dict类型。



再来看个例子：

```python
def test_args_kwargs(arg1, arg2, arg3):
    print("arg1:", arg1)
    print("arg2:", arg2)
    print("arg3:", arg3)
```

测试结果：

```python
# 首先使用 *args
>>> args = ("two", 3, 5)
>>> test_args_kwargs(*args)
arg1: two
arg2: 3
arg3: 5

# 现在使用 **kwargs:
>>> kwargs = {"arg3": 3, "arg2": "two", "arg1": 5}
>>> test_args_kwargs(**kwargs)
arg1: 5
arg2: two
arg3: 3
```

用kwargs来传递字典类型的参数，还蛮方便的呢。