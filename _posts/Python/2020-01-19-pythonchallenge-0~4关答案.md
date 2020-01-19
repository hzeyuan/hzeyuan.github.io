---
layout: post
title: "[pythonchallenge]-- 0~4关讲解"
subtitle: '[pythonchallenge]-- 0~4关讲解'
author: "Hzy"
header-style: text
tags:
  - Python
---


###  今天发现一个练习python的好项目，叫pythonchallenge，通过游戏关卡的形式，我们可以更好的学习Python！！



 [Hzy的博客](https://hezeyaun.cn)

> 今天说说前面5关，都遇到了什么。


### 第0关

> 看到图片，要求我们求出2的38次方

```
print(2**38)# 274877906944
```

> so easy,我们把结果复制到.html前面就可以来到下一关了。


### 第1关

>也是一张图片，可以看到

* K -> M
* O-> Q
* E -> G

> 观察一下规律，好像都是按26个字母，向前数两位的结果，下面还有一段话，我们来用我们的规律翻译一下它。

```
## 对下面的文字进行
instr = "abcdefghijklmnopqrstuvwxyz"
outstr = "cdefghijklmnopqrstuvwxyzab"
strans = str.maketrans(instr,outstr)

test = "g fmnc wms bgblr rpylqjyrc gr zw fylb. rfyrq ufyr amknsrcpq ypc dmp. bmgle gr gl zw fylb gq glcddgagclr ylb rfyr'q ufw rfgq rcvr gq qm jmle. sqgle qrpgle.kyicrpylq() gq pcamkkclbcb. lmu ynnjw ml rfc spj."
print(test.translate(strans))


# http://www.pythonchallenge.com/pc/def/map.html
print("map".translate(strans))

# 下一个链接为：http://www.pythonchallenge.com/pc/def/ocr.html
```

### 第2关

> 第二关是一本书，我们右键，打开页面源码查看，发现下面有个注释，要求我们在一堆乱码中找到`关键的字母`。

```
import requests
import re

content = requests.get('http://www.pythonchallenge.com/pc/def/ocr.html').text
anwser = re.sub('\n|\t|\r', '', content)  # 去除一些空格
anwser = re.findall("find.*?<!--(.*?)-->", anwser)  # 取出注释
anwser = list(str(anwser))  # 转化为列表
key = (s for s in anwser if s.isalpha()) #找到里面的字母
print("".join(key))
# 结果 equality

```

### 第3关

> 第3关和第2关很像，我们直接打开页面源码。

* 按照题目的要求，一个小写字母周围有三个大写字母
* 我们利用正则'[a-z]+[A-Z]{3}([a-z])[A-Z]{3}[a-z]+'匹配

```
import re

import requests

content = requests.get('http://www.pythonchallenge.com/pc/def/equality.html').text
notes = re.search('<!--(.*?)-->', re.sub('\n', '', content))
if notes:
    ret = re.findall('[a-z]+[A-Z]{3}([a-z])[A-Z]{3}[a-z]+', notes.group())
    print(ret)

# 答案：linkedlist

```

### 第4关

>这一题呢，也有一个图，然后我们可以点击进去，发现只有一串字符
* and the next nothing is 44827
* 右键查看源码，注释里告诉我们，通过一直找下一个数字，我们最多400次可以找到答案。

> 然后在编写过程你会发现以下几个问题

* ` Yes. Divide by two and keep going.`你需要直接用数字除以2,而不是用正则直接匹配
* There maybe misleading numbers in the text. One example is 82683. Look only for the next nothing and the next nothing is 63579

* 正则会匹配出两个数字，你得判断哪个数字才是你需要的。


```
import requests
import re

url = "http://www.pythonchallenge.com/pc/def/linkedlist.php?nothing=77864"
text = requests.get(url).text

for i in range(400):
    number_list = re.findall("\d+", text)
    if len(number_list) == 1:
        url = "http://www.pythonchallenge.com/pc/def/linkedlist.php?nothing=" + number_list[0]
    elif len(number_list) == 0:
        number = url.split('nothing=')[1]
        url = "http://www.pythonchallenge.com/pc/def/linkedlist.php?nothing=" + str(int(number) / 2)
    elif len(number_list) == 2:
        url = "http://www.pythonchallenge.com/pc/def/linkedlist.php?nothing=" + number_list[0]
        content = requests.get(url).text
        if content.find("You've been misleaded to here.") != -1:
            print("正确数字为{}".format(number_list[1]))
            url = "http://www.pythonchallenge.com/pc/def/linkedlist.php?nothing=" + number_list[1]
    r = requests.get(url)
    if r.status_code == 200:
        text = r.text
        print(str(i) + ": ", text)

# http://www.pythonchallenge.com/pc/def/linkedlist.php?nothing=66831的下一个答案：peak.html
```




### 明天再来看看就接下里的题目！