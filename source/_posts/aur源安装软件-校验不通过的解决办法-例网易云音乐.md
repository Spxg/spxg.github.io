---
title: 'AUR源安装软件 校验不通过的解决办法 例:网易云音乐'
tags: []
id: '169'
categories:
  - - 技术相关
date: 2017-05-06 19:00:43
---

 _**经过**_

 _今天太过无聊，就装[Manjaro linux](https://manjaro.org/) 17.01来消遣消遣，以打发时间（顺便说一下，17.01用U盘引导的建议使用[rufus](https://rufus.akeo.ie/)来制作，必须用DD模式）_ 我用的系统必须得装有网易云音乐啊，然后就用安装软件安装

*   先是这样

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/2017-05-06-17-11-29屏幕截图-1.png)

*   然后开始安装，变成了这样

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/2017-05-06-17-12-06屏幕截图.png)   ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/17c1fa5eddcdef96.jpg)我的网易云还没装就夭折了？ service.html是个什么玩意儿，似乎无关紧要。我在想能不能自己构建一个出来呢，把MD5修正了装上去，或者直接删掉验证更快，示范第一种

**_开始动手_**

*   我先来到了[AUR源的网站](https://aur.archlinux.org/packages) ，找到了网易云音乐的项目

![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/2017-05-06-17-21-19屏幕截图-1024x576.png)

*   看了看 [service.tml的内容](http://music.163.com/html/web2/service.html)

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/2017-05-06-17-20-46屏幕截图-1024x576.png)    ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/42805acb2ed6e66f.jpg)大概是条款变动了，作者没更新

*   开始同步修改，输入命令   git clone https://aur.archlinux.org/netease-cloud-music.git

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/2017-05-06-17-19-34屏幕截图.png) 有这几个文件![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/2017-05-06-18-35-16屏幕截图.png)，其中PKGBUILD和.SPCINFO都有对文件的MD5的验证内容，我们从这动手脚

*   下载[service.html](http://music.163.com/html/web2/service.html)，记录其MD5，修改

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/2017-05-06-18-39-40屏幕截图.png) 打开PKGBUILD，修改为正确值 ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/2017-05-06-18-41-16屏幕截图.png) 打开.SECINFO,修改为正确值 ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/2017-05-06-18-43-48屏幕截图.png)

*   构建

在同步目录输入makepkg ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/2017-05-06-18-54-00屏幕截图.png)

*   安装

输入命令 sudo pacman -U netease-cloud-music-1.0.0-3-x86\_64.pkg.tar.xz ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/2017-05-06-18-55-40屏幕截图.png)

*   安装成功

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/2017-05-06-18-57-04屏幕截图.png)![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/d49e95cfe2995aa.png)  

_**注意**_

本教程应该同样适用于类似问题
