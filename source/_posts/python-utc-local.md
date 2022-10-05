---
title: python编程笔记之-----UTC格式时间字符串与本地时间对象相互转换
categories:
  - Program
tags:
  - python
date: 2017-11-28 16:35:08
---

在程序编写中，经常会与时间打交道。通常涉及到时间的比较，时间戳与时间字符串的转换等等。在python中，与时间操作有关的常用模块有time和datetime。

<!--more-->

这篇文章记录一下UTC格式的字符串与本地时间字符串相互转换的方法。

```python
import datetime
utc_time_str = '2017-10-15T00:00:00.000Z'
utc_format_str = "%Y-%m-%dT%H:%M:%S.%fZ"
time_obj = datetime.datetime.strptime(start_time, utc_format_str)
time_format_str = "%Y.%m.%d"
local_time_str = time_obj.strftime(time_format_str)
```

```python
# UTC时间转换为时间戳 2016-07-31T16:00:00Z
def utc_to_local(utc_time_str, utc_format='%Y-%m-%dT%H:%M:%SZ'):
    local_tz = pytz.timezone('Asia/Chongqing')
    local_format = "%Y-%m-%d %H:%M"
    utc_dt = datetime.datetime.strptime(utc_time_str, utc_format)
    local_dt = utc_dt.replace(tzinfo=pytz.utc).astimezone(local_tz)
    time_str = local_dt.strftime(local_format)
    return int(time.mktime(time.strptime(time_str, local_format)))
 
# 本地时间转换为UTC
def local_to_utc(local_ts, utc_format='%Y-%m-%dT%H:%MZ'):
    local_tz = pytz.timezone('Asia/Chongqing')
    local_format = "%Y-%m-%d %H:%M"
    time_str = time.strftime(local_format, time.localtime(local_ts))
    dt = datetime.datetime.strptime(time_str, local_format)
    local_dt = local_tz.localize(dt, is_dst=None)
    utc_dt = local_dt.astimezone(pytz.utc)
    return utc_dt.strftime(utc_format)
```

参考链接：

https://www.cnblogs.com/shenwenlong/p/6088765.html

http://blog.sina.com.cn/s/blog_4da051a60102v221.html