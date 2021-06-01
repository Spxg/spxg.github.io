---
title: Ubuntu 14.04下安装SS及Proxychains
tags: []
id: '63'
categories:
  - - 技术相关
date: 2017-04-03 13:58:26
---

_很多机友因为要同步源码而去访问那些不存在的网站，所以我写这个FQ教程_

**安装SS教程**

*   建议直接安装shadowsocks-qt5，原来的方法不支持某些协议和加密方式
*   设置 PPA 源并安装 shadowsocks-qt5 sudo add-apt-repository ppa:hzwhuang/ss-qt5
*   更新源 sudo apt-get update
*   安装shadowsocks-qt5 sudo apt-get install shadowsocks-qt5

\*如果遇到依赖问题，输入 sudo apt-get -f install libappindicator1 libindicator7 接下来就是代理设置了，如果使用qt5的话，跳过文章中间的SS安装旧的教程，到文章下面看Proxychains安装配置教程

**旧的安装SS教程**

*   关于Ububtu 16.04 安装SS似乎可以直接 sudo apt-get install shadowsocks
*   国际惯例，先更新一下源，输入命令 sudo apt-get update

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/04/201607251469460824414830.png)

*   输入命令 sudo apt-get install python-gevent python-pip

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/04/201607251469461037102621.png)

*   安装SS程序 输入命令 sudo pip install shadowsocks

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/04/201607251469461120150276.png)

*   配置SS 输入命令生成配置文件 gedit ss.json

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/04/201607251469461210126751.png)

*   添加配置 { “server”:”服务器的ip”, “server\_port”:服务器端口, “local\_address”:”127.0.0.1″, “local\_port”:1080, “password”:”密码”, “timeout”:300, “method”:”aes-256-cfb”, “fast\_open”:false }
*   注意：method是加密方法，这里默认是aes-256-cfb，不一样的请自行修改

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/04/201607251469461434173767.png) 然后保存 \* 运行SS 输入命令 sslocal -c ss.json ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/04/201607251469461621557248.png) 到这SS客户端就安装和配置完了，然而现在还需要代理，可以设置系统全局代理，但是又只想让一个应用翻墙，比如同步源码。所以现在需要Proxychains

**Proxychains安装教程**

*   输入命令安装Proxychains sudo apt-get install proxychains

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/04/201607251469461834467776.png)

*   进行配置 输入命令 sudo gedit /etc/proxychains.conf

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/04/201607251469461913162769.png)

*   在最后，把socks4改成socks5，后面的端口改成你的SS的本地端口，我这写1080

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/04/201607251469462067144115.png) 然后就完成了Proxychains的安装与配置 \* 最后在SS运行的情况下运行Proxychains，在命令前加上proxychains就行，比如输入 proxychains firefox ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/04/201607261469462402126322.png)

*   然后进Google测试一下

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/04/201607251469462218621332.png)

*   同步源码输入 proxychains repo sync

# 联系方式

*   spxg0923@gmail.com
