---
title: python学习笔记之------元类
categories:
  - Program
tags:
  - python
date: 2017-12-04 16:34:30
---

"元类（Meta Class）",即用于创建类的类。那我们先来看看python中的类是个啥东西。

<!--more-->

```python
>>> class this_is_a_class(object):
...     pass
...
>>> a_object = this_is_a_class()
>>> print a_object
<__main__.this_is_a_class object at 0x105a11a90>
>>> type(a_object)
<class '__main__.this_is_a_class'>
>>> type(this_is_a_class)
<type 'type'>
>>> print this_is_a_class
<class '__main__.this_is_a_class'>
>>>
```

可以看出来，类其实也是个对象。那么这个对象能不能动态的改变呢？我们来试试看

```python
>>> this_is_a_class.func = func
>>> this_is_a_class.attr = 'attr'
>>> object_b = this_is_a_class()
>>> object_b.attr
'attr'
>>> object_b.func()
func
>>>
```

哇，是不是很神奇？我们直接给类赋值属性和方法，然后用这个类得到的实例也都具有了这些属性和方法。那么更近一步，我们可不可以自由的生成一个类呢？答案的肯定的，那么该如何创建一个类呢？看代码：

