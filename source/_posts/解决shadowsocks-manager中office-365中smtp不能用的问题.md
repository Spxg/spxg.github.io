---
title: 解决Shadowsocks-Manager中Office 365中SMTP不能用的问题
tags: []
id: '378'
categories:
  - - 技术相关
date: 2018-08-15 16:05:10
---

_最近换了个服务器，想体验更好的科学上网服务，于是就打算搭建webgui，项目地址[Shadowsocks-Manager](https://github.com/shadowsocks/shadowsocks-manager)_

       安装过程一波三折，好在都解决了，最让我头疼的是邮件一直都无法使用，我是Office365的smtp，起初以为是vps的问题，后来发现用Gmail可以使用，博客的Office365也正常

*   首先我尝试添加端口号587，后来也算是有所进展，毕竟成功通信了，却无法登陆  ，报错如下图

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/08/TIM截图20180815155551.png)![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/08/捕获.png)

_经过查询资料，发现了[node-smtp-client](https://www.npmjs.com/package/node-smtp-client)的用法_

*   原来这个程序还有个secure的用法，默认这个参数为true，据我了解，office365 smtp是没有SSL加密的，故将true改成false即可，成功解决问题

 

解决办法

在yml的email选项中添加如下代码即可

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2018/08/TIM截图20180815162116.png)
