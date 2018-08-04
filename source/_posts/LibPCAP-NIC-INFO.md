---
title: libpcap折腾系列之获取系统网卡信息
categories:
  - Program
tags:
  - linux
date: 2018-08-04 13:50:39
---

libpcap是是unix/linux平台下的网络数据包捕获库，大多数的网络监控软件都是以它为基础的，例如Tcpdump。

最近想在我的Openwrt路由器上开发一个数据包分析工具，需要学习一下libpcap的功能，于是乎写篇博文记录一下经过。

<!-- more -->

首先接触到的是获取系统中所有的网卡的函数`pcap_findalldevs`,该函数可以获取所有网卡设备，获取到的设备是一个`pcap_if_t`类型的结构体，里边有该设备的具体信息，包括网卡IP。下边是一个示例程序：

```C
#include <pcap.h>
#include <stdlib.h>
#include <stdio.h>
#include <sys/socket.h>
#include <arpa/inet.h>


int main(int argc, char *argv[])
{
    pcap_if_t *alldevs;
    pcap_if_t *device;
    pcap_addr_t *a;
    char *net;
    char *mask;
    char errbuf[PCAP_ERRBUF_SIZE];
    struct in_addr addr;
    struct sockaddr_in *eth_addr;

    if(pcap_findalldevs(&alldevs, errbuf) == -1)
    {
        fprintf(stderr, "Error in pcap_findalldevs: %s\n", errbuf);
        exit(EXIT_FAILURE);
    }
    device = alldevs;
    for(; device != NULL; device = device->next)
    {
        printf("Device name: %s\n", device->name);
        printf("Description: %s\n", device->description);
        printf("\tLoopback: %s\n", (device->flags & PCAP_IF_LOOPBACK)?"yes":"no");

        for(a = device->addresses; a; a = a->next)
        {
            printf("\tAddress Family Name: #%d\n", a->addr->sa_family);
            switch(a->addr->sa_family)
            {
                case AF_INET:
                    printf("\tAddress Family Name: AF_INET\n");
                    if(a->addr)
                        printf("\tAddress: %s\n", inet_ntoa(((struct sockaddr_in*)a->addr)->sin_addr));
                    if(a->netmask)
                        printf("\tNetmask: %s\n", inet_ntoa(((struct sockaddr_in*)a->netmask)->sin_addr));
                    if(a->broadaddr)
                        printf("\tBroadcast Address: %s\n", inet_ntoa(((struct sockaddr_in*)a->broadaddr)->sin_addr));
                    if(a->dstaddr)
                        printf("\tDestination Address: %s\n", inet_ntoa(((struct sockaddr_in*)a->dstaddr)->sin_addr));
                    break;
                case AF_INET6:
                    printf("\tAddress Family Name: AF_INET6\n");
                    if(a->addr)
                        //printf("\tAddress: %s\n", inet_ntoa(((struct sockaddr_in6*)a->addr)->sin6_addr));
                        printf("\tAddress %s\n", "ipv6");
                    break;
                default:
                    printf("\tAddress Family Name: Unknown\n");
                    break;

            }
        }
        printf("\n");
    }
}
```



进行编译：

```shell
#gcc pcap_eth_info.c -o eth_info -lpcap
```

运行结果：

```shell
# ./eth_info
Device name: enp0s3
Description: (null)
        Loopback: no
        Address Family Name: #17
        Address Family Name: Unknown
        Address Family Name: #2
        Address Family Name: AF_INET
        Address: 192.168.10.214
        Netmask: 255.255.255.0
        Broadcask Address: 192.168.10.255
        Address Family Name: #10
        Address Family Name: AF_INET6
        Address ipv6

Device name: any
Description: Pseudo-device that captures on all interfaces
        Loopback: no

Device name: lo
Description: (null)
        Loopback: yes
        Address Family Name: #17
        Address Family Name: Unknown
        Address Family Name: #2
        Address Family Name: AF_INET
        Address: 127.0.0.1
        Netmask: 255.0.0.0
        Address Family Name: #10
        Address Family Name: AF_INET6
        Address ipv6

Device name: nflog
Description: Linux netfilter log (NFLOG) interface
        Loopback: no

Device name: nfqueue
Description: Linux netfilter queue (NFQUEUE) interface
        Loopback: no

Device name: usbmon1
Description: USB bus number 1
        Loopback: no
```

接下来会涉及到如何在网卡上捕获数据包，解开数据包。