---
title: SOCKET编程----Echo Something
categories:
  - Linux
tags:
  - tcp program
date: 2017-09-19 02:19:13
---

在网络世界中，socket通信几乎是当前几乎所有上层应用层协议的基础，例如用的较多的http协议，ftp协议，SMTP协议的背后都是socket在担负着通信过程。这篇博文首先用一个简单的TCP socket程序实例来引入Linux环境下的socket编程技术。不管原理懂不懂，先搞个程序跑起来先！

<!--more-->

- 程序目的

  服务端：接收来自客户端的内容，计算客户端发送的消息长度，然后将长度返回给客户端。

  客户端：从标准输入读取输入内容，并发送给服务端，接收来自服务端的响应，将响应内容输出至标准输出。

  服务端代码：

  ```c
  #include <stdio.h>
  #include <stdlib.h>
  #include <strings.h>
  #include <sys/types.h>
  #include <sys/socket.h>
  #include <arpa/inet.h>
  #include <unistd.h>

  #define PORT 5555
  #define BACKLOG 2
  void process_conn_server(int s){

  	ssize_t size = 0;
  	char buffer[1024];
  	
  	for(;;){
  		
  		size = read(s,buffer,1024);
  		printf("read %d\n",size);		
  		if(size ==0){
  			return;
  		}
  	sprintf(buffer,"%d bytes altother\n",size);
  	write(s,buffer,strlen(buffer)+1);
  	}
  }
  int main(int argc, char *argv[])
  {
  	int ss,sc;
  	struct sockaddr_in server_addr;
  	struct sockaddr_in client_addr;
  	int err;
  	pid_t pid;

  	ss = socket(AF_INET,SOCK_STREAM,0);
  	if(ss < 0){
  		printf("socket error.\n");
  		return -1;
  	}

  	bzero(&server_addr,sizeof(server_addr));
  	server_addr.sin_family = AF_INET;
  	server_addr.sin_addr.s_addr = htonl(INADDR_ANY);
  	server_addr.sin_port = htons(PORT);

  	err = bind(ss,(struct sockaddr*)&server_addr,sizeof(server_addr));
  	if(err < 0){
  		printf("bind err.\n");
  		return -1;
  	}

  	err = listen(ss,BACKLOG);
  	if(err < 0){
  		printf("listen err.\n");
  		return -1;
  	}

  	for(;;){

  		socklen_t addrlen = sizeof(struct sockaddr);
  		
  		sc = accept(ss,(struct sockaddr*)&client_addr,&addrlen);
  		if(sc < 0){
  			continue;
  		}else{
  			printf("some one conn~\n");
  		}

  		pid = fork();
  		if(pid < 0 ){
  			close(ss);
  		}else{
  			process_conn_server(sc);
  			close(sc);
  		}
  	}
  }

  ```

  客户端代码：

  ```c
  #include <stdio.h>
  #include <stdlib.h>
  #include <strings.h>
  #include <sys/types.h>
  #include <sys/socket.h>
  #include <unistd.h>
  #include <arpa/inet.h>

  #define PORT 5555 
  void process_conn_client(int s){

  	ssize_t size = 0;
  	char buffer[1024];

  	for(;;){
  		size = read(0,buffer,1024);
  		if(size > 0){
  			write(s,buffer,size);
  			size = read(s,buffer,1024);
  			write(1,buffer,size);
  		}
  	}

  }
  int main(int argc,char *argv[]){

  	int s;
  	struct sockaddr_in server_addr;
  	s = socket(AF_INET,SOCK_STREAM,0);
  	if(s < 0){
  		printf("socket error\n");
  		return -1;
  	}

  	bzero(&server_addr,sizeof(server_addr));
  	server_addr.sin_family = AF_INET;
  	server_addr.sin_addr.s_addr = htonl(INADDR_ANY);
  	server_addr.sin_port = htons(PORT);
  	inet_pton(AF_INET,argv[1],&server_addr.sin_addr);
  	connect(s,(struct sockaddr*)&server_addr,sizeof(struct sockaddr));
  	process_conn_client(s);
  	close(s);
  	return 0;
  }

  ```

  ​

运行实例：

```bash
model@PWN-VirtualBox:~/NetworkProgram$ ./server 
some one conn~
read 5
------------------------------------------------------------------
model@PWN-VirtualBox:~/NetworkProgram$ ./client 127.0.0.1
abcd
5 bytes altother
```

这是一个简单的tcp通信的程序示例，在这个程序中，一次只能服务两个客户端，不太具备实用价值。但是这个示例完整的包括了tcp socket通信的各个步骤，可以在此基础上继续丰富，完成更复杂的业务逻辑。在之后的博文中我将继续来解释socket编程的各个知识点。