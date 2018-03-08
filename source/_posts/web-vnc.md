---
title: 在浏览器中通过VNC远程控制虚拟机
categories:
  - Linux
tags:
  - kvm noVNC
date: 2018-03-08 23:07:14
---

在平时使用云服务器的时候，会用到在浏览器中控制自己的服务器，一直很好奇是怎么实现的。以往也了解到VNC有浏览器applet可以实现，但是applet已经不流行了，而且使用起来也不是很方便。最近搜索到一款神器---noVNC,它将VNC协议转换成websocket，然后浏览器使用canvas绘图来实现浏览器远程控制。

<!--more-->

noVNC的官网：http://novnc.com/info.html， 它也是个开源项目，可以在Github上下载到源代码进行学习，这次就暂时不去深入研究它，先用用看先。

应用环境：

   VNC服务端：kvm为kvm中的虚拟机开放的vnc端口（就是上回刚刚安装好的mini-win7）

   noVNC服务端：正好我有块不关机树莓派，就把noVNC给放到树莓派上跑吧

树莓派上执行的操作：

![image](/blogimg/pi_novnc.png)

浏览器打开相应页面：

![image](/blogimg/bw_novnc.png)

Exciting!