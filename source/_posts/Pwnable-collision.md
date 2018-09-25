---
title:  pwnable练习之collision 
date: 2016-11-28 19:25:18
categories: CTF
tags:
- CTF
- Linux
- PWN
- pwnable
---
col.c代码如下：
```c
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
	int* ip = (int*)p;
	int i;
	int res=0;
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}

int main(int argc, char* argv[]){
	if(argc<2){
		printf("usage : %s [passcode]\n", argv[0]);
		return 0;
	}
	if(strlen(argv[1]) != 20){
		printf("passcode length should be 20 bytes\n");
		return 0;
	}

	if(hashcode == check_password( argv[1] )){
		system("/bin/cat flag");
		return 0;
	}
	else
		printf("wrong passcode.\n");
	return 0;
}

```
观察代码，可知代码逻辑过程是：  
1.先输入一个不少于20字节长度的值  
2.对输入的前五位整数求和(int类型，一个int等于4字节，5位int类型，恰好20字节)  
3.求得的和与0x21DD09EC比较，若相等则通过  
<!-- more -->
重点1：
```c
int* ip = (int*)p;
```
这里将输入的char类型的指针强制转换为了int指针，从而ip指针一次能够指向4个字节大小的内存区域。画个图模拟一下内存布局：  
![image](/blogimg/collision.png)

思路1：将0x21DD09EC除以5变为4字节为一组，分别输入，最后以4字节为一组进行求和即可。  
比如：
```python
>>> 0x21DD09EC/5
113626824
>>> hex(0x21DD09EC/5)
'0x6c5cec8'
```
哦，稍等。好像不能被5整除哦。换个方式：4 x 0x01010101 + 0x1dd905e8
```python
>>> hex(0x21DD09EC-4*0x01010101)
'0x1dd905e8'
>>>
```
接下我们把刚刚的值输入：
```bash
col@ubuntu:~$ ./col `python -c "print 16*'\x01'+'\xe8\x05\xd9\x1d'"`
daddy! I just managed to create a hash collision :)
col@ubuntu:~$ 

```
YES,GET FLAG!


