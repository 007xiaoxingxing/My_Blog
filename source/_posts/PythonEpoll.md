---
title: 在python中使用epoll实现非阻塞io
categories:
  - Program
tags:
  - python
date: 2017-10-19 15:08:41
---

#### 0x1 啥是epoll？

epoll - I/O event notification facility---事件驱动的io。

​	在传统的网络编程中，listen，send，recv函数都是阻塞的。在一个过程中你只能乖乖的等着数据的到来才能进行下一步操作，这就非常影响效率了。为了解决阻塞的问题，先后有了select和poll方式来轮询io事件，这两种方式虽然已经提高了效率，但是他们是无差别轮询，还是浪费了一点时间。所以有了epoll，它只将发生变化的io事件通知我们，大大提高了性能，广泛应用于高并发的请求的程序中。

<!--more-->

#### 0x2 在python中咋用

​	python中的select模块提供epoll的操作接口，那么先介绍一下使用epoll的基本步骤：

​		1.创建一个epoll对象

​		2.将需要监听的socket注册到epoll对象上

​		3.询问epoll对象，在某个时间段内是否发生了指定的io事件

​		4.得到发生了io事件的socket，对这些socket进行读写或者其他操作

​		5.告诉epoll对象，是否需要修改socket对象的状态或事件，继续进行监控

​		6.重复3-5，直到完成

​		7.销毁epoll对象

​	epoll对象的一些方法：

```python
import select 导入select模块

epoll = select.epoll() 创建一个epoll对象

epoll.register(文件句柄,事件类型) 注册要监控的文件句柄和事件

事件类型:

　　select.EPOLLIN    可读事件

　　select.EPOLLOUT   可写事件

　　select.EPOLLERR   错误事件

　　select.EPOLLHUP   客户端断开事件

epoll.unregister(文件句柄)   销毁文件句柄

epoll.poll(timeout)  当文件句柄发生变化，则会以列表的形式主动报告给用户进程,timeout

                     为超时时间，默认为-1，即一直等待直到文件句柄发生变化，如果指定为1

                     那么epoll每1秒汇报一次当前文件句柄的变化情况，如果无变化则返回空

epoll.fileno() 返回epoll的控制文件描述符(Return the epoll control file descriptor)

epoll.modfiy(fineno,event) fineno为文件描述符 event为事件类型  作用是修改文件描述符所对应的事件

epoll.fromfd(fileno) 从1个指定的文件描述符创建1个epoll对象

epoll.close()   关闭epoll对象的控制文件描述符
```

​	epoll的一个基于状态触发（level-triggered）的例子：

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-


import socket
import select


EOL1 = b'\n\n'
EOL2 = b'\n\r\n'
response = b'HTTP/1.0 200 OK\r\nDate: Mon, 1 Jan 2017 10:10:10 GMT\r\n'
response += b'Content-Type: text/plain\r\nContent-Length: 13\r\n\r\n'
response += b'Hello, World!'

#创建一个tcp套接字
serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
#设置端口复用
serversocket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
serversocket.bind(('0.0.0.0', 8080))
serversocket.listen(5)
serversocket.setblocking(0)
#创建epoll对象
epoll = select.epoll()
#将服务器的套接注册到epoll对象中，由epoll对象监听有关事件
epoll.register(serversocket.fileno(), select.EPOLLIN)


try:
	connections = {}
	requests = {}
	responses = {}
	while True:
        #从epoll对象中查询近1s时间内的I/O事件
		events = epoll.poll(1)
		for fileno, event in events:
            #如果发生事件的是服务器套接字，表示有客户端连接
			if fileno == serversocket.fileno():
				connection, address = serversocket.accept()
                #将接收到的套接字设置为非阻塞
				connection.setblocking(0)
                #向epoll对象中注册客户端连接，监听客户端是否发送数据
				epoll.register(connection.fileno(), select.EPOLLIN)
				connections[connection.fileno()] = connection
				requests[connection.fileno()] = b''
				responses[connection.fileno()] = response
                #epoll对象监听到有客户端发送数据的事件
			elif event & select.EPOLLIN:
                #尝试接收客户端的数据
				requests[fileno] += connections[fileno].recv(1024)
				if EOL1 in requests[fileno] or EOL2 in requests[fileno]:
					epoll.modify(fileno, select.EPOLLOUT)
					print '-'*40 + '\n' + requests[fileno].decode()[:-2]
            #epoll对象监听到有准备就绪的可发送数据的连接
			elif event & select.EPOLLOUT:
                #向客户端发送数据
				byteswritten = connections[fileno].send(responses[fileno])
				responses[fileno] = responses[fileno][byteswritten:]
				if len(responses[fileno]) == 0:
					epoll.modify(fileno, 0)
					connections[fileno].shutdown(socket.SHUT_RDWR)
            #epoll监听到终止连接的事件
			elif event & select.EPOLLHUP:
				epoll.unregister(fileno)
				connections[fileno].close()
				del connections[fileno]
	

finally:
	epoll.unregister(serversocket.fileno())
	epoll.close()
	serversocket.close()
```

运行结果：

```shell
//服务端
xing@xing:~/pypractice$ python epoll_demo1.py 
----------------------------------------
GET / HTTP/1.1
Host: 10.70.27.36:8080
Connection: keep-alive
Cache-Control: max-age=0
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.62 Safari/537.36
Upgrade-Insecure-Requests: 1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9

//客户端：
PS C:\Users\STAR_CHEN> http 10.70.27.36:8080
HTTP/1.0 200 OK
Content-Length: 13
Content-Type: text/plain
Date: Mon, 1 Jan 2017 10:10:10 GMT

Hello, World!
```

epoll基于边沿触发（edge-triggered）的一个例子：

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-


import socket
import select


EOL1 = b'\n\n'
EOL2 = b'\n\r\n'
response = b'HTTP/1.0 200 OK\r\nDate: Mon, 1 Jan 2017 10:10:10 GMT\r\n'
response += b'Content-Type: text/plain\r\nContent-Length: 13\r\n\r\n'
response += b'Hello, World!'


serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
serversocket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
serversocket.bind(('0.0.0.0', 8080))
serversocket.listen(5)
serversocket.setblocking(0)

epoll = select.epoll()
#同时注册可读和可写的掩码
epoll.register(serversocket.fileno(), select.EPOLLIN | select.EPOLLOUT)


try:
	connections = {}
	requests = {}
	responses = {}
	while True:
		events = epoll.poll(1)
		for fileno, event in events:
			if fileno == serversocket.fileno():
				try:
                  #自己进行循环，处理io事件
					while True:
						connection, address = serversocket.accept()
						connection.setblocking(0)
                        #同时注册可读和可写的掩码
						epoll.register(connection.fileno(), select.EPOLLIN | select.EPOLLET)
						connections[connection.fileno()] = connection
						requests[connection.fileno()] = b''
						responses[connection.fileno()] = response
				except socket.error:
					pass
			elif event & select.EPOLLIN:
				try:
					while True:
						requests[fileno] += connections[fileno].recv(1024)
				except socket.error:
					pass
				if EOL1 in requests[fileno] or EOL2 in requests[fileno]:
					epoll.modify(fileno, select.EPOLLOUT | select.EPOLLET)
					print '-'*40 + '\n' + requests[fileno].decode()[:-2]
			elif event & select.EPOLLOUT:
				try:
					while len(response[fileno]) > 0:
						byteswritten = connections[fileno].send(responses[fileno])
						responses[fileno] = responses[fileno][byteswritten:]
				except socket.error:
					pass
				if len(responses[fileno]) == 0:
					epoll.modify(fileno, 0)
					connections[fileno].shutdown(socket.SHUT_RDWR)
			elif event & select.EPOLLHUP:
				epoll.unregister(fileno)
				connections[fileno].close()
				del connections[fileno]
	

finally:
	epoll.unregister(serversocket.fileno())
	epoll.close()
	serversocket.close()
```

#### 0x3 总结

采用边沿触发的特点：

​	当被监控的文件描述符上有可读写事件发生时，epoll_wait()会通知处理程序去读写。如果这次没有把数据全部读写完(如读写缓冲区太小)，那么下次调用epoll_wait()时，它不会通知你，也就是它只会通知你一次，直到该文件描述符上出现第二次可读写事件才会通知你！！！这种模式比水平触发效率高，系统不会充斥大量你不关心的就绪文件描述符！！！

​	使用边沿触发需要注意，将连接设置为非阻塞，并且一次性将数据读取完。

水平触发的特点：

​	当被监控的文件描述符上有可读写事件发生时，epoll_wait()会通知处理程序去读写。如果这次没有把数据一次性全部读写完(如读写缓冲区太小)，那么下次调用 epoll_wait()时，它还会通知你在上没读写完的文件描述符上继续读写，当然如果你一直不去读写，它会一直通知你！！！如果系统中有大量你不需要读写的就绪文件描述符，而它们每次都会返回，这样会大大降低处理程序检索自己关心的就绪文件描述符的效率！！！



参考链接：

http://www.cnblogs.com/yuuyuu/p/5103744.html

 http://blog.csdn.net/hehe123456zxc/article/details/52526670

 http://blog.csdn.net/voidccc/article/details/8619632

 http://blog.csdn.net/q576709166/article/details/8649911  EPOLL中有关事件意义

