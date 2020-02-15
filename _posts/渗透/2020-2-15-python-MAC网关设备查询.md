---
layout: post
title: "[python渗透学习]-自己写一个MAC网关设备查询脚本"
subtitle: '网关设备查询脚本'
author: "Hzy"
header-style: text
tags:
  - 渗透
  - 
---

> 原文连接:[Hzy 博客](https://hzeyuan.cn)

### 今天在通过apr协议获取到MAC地址的时候，发现居然没有现成查询网关涉笔的包...


> 找了好久，发现不少网站有，比如国内的一个：https://mac.51240.com/


### 可以利用爬虫查询..但我想自己写一个哈哈哈，那就开始动手吧。


### 1.首先我们需要一份纪录了MAC和设备映射关系的oui文件:

>可以在这里下载：http://standards-oui.ieee.org/oui/oui.txt


### 2. 有了文件以后呢，我们就要充分利用这个文件啦。

> 思路，对这个oui.txt进行处理，把十六进制和对应的设备名称存到一个新的文件里。

```
def new_oui():
    i = 1
    results = []
    with open("oui.txt", encoding='UTF-8')as f:
        for l in f.readlines():
            l = l.strip('\n').strip(" ").strip('\t')
            if l == '':
                continue
            else:
                ret = re.findall(r"^[A-F0-9].-[A-F0-9].-[A-F0-9].+$", l)  # eg 9C8E99
                if ret:
                    s = ret[0].replace('\t', '')
                    # ''.join(s[:9].split('-'))
                    i += 1
                    six = ''.join(s[:9].split('-'))
                    results.append([int(six, 16), six, s[16:]])
        results.sort()
    with open('new-oui.txt', 'w')as f:
        for r in results:
            print(r)
            f.write(str(r[0]) + "|" + str(r[1]) + "|" + r[2] + "\n")
```


>然后我们可以到表变成了这样

```
12|00000C |Cisco Systems, Inc
98|000062 |BULL HN INFORMATION SYSTEMS
107|00006B |Silicon Graphics

```


### 3.接下来，我们在写一个解析的方法，当我们输入MAC地址时，返回MAC的设备名

> 思路其实也很简单，把文件加载进来，二分查找


```
def MACparse(MAC):
    result = dict()
    l = list()
    if isinstance(MAC, str):
        l.append(MAC)
    elif isinstance(MAC, list):
        l = MAC
    with open('new-oui.txt', 'r') as f:
        oui = f.readlines()
        left, right = 0, len(oui) - 1
        for m in l:
            m1 = m.replace(':', '')[:6]
            # 二分查找
            while left <= right:
                aim = int(m1, 16)
                mid = (left + right) // 2
                oui_split = oui[mid].split('|')
                n = int(oui_split[0])
                if n > aim:
                    right = mid - 1
                elif n < aim:
                    left = mid + 1
                else:
                    result[m] = oui_split[2].split('\n')[0]
                    break
    return result
```



### 最后就完成啦！但是感觉这个oui文件不够全...




