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

```python
>>> class_name = "CreateTest"
>>> class_parents = (object,)
>>> class_body="""
... def __init__(self, x):
...     self.x = x
... def say_hello(self):
...     print "hello,guys!"
... """
>>> class_dict = {}
>>> exec(class_body, globals(), class_dict)
>>> CreateTest = type(class_name, class_parents, class_dict)
>>> object_a = CreateTest(4)
>>> object_a.x
4
>>> object_a.say_hello()
hello,guys!
```

是不是感觉很奇怪，我们使用type居然自己创造了一个类，而且还给这个类赋予了属性和方法。那很显然，type就是python中掌管类的创建的类，它也就是个元类。

那么问题来了，我们可不可以创建自己的元类呢？答案也是肯定的，还是来看代码：

```python
class MetaClass(type):
  def __new__(self, name, bases, attr):
      attrs = ((name, value) for name, value in attr.items() if not name.startswith('__'))
      uppercase_attr = dict((name.upper(), value) for name, value in attrs)
      def say_hello(cls):
        print 'hello guys!'
      t = type.__new__(self, name, bases, uppercase_attr)
      t.say_hello = say_hello
      return t
  
class MetaTest(object):
  __metaclass__ = MetaClass
  bar = 'attrs'
  
object_1 = MetaTest()
object_1.say_hello()
print object_1.BAR
print object_1.bar
```

运行测试结果：

```shell
hello guys!
attrs

Traceback (most recent call last):
  File "E:\Study\PythonPractice\meta_two.py", line 18, in <module>
    print object_1.bar
AttributeError: 'MetaTest' object has no attribute 'bar'
```

通过这个例子可以看出，我们可以使用元类对类的创建进行类似hook的操作，为新创建的类添加新的属性或者更改它现有的属性，这使得我们站在了上帝的角度可以管理类的创建行为。

注意事项：

\__metaclass\__的查找顺序是首先在当前文件查找全局变量\__metaclass__，如果没找到会在父类寻找该属性父类都找不到的话，会从模块的层次去查找，还找不到的话就是用默认的type作为元类来创建对象。





部分参考资料：

《python参考手册》

http://blog.jobbole.com/21351/

https://stackoverflow.com/questions/100003/what-is-a-metaclass-in-python