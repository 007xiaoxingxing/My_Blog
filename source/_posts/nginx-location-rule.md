---
title: Nginx location的匹配规则
categories:
  - Linux
tags:
  - nginx
date: 2017-12-13 16:44:23
---

​      今天有个群里的老兄提了个问题，具体就是他配置的nginx不能正常访问静态文件，引起前端未能正确加载js，css等文件。我一看，这问题简单呀，nginx配个location就可以了的。然而自己之前没有实际动手配置过，中途还闹出了笑话，此处是一个悲伤脸。

<!--more-->

   从这个坑里边我有学到点关于nginx配置的知识。

#### location 匹配规则

~      #波浪线表示执行一个正则匹配，区分大小写
~*    #表示执行一个正则匹配，不区分大小写
^~    #^~表示普通字符匹配，如果该选项匹配，只匹配该选项，不匹配别的选项，一般用来匹配目录
=      #进行普通字符精确匹配
@     #"@" 定义一个命名的 location，使用在内部定向时，例如 error_page, try_files

#### location匹配的优先级

= 精确匹配会第一个被处理。如果发现精确匹配，nginx停止搜索其他匹配。
普通字符匹配，正则表达式规则和长的块规则将被优先和查询匹配，也就是说如果该项匹配还需去看有没有正则表达式匹配和更长的匹配。
^~ 则只匹配该规则，nginx停止搜索其他匹配，否则nginx会继续处理其他location指令。
最后匹配理带有"~"和"~*"的指令，如果找到相应的匹配，则nginx停止搜索其他匹配；当没有正则表达式或者没有正则表达式被匹配的情况下，那么匹配程度最高的逐字匹配指令会被使用。



再贴个配置的例子：

```conf
location  = / {
  # 只匹配"/".
  [ configuration A ] 
}
location  / {
  # 匹配任何请求，因为所有请求都是以"/"开始
  # 但是更长字符匹配或者正则表达式匹配会优先匹配
  [ configuration B ] 
}
location ^~ /images/ {
  # 匹配任何以 /images/ 开始的请求，并停止匹配 其它location
  [ configuration C ] 
}
location ~* .(gif|jpg|jpeg)$ {
  # 匹配以 gif, jpg, or jpeg结尾的请求. 
  # 但是所有 /images/ 目录的请求将由 [Configuration C]处理.   
  [ configuration D ] 
}
```

我踩到的坑是location之后的目录会被添加到root配置的目录之后去查找，如果未找到会返回404。举个例子：

```conf
location /static/ {
  root /var/www/html/;
}
```

访问 `http://xxx.com/static/js/a.js` ,nginx查找的路径是/var/www/html/static/js/a.js。

至于这个location的用处嘛，既可以用来处理静态资源，也可以在里边配置反代，获取其他资源。

参考链接：

http://www.nginx.cn/115.html