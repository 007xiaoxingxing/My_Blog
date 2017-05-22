---
title: 修改OpenSSH源码，记录爆破SSH端口的密码
categories:
  - Linux
tags:
  - ssh
  - 暴力破解
  - 记录  
date: 2017-05-22 14:56:04
---

### 起因

自从用上了VPS,爆破ssh端口的小黑们就一刻也没挺过，虽然我仍然使用的密码登录，自认为自己设置的密码还是足够的强健，但是还是想记录一下他们爆破所用的字典。搜索了几种方式，例如可以使用蜜罐，更改openssh源码等方式来记录爆破字典。为了省事，我采用了修改源码的方式来进行记录。

<!-- more -->

### 步骤1  搭建编译环境

我所使用的环境是CentOS 7,需要安装openssl-devel和pam-devel

```bash
root@vultr:yum install openssl-devel pam-devel
```

卸载原有系统自带的openssh-server

```bash
#查看系统中安装的所有openssh有关的软件包
[root@vultr]# rpm -qa | grep openssh
openssh-clients-6.6.1p1-33.el7_3.x86_64
openssh-6.6.1p1-33.el7_3.x86_64
openssh-server-6.6.1p1-33.el7_3.x86_64
#卸载openssh和openssh-server
[root@vultr openssh-6.6p1]# yum remove openssh-server openssh
#删除原有的ssh配置文件
[root@vultr openssh-6.6p1]# rm -f /etc/ssh/*
```

OK,执行上以上操作后，准备工作就算完成了。

### 步骤2 下载openssh的源码，并解压

```bash
root@vultr#wget https://mirrors.evowise.com/pub/OpenBSD/OpenSSH/portable/openssh-6.6p1.tar.gz
root@vultr#tar -zxvf openssh-6.6p1.tar.gz
```

### 步骤3 修改源码，使之能记录登录密码

需要修改的源码文件为  
> auth2-passwd.c
> 修改后的源码内容如下：
```c
#include "hostfile.h"
#include "auth.h"
#include "buffer.h"
#ifdef GSSAPI
#include "ssh-gss.h"
#endif
#include "monitor_wrap.h"
#include "servconf.h"

/* import */
extern ServerOptions options;

static int
userauth_passwd(Authctxt *authctxt)
{



	char *password, *newpass;
	int authenticated = 0;
	int change;
	u_int len, newlen;

	change = packet_get_char();
	password = packet_get_string(&len);
  
	//加上这行，就可以将登录用户名，密码添加到日志中
	logit("user=%s,pass=%s",authctxt->user,password);
	
	if (change) {
		/* discard new password from packet */
		newpass = packet_get_string(&newlen);
		explicit_bzero(newpass, newlen);
		free(newpass);
	}
	packet_check_eom();

	if (change)
		logit("password change not supported");
	else if (PRIVSEP(auth_password(authctxt, password)) == 1)
		authenticated = 1;
	explicit_bzero(password, len);
	free(password);
	return authenticated;
}

Authmethod method_passwd = {
	"password",
	userauth_passwd,
	&options.password_authentication
};
```



### 步骤3 设置编译参数，并进行编译安装

```bash
#运行configure,并生成makefile
[root@vultr openssh-6.6p1]#./configure --prefix=/usr --sysconfdir=/etc/ssh --with-pam --with-md5-passwords
#编译并安装
[root@vultr openssh-6.6p1]# make && make install

```

### 步骤4 添加服务，设置开机启动

```bash
#将启动脚本复制到/etc/init.d
[root@vultr openssh-6.6p1]# cp contrib/redhat/sshd.init /etc/init.d/sshd
#使ssh开机启动
[root@vultr openssh-6.6p1]# systemctl enable sshd
sshd.service is not a native service, redirecting to /sbin/chkconfig.
Executing /sbin/chkconfig sshd on

```

之后把服务器重启一下，记录到的日志内容在：

> /var/log/messages

中。把日志处理一下，就可以搜集一波黑客的爆破字典拉。我们来看看效果如何

![image](/blogimg/openssh-code.png)

可以从日志中已经有黑客的爆破密码记录了。嗯，自己送上门来的字典，为什么不收着呢？