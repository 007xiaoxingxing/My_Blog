---
title: 利用Python的PIL截屏
date: 2016-12-04 10:23:02
categories: Program
tags:
- python
- PIL
---

### 0x1：在Windows 64位系统上安装python的PIL库

Pillow的官方网站上默认只提供32位版本的安装程序，所以我们需要到第三方网站去下载64位版本的Pillow安装包。[Win 64 PIL下载地址](http://www.lfd.uci.edu/~gohlke/pythonlibs/) 下载下来会发现是.whl扩展名。其安装方法：  

```cmd
pip install Pillow-3.4.2-cp27-cp27m-win_amd64.whl
```

### 0x2:利用PIL截屏

```python
from PIL import ImageGrab
image=ImageGrab.grab()
image.save("E:/grab.jpg","jpeg")

```

仅仅需要简单的两行代码就可以实现截取整个屏幕并保存为文件，很方便。grab方法还可以指定参数，截取屏幕某坐标范围内的内容。