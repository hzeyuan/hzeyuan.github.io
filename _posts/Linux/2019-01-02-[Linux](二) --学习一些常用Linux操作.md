---
layout: post
title: "[Linux]（二） --学习一些常用Linux操作"
subtitle: '[Linux]（二） --学习一些常用Linux操作'
author: "Hzy"
header-style: text
tags:
  - Linux
---

今天学习了一些在Linux里常用的命令，别一到命令行界面就傻了....
单纯看命令不太好，于是就实验了下：

>例子：我想在 usr/local/games文件夹里创建一个test.txt文本，输入文本，进行修改，移动，删除等。
>
##### 1.先查看当前目录
>root@kali:~# pwd
/root
>
目录中并没有usr文件夹，所以返回上一级看看
![当前目录内容](https://upload-images.jianshu.io/upload_images/11948845-ecea3a2dc6b4af0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 2.返回到上一级
>root@kali:/# cd ../
/
>
![当前目录中有usr文件夹](https://upload-images.jianshu.io/upload_images/11948845-5d7d99e3973c3d9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##### 3.进去到指定目录,创建test.txt文件，输入123321
>root@kali:/# cd usr/local/game
root@kali:/usr/local/games# touch test.txt 
root@kali:/usr/local/games# echo 123321 >test.txt
>
##### 4.查看文本里的内容
![查看文本里的内容](https://upload-images.jianshu.io/upload_images/11948845-734d3e47ab2b432c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##### 5.用sed命令修改你文本内容，用grep命令进行查找
![用sed命令修改你文本内容](https://upload-images.jianshu.io/upload_images/11948845-8a8124bb0d34c50d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![用grep命令进行查找](https://upload-images.jianshu.io/upload_images/11948845-312cd805d8fbf3d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>123321是旧文本，789987是新文本，-i：直接操作文本内容，不在屏幕打印。
s：替换的意思，g:结束
其实就是固定的格式。
>
当然还可以使用vim编辑器，但我还不会..下次学习。

##### 6.复制文本和移动文本：
![复制文本](https://upload-images.jianshu.io/upload_images/11948845-b7b63e45b0a57121.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
把test.txt也移动到home/user用户目录下就是~
![移动文本](https://upload-images.jianshu.io/upload_images/11948845-0e1e08c4f909e5ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 7.删除文本
![删除文本](https://upload-images.jianshu.io/upload_images/11948845-6d014d5df0b26c87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

刚刚使用到的命令：
|命令|功能|命令|功能|
|---|---|---|----|
|pwd|显示当前的目录|ls|查看目录下内容|
|cd|改变所在目录|cat|显示文件内容|
|grep|在文件中查找某字符|cp|复制文件|
|touch|创建文件|mv|移动文件|
|rm|删除文件|rmdir|删除目录|

命令还有很多具体的用法，需要的时候就man了。
今天就学习这些了。

参考文章：
https://wenku.baidu.com/view/ffc4627981c758f5f61f67a1.html