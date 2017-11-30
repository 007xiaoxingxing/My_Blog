---
title: Python学习笔记之------atexit模块
categories:
  - Program
tags:
  - python
date: 2017-11-30 14:57:00
---

atexit模块用于注册python解释器退出时要执行的函数，这个模块只提供一个函数：
register（func, [,args [,kwargs]]）
解释器退出的时候会按照注册的顺序逆序调用这些函数，如果出现了错误，将一条异常信息打印到标准错误，然后继续执行下一个回调，直到所有的回调都执行完毕后，重新抛出最后接收到的异常。register支持装饰器语法。
<!--more-->
这里给出一个例子程序：
```python
# -*- coding: utf-8 -*-

import atexit

def exit0(*args, **kwargs):
    print 'exit0'
    for arg in args:
        print ' ' * 4, arg

    for item in kwargs.items():
        print ' ' * 4, item

def exit1():
    print 'exit1'
    raise Exception, 'exit1'

def exit2():
    print 'exit2'

atexit.register(exit0, *[1, 2, 3], **{"a":1, "b":2})
atexit.register(exit1)
atexit.register(exit2)

@atexit.register
def exit3():
    print 'exit3'

if __name__ == '__main__':
    pass
```
运行结果：
```shell
❯ python atexit_example.py 
exit3
exit2
exit1
Error in atexit._run_exitfuncs:
Traceback (most recent call last):
  File "/usr/lib/python2.7/atexit.py", line 24, in _run_exitfuncs
    func(*targs, **kargs)
  File "atexit_example.py", line 15, in exit1
    raise Exception, 'exit1'
Exception: exit1
exit0
     1
     2
     3
     ('a', 1)
     ('b', 2)
Error in sys.exitfunc:
Traceback (most recent call last):
  File "/usr/lib/python2.7/atexit.py", line 24, in _run_exitfuncs
    func(*targs, **kargs)
  File "atexit_example.py", line 15, in exit1
    raise Exception, 'exit1'
Exception: exit1

```
参考链接：
https://docs.python.org/2/library/atexit.html
http://python.jobbole.com/81473/