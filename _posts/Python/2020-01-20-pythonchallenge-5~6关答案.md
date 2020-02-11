---
layout: post
title: "[pythonchallenge]-- 5~6关讲解"
subtitle: '[pythonchallenge]-- 5~6关讲解'
author: "Hzy"
header-style: text
tags:
  - Python
---


> 上一篇文章:[[pythonchallenge]-- 0~4关讲解](https://zhuanlan.zhihu.com/p/103515832)

 [Hzy的博客](https://hzeyuan.cn)


 ### 第5关

 > 这一关同样，我们得看网页源码，会发现一个banner.p链接，然后我们打开，发现一堆符号，我们需要`反序列化`

 * 这里我们使用pickle来进行反序列化。
* 序列化后，会发现是一个二维列表，列表里的很多项`('',数字)`，或者`('#',数字)`
 * 这里的数字指的是，符号出现的次数。
 * 我们把所有的字符打印出来，就可以看见答案了。
 >代码:

 ```
 import requests
import pickle

text = requests.get("http://www.pythonchallenge.com/pc/def/banner.p").content
print(pickle.loads(text))
for list in pickle.loads(text):
    print(''.join(l[0] * l[1] for l in list))


# http://www.pythonchallenge.com/pc/def/channel.html
 ```

 > 会发现一个channel的字符



 ### 第6关

 > 这一关就只有一个拉链的图，同样我们打开源码

 
 #### 第一行有注释：<!-- <-- zip -->，发现整个源码里，有个表单就没其他的了，看了半天，知道咋整。

 * 最后发现原来要我们把html换成zip,下载一个zip文件
* 打开这个文件我们会发现，全是一些txt的文件。
* 文件里写着：Next nothing is 91038，跟之前的某一关很像。
* 里面有个readme.txt：`提示我们要从90052.txt开始，同时答案在压缩包中`！
* 然后我们就从90052开始遍历呗。
* 结果到了46145.txt，会打印：Collect the comments.
* 结合刚刚提示答案在压缩包中，原来答案就是每个文件的注释合起来。
* 我们打印所有的文件的注释，得到结果：hockey

>代码


```
import re

first = "90052.txt"
while True:
    with open("./channel/" + first, "r") as t:
        text = t.readline()
        answer = re.findall("\d+", text)
        if len(answer) == 1:
            first = answer[0] + ".txt"
            print("path:{} and text:{}".format(first, text))
        else:
            print(first)
            print(text)
            break
# Collect the comments.
import zipfile
first = "90052.txt"
file = zipfile.ZipFile('./channel.zip', 'r')
print(file)
while True:
    line = str(file.read("%s" % first))
    m = re.search("\d+", line)
    print(file.getinfo("%s" % first).comment.decode("utf-8"), end="")
    if m is None:
        print(file.getinfo("%s" % first).comment.decode("utf-8"))
        break

    first =  m.group(0)+".txt"

```


### 接着打开连接：http://www.pythonchallenge.com/pc/def/hockey.html。

* 会得到下面一句话：it's in the air. look at the letters.
* ... 字母在空气中..答案是氧气:oxygen,我看了别人的才知道的，俺想不出来。


> 明天在来做一下题。


