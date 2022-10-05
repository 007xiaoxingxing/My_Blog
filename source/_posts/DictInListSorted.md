---
title: Python编程实践-----List中嵌套Dict，根据Dict的value进行排序
categories:
  - Program
tags:
  - python
date: 2017-11-17 11:03:54
---

​	今天遇到一个问题，我有一个list，其中放了多个dict，我想根据dict中的value对这个list进行排序。经过搜索，发现可以利用python中的sorted方法来完成。特记录一下，以免忘记。

<!--more-->

这是原本的list内容：

```json
[
    {
      "date": "2017年11月17日 10:20",
      "link": "http://toutiao.secjia.com/cisco-uav-cve-2017-12337",
      "title": "12款思科产品出现未授权访问漏洞CVE-2017-12337 可拿设备Root权限"
    },
    {
      "date": "2017年11月17日 09:55",
      "link": "http://toutiao.secjia.com/office-mcv-cve-2017-11882",
      "title": "Office内存破坏漏洞CVE-2017-11882 可执行任意代码失败了还可DoS"
    },
    {
      "date": "2017年11月17日 09:42",
      "link": "http://toutiao.secjia.com/ios-xe-xss-cve-2017-12304",
      "title": "思科IOS及IOS XE软件再次爆出跨站脚本漏洞CVE-2017-12304 可以执行任意代码"
    },
    {
      "date": "2017年11月16日 13:30",
      "link": "http://toutiao.secjia.com/vep-charter",
      "title": "下载 | 美政府发布《未分类漏洞权益VEP宪章》 增加0Day漏洞审查及披露的透明度"
    },
    {
      "date": "2017年11月16日 15:34",
      "link": "http://toutiao.secjia.com/mac-security-guideline",
      "title": "mac电脑安全指南11招 普通用户很容易上手"
    },
    {
      "date": "2017年11月16日 10:25",
      "link": "http://toutiao.secjia.com/2017q3-apt-report",
      "title": "卡巴发布2017Q3APT威胁报告 指出攻击者有针对供应链攻击的转移倾向"
    },
    {
      "date": "2017年11月16日 14:20",
      "link": "http://toutiao.secjia.com/libxls-rce",
      "title": "开源Excel读取库libxls爆出7个远程代码执行漏洞CVE-2017-12108 可破坏内存"
    },
    {
      "date": "2017年11月16日 09:46",
      "link": "http://toutiao.secjia.com/oracle-tuxedo-cve-2017-10269",
      "title": "Oracle Tuxedo远程安全漏洞CVE-2017-10269 Core组件及Fusion中间件受到影响"
    }
  ]
```

我的需求是根据把list根据dict中的date进行排序，解决办法是这样的：

```python
news_arr = sorted(news_arr, key=lambda e: e.__getitem__('date'), reverse=True)
```

参考链接：

http://m.blog.csdn.net/Tangzongyu123/article/details/75200619