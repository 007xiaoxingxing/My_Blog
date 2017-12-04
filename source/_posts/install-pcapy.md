---
title: win10+python2.7  安装pcapy
categories:
  - Program
tags:
  - python
date: 2017-12-04 15:33:21
---

1.首先需要安装vcforpython

<!--more-->

https://download.microsoft.com/download/7/9/6/796EF2E4-801B-4FC4-AB28-B59FBF6D907B/VCForPython27.msi

2.设置注册表

http://blog.csdn.net/donger_soft/article/details/44838109

3.下载winpcap开发包

http://www.winpcap.org/install/bin/WpdPack_4_1_2.zip

4.解压winpcap开发包到vc的对应包含目录

父目录：C:\Users\用户名\AppData\Local\Programs\Common\Microsoft\Visual C++ for Python\9.0\VC

对应的lib，include

5.pip install pcapy或者下载pcapy到本地解压，执行python setup.py install