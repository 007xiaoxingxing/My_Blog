---
title: 与Tornado的第一次接触
date: 2016-12-15 19:59:25
categories:
- Program
tags:
- python
- web
- tornado
---
Tornado框架是一款由Facebook团队开源的高效率的轻量级的非阻塞式web框架，由python编写而成。框架内部的具体实现我暂时不去深究，咱们首选学习一下它的简单使用，爽一爽先。
<!-- more -->
### 0x1:tornado的安装
安装Tornado也简直不能再省事儿，一句话就可以搞定：
```bash
star-chen@starchen-vb:~$ pip install tornado
Collecting tornado
  Downloading tornado-4.4.2.tar.gz (460kB)
    100% |████████████████████████████████| 460kB 21kB/s 
Collecting singledispatch (from tornado)
  Downloading singledispatch-3.4.0.3-py2.py3-none-any.whl
Collecting certifi (from tornado)
  Downloading certifi-2016.9.26-py2.py3-none-any.whl (377kB)
    100% |████████████████████████████████| 378kB 37kB/s 
Collecting backports_abc>=0.4 (from tornado)
  Downloading backports_abc-0.5-py2.py3-none-any.whl
Collecting six (from singledispatch->tornado)
  Downloading six-1.10.0-py2.py3-none-any.whl
Building wheels for collected packages: tornado
  Running setup.py bdist_wheel for tornado ... done
  Stored in directory: /home/star-chen/.cache/pip/wheels/b3/db/47/46e05d1ee3ecfba252fcab42f0a156dab0df0cddf99fa0827c
Successfully built tornado
Installing collected packages: six, singledispatch, certifi, backports-abc, tornado
Successfully installed backports-abc-0.5 certifi-2016.9.26 singledispatch-3.4.0.
```
### 0x2:tornado的helloworld 
话不多讲，直接上代码
```python
import tornado.ioloop
import tornado.web

class MainHandler(tornado.web.RequestHandler):
	def get(self):
		self.write("hello world");

def HelloApp():
	return tornado.web.Application([
		(r"/",MainHandler),
	])

if __name__ == "__main__":
	app = HelloApp()
	app.listen(8888)
	tornado.ioloop.IOLoop.current().start()
```
看起来还是十分简洁明了的，就这么几句代码，就包含了路由规则的创建、事件处理函数的绑定、服务器的监听开启。
### 0x3:代码运行
```bash
$python hello.py
```
```bash
$curl localhost:8888
hello world
```
用浏览器访问一下，可以看到浏览器上已经输出了“hello world”,说明咱们的tornado已经正常运行了。接下来继续学习怎么在tornado中取得用户提交的参数。
