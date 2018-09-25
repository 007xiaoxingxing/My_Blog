---
title: python中使用Unix socket通信
categories:
  - Program
tags:
  - python socket
date: 2017-10-19 11:12:15
---

对于socket模式的通信来说，除了常见的TCP/IP套接字还有Unix套接字。不过说Unix套接字是系统上的一个文件，而非主机名和端口，而Unix套接字关闭之后，文件系统上仍然会存在套接字文件，需要在服务端程序启动之前进行删除，它的其他操作和TCP/IP套接字是一样的。下面用一个简单的echo server记录一下Unix套接字的基本使用方法。

<!--more-->

server端代码：

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-


import socket
import sys
import os

server_addr = './uds.sock'
#确保sock不存在
try:
	os.unlink(server_addr)
except OSError:
	if os.path.exists(server_addr):
		raise
#创建一个unix类型的socket
sock = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
#将socket绑定到地址上
print >>sys.stderr, 'Starting up on %s', server_addr
sock.bind(server_addr)
#开始在绑定的地址上监听
sock.listen(5)
#死循环，阻塞式的接收数据
while True:
	print >>sys.stderr, 'Waiting for connetion...'
	conn, client_addr = sock.accept()
	try:
		print >>sys.stderr, 'connection from ', client_addr
		#使用较小的缓冲区接收数据，并返回它到客户端
		while True:
			data = conn.recv(6)
			print >>sys.stderr, 'recv data %s' % data
			if data:
				print >>sys.stderr, 'echo %s to client' % data
				conn.sendall(data)
			else:
				print >>sys.stderr, 'no data from client'
				break
	#记得关闭连接
	finally:
		conn.close() 
```

客户端代码：

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import socket
import sys

#创建一个unix socket
sock = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
#设置服务端socket的地址
server_addr = './uds.sock'
print >>sys.stderr, 'connecting to %s', server_addr
try:
	sock.connect(server_addr)
except socket.error, msg:
	print >>sys.stderr, msg
	sys.exit(1)
#向服务器发送数据和接收数据
try:
	#发送数据
	message = "This is the message. It will be repeated."
	print >>sys.stderr, 'sending %s' % message
	sock.sendall(message)

	amount_received = 0
	amount_expected = len(message)

	while amount_received < amount_expected:
		data = sock.recv(16)
		amount_received += len(data)
		print >>sys.stderr, 'received %s' % data
finally:
	print >>sys.stderr, 'closing socket'
	sock.close()
```

运行结果：

服务端：

```shell
xing@xing:~/pypractice$ python uds_server.py 
Starting up on %s ./uds.sock
Waiting for connetion...
connection from  
recv data This i
echo This i to client
recv data s the 
echo s the  to client
recv data messag
echo messag to client
recv data e. It 
echo e. It  to client
recv data will b
echo will b to client
recv data e repe
echo e repe to client
recv data ated.
echo ated. to client
recv data 
no data from client

```

客户端：

```shell
xing@xing:~/pypractice$ python uds_client.py 
connecting to %s ./uds.sock
sending This is the message. It will be repeated.
received This is the mess
received age. It will be 
received repeated.
closing socket
```

使用Unix套接字需要注意程序是否有权限去读写套接字文件。

Unix套接字最主要的一个用途就是用于进程间通信，为此还有本大部头的书籍《Unix网络编程卷2：进程间通信》，看来还得买来学习一下。

这里有个简单的进程间通信的小例子：

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import socket
import os

parent, child = socket.socketpair()

pid = os.fork()

if pid:
    print 'in parent, sending message'
    child.close()
    parent.sendall('ping')
    response = parent.recv(1024)
    print 'response from child:', response
    parent.close()

else:
    print 'in child, waiting for message'
    parent.close()
    message = child.recv(1024)
    print 'message from parent:', message
    child.sendall('pong')
    child.close()
```

运行结果：

```shell
xing@xing:~/pypractice$ python pro.py 
in parent, sending message
in child, waiting for message
message from parent: ping
response from child: pong
```

参考链接：https://pymotw.com/2/socket/uds.html