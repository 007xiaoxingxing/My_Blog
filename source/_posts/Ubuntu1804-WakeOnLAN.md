---
title: Ubuntu18.04开启WakeOnLAN
categories:
  - Linux
tags:
  - ubuntu
date: 2018-09-25 20:45:39
---

  最近我发现我的一台Ubuntu18.04的机器不能正常的进行WakeOnLAN了，回想之前是Ubuntu16.04的时候还是正常的，猜想是因为系统设置的问题，于是乎求助google，进行了一番搜索后找到了如下的解决方法：

<!--more-->

#### 安装ethtool

```shell
root@dev-star:~# apt-get install ethtool
```

#### 建立开机启动服务

在/etc/systemd/system中建立wol@.service

```shell
root@dev-star:/opt# cd ~
root@dev-star:~# cat /etc/systemd/system/wol\@.service 
[Unit]
Description=Wake-on-LAN for %i
Requires=network.target
After=network.target

[Service]
ExecStart=/sbin/ethtool -s %i wol g
Type=oneshot

[Install]
WantedBy=multi-user.target
```

enable开机启动项

```shell
root@dev-star:~# systemctl enable wol@enp3s0
```

#### 重启后检查

```shell
root@dev-star:~# ethtool enp3s0
Settings for enp3s0:
	Supported ports: [ TP MII ]
	Supported link modes:   10baseT/Half 10baseT/Full 
	                        100baseT/Half 100baseT/Full 
	                        1000baseT/Half 1000baseT/Full 
	Supported pause frame use: No
	Supports auto-negotiation: Yes
	Supported FEC modes: Not reported
	Advertised link modes:  10baseT/Half 10baseT/Full 
	                        100baseT/Half 100baseT/Full 
	                        1000baseT/Full 
	Advertised pause frame use: Symmetric Receive-only
	Advertised auto-negotiation: Yes
	Advertised FEC modes: Not reported
	Link partner advertised link modes:  10baseT/Half 10baseT/Full 
	                                     100baseT/Half 100baseT/Full 
	                                     1000baseT/Full 
	Link partner advertised pause frame use: Symmetric Receive-only
	Link partner advertised auto-negotiation: Yes
	Link partner advertised FEC modes: Not reported
	Speed: 1000Mb/s
	Duplex: Full
	Port: MII
	PHYAD: 0
	Transceiver: internal
	Auto-negotiation: on
	Supports Wake-on: pumbg
	Wake-on: g  #这里是g表示已经成功了
	Current message level: 0x00000033 (51)
			       drv probe ifdown ifup
	Link detected: yes
```

经过这一番操作，在系统关机后依然可以使用wakeonlan唤醒机器了。