---
title: Arch Linux基本环境安装文本与视频教程(不含桌面环境)
tags: []
id: '326'
categories:
  - - 技术相关
date: 2018-05-06 11:46:10
---

_前言_

**在很早之前就想发布一个安装[_arch\_linux_](https://zh.wikipedia.org/wiki/Arch_Linux)的教程，可是一直没有时间，最近一名网友想安装arch，所以我就写了这篇教程，还做了个视频**

_视频教程_

[Youtube](https://www.youtube.com/watch?v=nH7NuSXKPBQ)

[Onedrive](https://spxg-my.sharepoint.com/:v:/g/personal/spxg_spxg_me/EfW0BObAeGBJqF3CQb_UTsMBRIf0AeHTP2ptlgvQnDyHow?e=EdesMG)

[Google Drive](https://drive.google.com/open?id=1U8_4qk-u1Hh9A7qkuIoyefPaBWhodw0W)

## 准备工作

      支持64位的电脑一台

有稳定的网络环境(arch为在线安装)

       有可以用的引导设备(至于[**镜像**](https://www.archlinux.org/download/)怎么写入并且引导，不做阐述)

## 开始安装

首先并不困难地进入了命令行

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180505235341.png)

### 测试网络联通性

`ping wordpress-1253676827.file.myqcloud.com` ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180505235453.png)

### 换源

`nano /etc/pacman.d/mirrorlist` 把第一个地址改成http://mirrors.aliyun.com/xxxx，如图，这是阿里云的镜像站(视频后面因为同步慢，我又改了地址) ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180506001823.png) 改完后，输入CTRL+X，然后选择y保存退出

### 分区

`fdisk -l` 这个命令是用来查看硬盘设备的，找到我们要分区的硬盘，之后输入 `fdisk /dev/sdax`  _//_ _我这里是sda_ ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180506002409.png) 我的分区计划是

*   EFI:200MB
*   根目录:10GB
*   HOME:9.8GB

如果是gpt分区可以输入g，然后再输入n，mbr直接输入n创建分区，回车两次，然后输入你要的空间大小，可以表示为+xM或者+xGB，回车，再创建，以此类推，最后输入w保存，如图 ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180506003803.png) 分区好后，我们可以通过命令来查看分区情况，输入lsblk ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180506004255.png) 可以清楚的看到，sda硬盘被分了三个区，接下来我们要做的就是格式化分区

*   efi:fat分区格式
*   根目录:ext日志式
*   HOME:ext日志式

所以我们输入命令，请具体情况具体分析 `mkfs.fat -F32 /dev/sdax` _// 我这x=1_ _**mbr的输入mkfs.ext4 /dev/sdax**_ `mkfs.ext4 /dev/sday`  _//_ _我这y=2_ `mkfs.ext4 /dev/sdaz` _//_ _我这z=3_ ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180506004442.png)

### 挂载分区

依次输入命令 `mount /dev/sda2 /mnt`  _// 挂载sda2分区到/mnt，也就是装完系统后的根目录_ `mkdir -p /mnt/home` _// 创建/mnt/home目录，也就是装完系统后的/home目录_ `mount /dev/sda3 /mnt/home` _// 挂载sda3分区到/mnt/home，也就是装完系统后的/home_ `mkidr -p /mnt/boot/EFI` _// 创建/mnt/boot/EFI目录，也就是装完系统后的/boot/EFI目录，**mbr的输入mkidr -p /mnt/boot**_ `mount /dev/sda1 /mnt/boot/EFI` _// 挂载sda1分区到/mnt/boot/EFI，也就是装完系统后的/boot/EFI，**mbr的输入mount /dev/sda1 /mnt/boot**_ **注意顺序不能乱，因为挂/mnt的时候/mnt/home和/mnt/boot/EFI会莫名的消失** ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180506005530.png)

### 安装基本环境

一路回车即可 `pacstrap -i /mnt base base-devel` ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180506005751.png)

### 生成分区表

`genfstab -U /mnt >> /mnt/etc/fstab` `cat /mnt/etc/fstab` _// 检验分区表是否正确_ ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180506005954.png)

### 切换环境

`arch-chroot /mnt /bin/bash`

### 然后修改语言和时区

`nano /etc/locale.gen` 把en\_US.UTF-8 UTF-8 zh\_CN.UTF-8 UTF-8 的注释取消，然后保存，退出 ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180506010143.png) 接着输入 `locale-gen` ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180506010245.png) `echo LANG=en_US.UTF-8 > /etc/locale.conf` ` ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime` ` hwclock --systohc --utc` ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180506010457.png) 接着就可以安装引导了

###       UEFI引导

`pacman -S dosfstools grub efibootmgr` `grub-install --target=x86_64-efi --efi-directory=/boot/EFI --recheck` ` grub-mkconfig -o /boot/grub/grub.cfg` _// 生成GRUB菜单_

### ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180506111130.png)

### MBR引导

 `pacman -S grub os-prober` ` grub-install --recheck /dev/sda` **// 注意:sda不带分区号!** ` grub-mkconfig -o /boot/grub/grub.cfg` _// 生成GRUB菜单_

### 修改root密码

`passwd` ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180506112013.png)

### 添加用户

 `useradd -m -g users -s /bin/bash 用户名` ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180506111950.png)

### 修改添加的用户的密码

`passwd 用户名` ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180506112031.png)

### 赋予添加用户sudo权限

 `nano /etc/sudoers` // **root ALL=(ALL) ALL 下面添加 用户名 ALL=(ALL) ALL 保存退出** ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180506111928.png)

### 最后，退出重启即可

`exit` `reboot` ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/05/TIM截图20180506115133.png)

_其他_

关于图形化的教程我就不赘述了，可以去网上查找其他资料

**各种美化后的效果图**

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/02/Screenshot-from-2018-02-10-20-26-43.png)
