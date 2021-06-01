---
title: Arch Linux 关于Gnome-Terminal透明
tags: []
id: '286'
categories:
  - - 技术相关
date: 2018-02-10 21:12:42
---

# 前言

最近装了[Arch](https://www.archlinux.org/)来玩玩，装了gnome桌面环境，当然就要开始美化了，美化到最后，发现我使用最多的终端已经没有透明功能，查阅资料后，得知安装[aur](https://aur.archlinux.org/)里的[gnome-terminal-fedora](https://aur.archlinux.org/packages/gnome-terminal-fedora)即可，于是乎我发现了一些问题，直接用yaourt安装会出错，_**可以直接在安装过程中修改PKGBUILD，也可以手动，本教程讲的手动修改**_

# 首先

从aur里同步两个必要的包

git clone https://aur.archlinux.org/vte3-notification.git

以及

git clone https://aur.archlinux.org/gnome-terminal-fedora.git

![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/02/Screenshot-from-2018-02-10-21-11-51.png) 然后进入vte3-notification，并执行

makepkg

之后会出现如图所示的错误 ![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/02/Screenshot-from-2018-02-10-21-13-38.png) 这就是报错，现在要解决这些错误，打开报错链接，发现403报错，后来经过进一步查看，_**发现网站域名已从pkgs.fedoraproject.org变成src.fedoraproject.org**_ 知道原因以后就轻松了

# 解决方法

在vte3-notification目录下，输入

vi PKGBUILD

将pkgs都改成src即可 ![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/02/Screenshot-from-2018-02-10-21-22-23.png) 同理，进入gnome-terminal-fedora，打开PKGBUILD，将pkgs改成src即可

# 打包安装（安装顺序不能乱，因为terminal编译对第一个软件包有依赖性）

先在vte3-notification目录下输入

makepkg

就开始编译了（要注意的是Python版本得是3或以上，因为我android需要2.7版本，所以编译过程中发生了报错） 编译完成后 ![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/02/Screenshot-from-2018-02-10-21-28-35.png) 之后输入命令开始安装

sudo pacman -U vte3-notification-0.50.2-1-x86\_64.pkg.tar.xz vte-notification-common-0.50.2-1-any.pkg.tar.xz

如图，安装即可 ![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/02/Screenshot-from-2018-02-10-21-31-09.png) 进入gnome-terminal-fedora目录 同理，输入命令后，安装即可，如图 ![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/02/Screenshot-from-2018-02-10-21-31-09.png) 打开gnome-terminal-fedora终端即可，我把原来的gnome-terminal卸载了 ![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/02/Screenshot-from-2018-02-10-21-43-12.png)

# 最终效果图

![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/02/Screenshot-from-2018-02-10-20-26-43.png)

# 最终软件包下载

[gnome-terminal-fedora](https://drive.google.com/file/d/1f_jeUJ9SkPifsm7XkaJsYxi55Rx50NYk/view?usp=sharing) [vte-notification](https://drive.google.com/file/d/1nFo603Iz9OuiVPCyv4gUodIFYGPo5CCH/view?usp=sharing) [vte3-notification](https://drive.google.com/file/d/1kcREzawaMKAUjhOubA66Q72JBIiV76FE/view?usp=sharing)