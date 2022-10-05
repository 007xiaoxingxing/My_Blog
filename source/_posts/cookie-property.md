---
title: Cookie各属性的作用
categories:
  - Program
tags:
  - web cookie
date: 2018-03-08 20:33:31
---

#### Cookie是什么

由于HTTP协议是一种无状态协议，服务端是无法根据网络连接来区别不同用户身份的。Cookie在这种情况下充当了一个通行证的作用，服务器就可以根据Cookie来确认用户身份了。

Cookie实际上是一小段文本信息，通常情况下是通过HTTP HEADER进行发送和接收。

<!--more-->

![image](/blogimg/setcookie.png)

![image](/blogimg/sendcookie.png)

#### Cookie的几个常用属性

cookie的值是已键值对表示的，除开Cookie带的值，还有一些用于表示过期时间和用于安全目的的属性。

* name   Cookie值的键


* value   Cookie值的内容


* domain     表示可以访问该Cookie的域名


* path     可以访问该Cookie的页面路径。常见的值有/


* expires      Cookie的超时时间，到达时间则失效。


* size    Cookie的大小，这个不常见


* http-only        表明该Cookie只能在http头中传输，js不能访问到


* secure     表明Cookie只能在使用https协议的传输，使用http协议时不会传输。这里遇到一个坑点，当把secure属性去掉之后，需要重启浏览器才能生效。

#### Cookie的几个作用

session管理：      常见的是session_id这个Cookie，在http访问的过程用于用户身份的标识。

个性化 ：       这个可见于购物车

user tracking：    追踪用户行为，貌似很多广告就是因为这个。

#### 针对Cookie的攻击手段 

Cookie窃取，例如通过站点的xss漏洞可能窃取到用户浏览器的Cookie。

Cookie欺骗，因为Cookie是用户可控的，如果某些敏感字段写在Cookie里边，很容易被恶意篡改。