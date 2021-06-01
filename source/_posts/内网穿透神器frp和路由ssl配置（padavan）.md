---
title: 内网穿透神器frp和路由SSL及Aria2远程下载配置（Padavan）
tags: []
id: '256'
categories:
  - - 技术相关
date: 2017-08-06 21:48:08
---

# 前言(路由公网IP请只看SSL获取）

[frp](https://github.com/fatedier/frp)是优秀的内网穿透工具，配置起来简单，这里主要说的是ssh和web的配置，其他的在官方说明文档有 SSL的获取可以看逗比根据地的 [免费申请SSL证书教程](https://doub.io/wlzy-28/)

## 1.获取[frp](https://github.com/fatedier/frp/releases)

我这里使用Linux 64位系统

```
wget https://github.com/fatedier/frp/releases/download/v0.13.0/frp_0.13.0_linux_amd64.tar.gz
```

之后再解压

```
tar xzvf frp_0.13.0_linux_amd64.tar.gz
```

## 2.开始配置

进入目录

```
cd frp_0.13.0_linux_amd64
```

目录中有frps.ini frpc.ini等文件，frps是服务端用的，也就是有公网ip的机器；frpc.ini是客户端用的，也就是内网的机器。 如果仅仅使用ssh功能，服务端就不需要配置了，先用screen,免得关闭服务端的ssh后，frp断线

```
screen -S frp
```

之后再启用服务端

```
./frps -c frps.ini
```

没报错啥的就好了

*   开始客户端的配置 同样的下载解压步骤就不重复了 打开配置文件frpc.ini

```
nano frpc.ini
```

然后将

```
server_addr =
```

后面改成自己服务器的IP地址，然后运行（至于Padavan，只要在相应界面改一下配置启动就行）

```
./frpc -c frpc.ini
```

最后用xshell vssh jucessh等ssh软件就可访问，_这里ssh默认为6000_ ,ubuntu啥的需要装openssh，arch linux请看[Arch\_wiki](https://wiki.archlinux.org/index.php/Secure_Shell_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

## 3.Web界面的穿透，这里主要讲https

*   首先在Padavan>系统管理>服务 中开启https
*   然后在下方![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/08/A0C4EB4B-6498-4DD4-ADFF-6E38AD0FAFAB.jpg)，分别填上crt文件和key文件的内容
*   之后开始配置服务端frps.ini

```
nano frps.ini
```

下方输入

```
vhost_http_port = 80
vhost_https_port = 443
```

如果端口有冲突，换一个。保存，运行。这里用了http和https，http用来aria下载，https用来Web岂不美滋滋 然后就是客户端的配置了 在Padavan>花生壳内网穿透>frp中添加内容

```
[web01]
# https web
type = https
local_ip = 192.168.123.1
local_port = 443
custom_domains = xxxxxx.xxx

[web02]
# aria
type = http
local_ip = 192.168.123.1
local_port = 6800
custom_domains = xxxxxx.xxx
```

这里custom\_domains是域名，需要将域名解析到服务端 保存，运行即可

## 效果展示

*   Web ![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/08/QQ20170806-220521.png)
*   Aria2远程下载 ![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/08/QQ20170806-220554.png)