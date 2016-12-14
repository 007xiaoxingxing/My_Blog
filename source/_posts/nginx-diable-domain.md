---
title: 配置nginx，防止ip被恶意绑定域名
date: 2016-12-15 04:53:41
categories: Linux
tags:
- nginx
- domain
---
### 0x1 事件诱因  
偶然在百度上搜索了一下我这个博客的域名，发现百度有个快照是另外一个不相干的域名解析到我现在用的这个ip上了。作为一个有点洁癖的我来说是不能忍的。
###  0x2 解决方案  
最简单粗暴的办法就是给域名不匹配的请求返回一个错误代码就可以了，或者委婉的一点，把请求重定向到目前所使用的域名上。我选择了前者。
### 0x3 配置文件具体内容  
```
server {
        listen       80;
        listen       [::]:80 default_server;
        server_name  blog.star-chen.com;
        root         /var/blog;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
	if ($host !~ ".*star-chen.com"){
		return 500;
	}
        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

```
简单粗暴，有效果！