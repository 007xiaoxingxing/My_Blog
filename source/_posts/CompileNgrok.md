---
title: 在自己的VPS上编译部署Ngrok
categories:
  - Linux
tags:
  - 转发
date: 2017-06-02 11:12:12
---

由于我有一些测试服务需要从内网映射到公网，但是又不想使用其他第三方的转发服务，索性就自己编译一把Ngrok放到自己的服务器上用吧。

<!-- more -->

### Ngrok是个啥？

> ngrok is a reverse proxy that creates a secure tunnel from a public endpoint to a locally running web service. ngrok captures and analyzes all traffic over the tunnel for later inspection and replay.
>
> -------github上作者对ngrok的说明

简单的来说，ngrok就是一个反向代理，可以通过公网服务器来和本地内网机器之间建立一个隧道，使得可以从外网访问内网的服务。

### 准备工作

1. 在服务器上搭建go环境

我选择直接下载go的源码

>  [http://www.golangtc.com/download](http://www.golangtc.com/download) 

选取适合服务器系统的go环境压缩包，由于我的VPS是Centos 7 64位，所以我下载的是1.8 amd64

```bash
[root@vultr ngrok_compile]# wget http://www.golangtc.com/static/go/1.8/go1.8.linux-amd64.tar.gz
```

然后需要解压一下，需要注意的是要把go解压到/usr/local，免得后面出现一些不可描述的问题。

```bash
[root@vultr ngrok_compile]# tar -C /usr/local  -xzf go1.8.linux-amd64.tar.gz
```

为go创建个软连接到/usr/bin，方便直接使用go命令

```bash
[root@vultr ngrok_compile]# ln -s /usr/local/go/bin/* /usr/bin
```

到此，go环境就搭建好了。

2. 克隆ngrok的git仓库到本地

```bash
[root@vultr ngrok_compile]# git clone https://github.com/inconshreveable/ngrok.git
```

3. 设置一些编译所需的环境变量

```bash
[root@vultr ngrok]# export GOPATH=/opt/ngrok_compile/ngrok/
[root@vultr ngrok]# export NGROK_DOMAIN="ngrok.star-chen.com"
```

4. 为域名生成证书

```bash
[root@vultr ngrok_compile]# cd ngrok
[root@vultr ngrok]# openssl genrsa -out rootCA.key 2048
[root@vultr ngrok]# openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=$NGROK_DOMAIN" days 5000 -out rootCA.pem
[root@vultr ngrok]# openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=$NGROK_DOMAIN" -days 5000 -out rootCA.pem
[root@vultr ngrok]# openssl -genrsa -out sever.key 2048
[root@vultr ngrok]# openssl genrsa -out sever.key 2048
[root@vultr ngrok]# openssl req -new -key server.key -subj "/CN=$NGROK_DOMAIN" -out server.csr
[root@vultr ngrok]# openssl genrsa -out server.key 2048
[root@vultr ngrok]# openssl req -new -key server.key -subj "/CN=$NGROK_DOMAIN" -out server.csr
[root@vultr ngrok]# openssl x509 -req -in server.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out server.crt -days 5000
```

5. 把生成的证书拷贝到指定的目录下

```bash
[root@vultr ngrok]# cp rootCA.pem assets/client/tls/ngrokroot.crt 
[root@vultr ngrok]# cp server.crt assets/server/tls/snakeoil.crt 
[root@vultr ngrok]# cp server.key assets/server/tls/snakeoil.key 
```

6.为域名设置A记录泛解析

![img](/blogimg/fan.png)

### 开始编译

```bash
#先编译个Linux 64位的服务端和客户端程序
[root@vultr ngrok]#  GOOS=linux GOARCH=amd64 make release-server
[root@vultr ngrok]#  GOOS=linux GOARCH=amd64 make release-client
#再编译个windows64位的客户端
[root@vultr ngrok]#  GOOS=windows GOARCH=amd64 make release-client
```

看一下编译好的程序

```bash
[root@vultr ngrok]# ls -la bin
total 21544
drwxr-xr-x 3 root root     4096 Jun  2 03:06 .
drwxr-xr-x 9 root root     4096 Jun  2 02:31 ..
-rwxr-xr-x 1 root root  2527810 Jun  2 02:31 go-bindata
-rwxr-xr-x 1 root root 10672322 Jun  2 02:31 ngrok
-rwxr-xr-x 1 root root  8841733 Jun  2 03:06 ngrokd
drwxr-xr-x 2 root root     4096 Jun  2 02:35 windows_amd64

```

### 运行测试

服务端：

```bash
 #因为我的服务上的80和443端口上运行着这个博客，所以我把http和https的端口进行了更改
 [root@vultr ngrok]# bin/ngrokd -domain=ngrok.star-chen.com -httpAddr=":8080" -httpsAddr=":8081"
[04:00:34 UTC 2017/06/02] [INFO] (ngrok/log.(*PrefixLogger).Info:83) [registry] [tun] No affinity cache specified
[04:00:34 UTC 2017/06/02] [INFO] (ngrok/log.Info:112) Listening for public http connections on [::]:8080
[04:00:34 UTC 2017/06/02] [INFO] (ngrok/log.Info:112) Listening for public https connections on [::]:8081
[04:00:34 UTC 2017/06/02] [INFO] (ngrok/log.Info:112) Listening for control and proxy connections on [::]:4443
[04:00:34 UTC 2017/06/02] [INFO] (ngrok/log.(*PrefixLogger).Info:83) [metrics] Reporting every 30 seconds


```

windows客户端:

在ngrok.exe的所在目录新建一个简单的配置文件，内容如下：

```
server_addr: "ngrok.star-chen.com:4443"   #尤其注意，这里的域名需要和你生成证书所使用的域名一致
trust_host_root_certs: false  
```

运行客户端程序：

```cmd
PS G:\> .\ngrok.exe -config .\ngrok.cfg  -subdomain local 80 #subdomain为指定的子域名
```

运行的显示，status为online则表明已经建立了转发连接。

```cmd
ngrok

Tunnel Status                 online
Version                       1.7/1.7
Forwarding                    http://local.ngrok.star-chen.com:8080 -> 127.0.0.1:80
Forwarding                    https://local.ngrok.star-chen.com:8080 -> 127.0.0.1:80
Web Interface                 127.0.0.1:4040
# Conn                        0
Avg Conn Time                 0.00ms


```

然后访问 http://local.ngrok.star-chen.com:8080  就可以访问到我本机的xampp测试环境了。当然客户端还可以转发tcp连接，例如我的虚拟机里边的ssh：

```cmd
PS G:\> .\ngrok.exe -config .\ngrok.cfg  -proto tcp 192.168.10.176:22
```

这样子，在外边也可以方便的ssh到内网中的虚拟机了。