---
title: Python学习笔记之----repr()函数
categories:
  - Program
tags:
  - python note
date: 2017-09-28 19:41:10
---

在python中，可以通过str()方法和repr()方法将一个对象转换为字符串。但是这两个方法也有少许的不同，python官方手册上对这两个方法的描述分别如下：

<!--more -->

1. repr()

> repr(object) 
>
> Return a string containing a printable representation of an object. This is the same value yielded by conversions (reverse quotes). It is sometimes useful to be able to access this operation as an ordinary function. For many types, this function makes an attempt to return a string that would yield an object with the same value when passed to [`eval()`](#eval), otherwise the representation is a string enclosed in angle brackets that contains the name of the type of the object together with additional information often including the name and address of the object. A class can control what this function returns for its instances by defining a [`__repr__()`](../reference/datamodel.html#object.__repr__) method.

2.str()

> class  str (object='') 
>
>   Return a string containing a nicely printable representation of an object. For strings, this returns the string itself. The difference with `repr(object)` is that `str(object)` does not always attempt to return a string that is acceptable to [`eval()`](#eval); its goal is to return a printable string. If no argument is given, returns the empty string, `''`.For more information on strings see [Sequence Types — str, unicode, list, tuple, bytearray, buffer, xrange](stdtypes.html#typesseq) which describes sequence functionality (strings are sequences), and also the string-specific methods described in the [String Methods](stdtypes.html#string-methods) section. To output formatted strings use template strings or the `%` operator described in the [String Formatting Operations](stdtypes.html#string-formatting) section. In addition see the [String Services](strings.html#stringservices) section. See also [`unicode()`](#unicode).

简单来说就是str()得到的字符串对象更方便用于人类识别和打印，repr()方法得到的字符串对象更容易被python解释器识别，而且使用repr()方法，这个等式通常是成立的obj == eval(repr(obj))

```python

>>> obj = "hello python"
>>> obj == eval(repr(obj))
True
#而str()方法则不具备这个功能
>>> obj == eval(str(obj))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<string>", line 1
    hello python
               ^
SyntaxError: unexpected EOF while parsing
```

