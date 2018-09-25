---
title:  pwnable练习之fd 
date: 2016-11-27 19:25:18
categories: CTF
tags:
- CTF
- Linux
- PWN
- pwnable
---
fd.c的代码如下：
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
	if(argc<2){
		printf("pass argv[1] a number\n");
		return 0;
	}
	int fd = atoi( argv[1] ) - 0x1234;
	int len = 0;
	len = read(fd, buf, 32);
	if(!strcmp("LETMEWIN\n", buf)){
		printf("good job :)\n");
		system("/bin/cat flag");
		exit(0);
	}
	printf("learn about Linux file IO\n");
	return 0;

}

```
代码逻辑比较清楚，需要输入一个数字，然后减去0x1234，所得的值作为read函数的第一个参数，然后调用read函数读取输入，然后将读取的值和“LETMEWIN\n”比较，若相等则cat输出flag文件中的值。  
接下来我们看一下read函数的函数描述:
<!-- more -->
```c
NAME
       read - read from a file descriptor

SYNOPSIS
       #include <unistd.h>

       ssize_t read(int fd, void *buf, size_t count);

```
可以看到第一个参数为文件描述符。在Linux系统中存在三个比较特殊的文件描述符：  

| 文件描述符  |  用途  | POISX名称   |  stdio流  |
| ------------|-------:| -----------:| ---------:|
|0            |标准输入|STDIN_FILENO |  stdin    |
|1            |标准输出|STDOUT_FILENO|  stdout   |
|2            |标准错误|STDERR_FILENO|  stderr   |
这里我们可以让read参数的第一个参数为0（从标准输入读取），然后输入“LETMEWIN”即可让程序执行cat flag操作。
```
fd@ubuntu:~$ ./fd 4660
LETMEWIN
good job :)
mommy! I think I know what a file descriptor is!!

```
注：fd,file decriptor 文件描述符

