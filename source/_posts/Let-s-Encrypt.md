---
title: Let's Encrypt && Nginx配置证书，实现https访问
date: 2016-12-15 15:01:36
categories:
- Linux
tags:
- https
- nginx
---

### 0x1:Why?

在聊HTTPS之前，先来了解一下被广泛使用的HTTP协议

> HTTP函数作为请求-响应于协议的客户端-服务器计算模型。一个网络浏览器，例如，可以是*客户机*和一个计算机上运行的应用托管一个网站可以是*服务器*。该客户机提交一个HTTP *请求*消息发送到服务器。服务器，它提供*的资源*如HTML文件和其他内容，或代表客户机的执行其它功能，返回一个*响应*消息给客户端。响应包含关于请求完成状态信息，并且还可以含有在其消息主体请求的内容。
>
> 网络浏览器是一个的例子*的用户代理*（UA）。其他类型的用户代理包括由搜索服务提供商（使用的索引软件网络爬虫，语音浏览器，移动应用程序和其他软件访问，消费，或显示网页内容。
>
> 的HTTP被设计成允许中间网络元件，以改善或允许客户端和服务器之间的通信。高流量的网站往往受益于Web缓存的代表提供的内容服务器上游服务器以提高响应时间。Web浏览器的缓存以前访问网络资源并尽可能减少网络流量重用他们。HTTP代理服务器的专用网络边界可以方便为客户的沟通没有一个全球可路由的地址，通过与外部服务器中继消息。
>
> HTTP是一个应用层协议的框架内，设计的因特网协议套件。它的定义假定底层和可靠的传输层的协议，和传输控制协议（TCP）是常用的。然而HTTP可以适于使用不可靠的协议，如用户数据报协议（UDP），例如在HTTPU和简单服务发现协议（SSDP）。
>
> HTTP资源得到确认和位于网络通[统一资源定位器（URL）的，使用统一资源标识符（URI的）计划，*HTTP*和*HTTPS*。URI和超链接的HTML文档形成相互关联的超文本文档。
>
> HTTP / 1.1是原来的HTTP（HTTP / 1.0）的修订。在HTTP / 1.0的单独连接到同一台服务器为每个资源的要求而作出。HTTP / 1.1可以重复使用的连接多次下载图像，脚本，样式表，*等等*页面已交付之后。因此，HTTP / 1.1的通信经验少的延迟为建立TCP连接的呈现相当大的开销。

-------引自wikipedia [维基百科.HTTP词条]( https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)

<!-- more -->

但是HTTP协议本身不具备加密功能，所有的通讯数据都是在网络上裸奔。所以HTTP链接很容易被“中间人攻击“，HTTP链接网络环节中的中间人（浏览器、路由器、ISP.....）可以对数据进行嗅探，欺骗，劫持，阻挡等等。明文传输的用户数据还容易泄露。

那什么又是HTTPS呢？  

> HTTPS（全称：Hyper Text Transfer Protocol over Secure Socket Layer），是以安全为目标的[HTTP](http://baike.baidu.com/view/9472.htm)通道，简单讲是HTTP的安全版。即HTTP下加入SSL层，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。 它是一个URI scheme（抽象标识符体系），句法类同http:体系。用于安全的HTTP数据传输。https:URL表明它使用了HTTP，但HTTPS存在不同于HTTP的默认端口及一个加密/身份验证层（在HTTP与TCP之间）。这个系统的最初研发由网景公司(Netscape)进行，并内置于其浏览器Netscape Navigator中，提供了身份验证与加密通讯方法。现在它被广泛用于万维网上安全敏感的通讯，例如交易支付方面。  

------引自百度百科 [百度百科.HTTPS词条](http://baike.baidu.com/link?url=ZSi0Ny56ylfHKJ4PEny2ogN3HkMf8QClbf_TulpLkRcei0-Z2w4SnxLVBnJPX5qkHHZOgKgQl78yBX60xdorUa)

相比较于HTTP协议，HTTPS在HTTP的基础上增加了SSL（安全套接字层）。SSL的引入为连接的建立和数据的传输都进行了加密，在密钥足够健壮的情形下，是比较安全的。相较于HTTP，HTTPS提供了以下三个强大的功能：

1. 内容加密。浏览器到百度服务器的内容都是以加密形式传输，中间者无法直接查看原始内容。
2. 身份认证。保证用户访问的是百度服务，即使被 DNS 劫持到了第三方站点，也会提醒用户没有访问百度服务，有可能被劫持
3. 数据完整性。防止内容被第三方冒充或者篡改。

###  0x2:How?

要进行HTTPS加密，我们首先需要向证书颁发机构（CA）申请加密证书。免费的证书颁发机构有Startssl [Start ssl](https://www.startssl.com/)、Let's Encrypt [Let's Encrypt](https://letsencrypt.org/)等。我用的是Let's Encrypt颁发的证书，它操作起来比较简单。它的官方网站上有介绍具体的申请步骤。[申请Let's encrypt证书](https://letsencrypt.org/getting-started/) 下面是我参考网上一哥们一篇博文[Jerry Qu 申请Let's Encrypt](https://imququ.com/post/letsencrypt-certificate.html)实际申请证书的具体过程如下：

1. 创建帐号

创建一个目录，例如 *ssl*，用来存放各种临时文件和最红生成的证书文件。首先创建一个私钥文件

```bash
#openssl genrsa 4096 > account.key
```

2. 创建CSR文件

生成CSR（Certificate Signing Request，证书签名请求）文件。需要准备一个域名私钥，这里我选择RSA私钥。

```bash
#openssl genrsa 4096 > domain.key
#openssl req -new -sha256 -key domain.key -out domain.csr //手动生成csr文件，根据提示进行操作即可
```

3. 配置验证服务

CA在颁发证书时，需要验证域名的所有权，证明你对该域名所在的服务器有操作的权限。Let's Encrypt采用的是服务器上生成一个随机验证文件，再访问CSR中填写的域名，如果访问成功，则证明你对该域名有所有权。为了偷懒，我就不像那位博主一样建立单独的目录了，而是直接在我的web根目录建立一个隐藏目录，用来存放之后生成的随机验证文件。

```bash
#mkdir -p /var/blog/.well-known/acme-challenge //建立验证文件存放目录
```

4. 获取HTTPS证书

先下载一个acme-tiny的脚本，保存在*ssl*目录：

```bash
#wget https://raw.githubusercontent.com/diafygi/acme-tiny/master/acme_tiny.py
```

指定账户私钥、CSR、验证目录，然后执行脚本：

```bash
python acme_tiny.py --account-key ./account.key --csr ./domain.csr --acme-dir /var/blog/ > ./signed.crt

```

如果执行成功，在当前目录下会生成一个signed.crt文件，这就是申请号的证书文件。

下载Let's Encrypt的中间证书，并和网站证书合并在一起:

```bash
#wget -O - https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem > intermediate.pem
#cat signed.crt intermediate.pem > chained.pem
```

再把根证书和中间证书合并在一起：

```bash
#wget -O - https://letsencrypt.org/certs/isrgrootx1.pem > root.pem
#cat intermediate.pem root.pem > full_chained.pem
```

5. 配置Nginx，开启https

   ```bash
   ssl_certificate     ~/ssl/chained.pem;
   ssl_certificate_key ~/ssl/domain.key;
   ```

   ​

   因为Let's Encrypt签发的证书的有效期只有90天，需要定时使用脚本进行更新。就是把上面的获取过程写进一个脚本，设置crontab，让其自动执行更新即可。我也不确定这台服务器我会续费到多久，暂时先用着吧。  

   ----后续，续期证书的脚本

   ```bash
   #!/bin/bash
   openssl genrsa 4096 > account.key
   openssl genrsa 4096 > domain,key
   openssl req -new -sha256 -key domain.key -out domain.csr
   python acme_tiny.py --account-key ./account.key --csr ./domain.csr --acme-dir /var/www/blog/.well-known/acme-challenge/ > ./signed.crt
   wget -O - https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem > intermediate.pem
   cat signed.crt intermediate.pem > chained.pem
   wget -O - https://letsencrypt.org/certs/isrgrootx1.pem > root.pem
   cat intermediate.pem root.pem > full_chained.pem

   ```

   ​

### 0x3 That's All

经过一番折腾，在我的域名前，终于看到了浏览器地址栏我的域名前有了一把绿色的小锁，*It's Encrypt!*  

最后还是感谢一下Jerry Qu撰写的博客文章， [Jerry Qu](https://imququ.com/)