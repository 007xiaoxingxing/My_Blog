---
title: Python学习笔记之-----闭包
categories:
  - Program
tags:
  - python
date: 2017-11-23 14:50:44
---
在函数式编程中，经常听到闭包。那么闭包到底是什么呢？使用闭包有什么好处？在python中又应该怎么使用呢？这里用几个小例子记录一下我了解到的python闭包。
<!--more-->
#### 什么是闭包
闭包（Closure）是词法闭包（Lexical Closure）的简称，是引用了自由变量的函数。这个被引用的自由变量将和这个函数一同存在，即使已经离开了创始它的环境也不例外。所以，闭包是函数和与其相关的引用环境组合而形成的实体。
#### 一个闭包的例子

```python
from urllib import urlopen
def page(url):
  def get():
    return urlopen(url).read()
  return get

baidu_page = page("http://www.baidu.com")
print baidu_page
print baidu_page()
```
在这个例子中，在page函数中，其内部的get函数作为返回值给返回了，在执行get()函数的时候，它会使用原来提供给page（）函数的url去调用urlopen（url）。
如果想查看闭包中的变量内容，可以这样：
```shell
>>>baidu_page.__closure__  #可以看到其closure属性是一个元组
>>>baidu_page.__closure__[0].cell_contents #查看元组的内容，即传入的url
```
#### 使用闭包需要注意的地方
闭包中是不能修改外部作用域的局部变量的
```python
def foo():
  m = 0
  def foo1():
    m = 1
    print m
  print m
  foo1()
  print m

>>>foo()
```
错误的使用局部变量
```python
def foo():
  a = 1
  def bar():
    a = a + 1
    return a
  return bar
>>>c = foo()
>>>print c()
```
这样在调用的时候会产生错误，因为在bar内部，a在=号之前，被认为是bar的局部变量，等待赋值，而稍后去取a的值，而此时a是未被初始化的，所以会报一个a未定义的错误。解决这个问题的话，可以将a定义为一个可变变量，例如list

#### 闭包的应用例子
在有的函数中，如果想让函数调用保持某个状态，那么使用闭包是一种非常有效的方式。
另外一个就是常用的装饰器了。
闭包有效的减少了函数所需定义的参数数目，便于用于并行计算。

#### 参考链接
http://python.jobbole.com/82624/
https://www.cnblogs.com/ma6174/archive/2013/04/15/3022548.html
https://www.cnblogs.com/ChrisChen3121/p/3208119.html
http://python.jobbole.com/82296/
《python参考手册》
