---
title: 使用七牛云对象存储功能加速WordPress站
tags: []
id: '199'
categories:
  - - 技术相关
date: 2017-05-07 17:45:41
---

**_起因_**

今天我拿网站出去装B的时候，有人反应根本打不开，我觉得他们![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/240d82b5cc0483b9.jpg)，我怎么能好好访问呢。后来想想我服务器是国外的，而我![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/QQ截图20170507170542.png)全程FQ![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/TX4WO77_DT8SX1J3_YD.jpg)，看来并不是他们的问题![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/8MM7WXFSKR1YKC8.gif)，然后我就有了使用七牛云对象储存功能给国内访问博客加速的想法

**_开始_**

*   当然是先去[七牛云官网](https://www.qiniu.com/)注册使用，注册就不多说了
*   创建一个对象储存空间，名称自己写，区域自己选，然后就创建吧![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/G0AAGUQO6A34I7.png)
*   得到测试域名，下面要用到![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/LJ0Q_6LH1H6V_@UZO.png)
*   写上镜像源，选择镜像储存，格式http(s)://xxxx.xxx/  robots.txt不使用默认，[robots.txt下载](https://cloud.wordpress-1253676827.file.myqcloud.com/robots.txt)![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/1RNCUZD61_TVO1IPFLE.png)
*   回到博客后台，安装七牛云插件，两个都要安装，先装WPJAM Basic![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/N10PV@ZXOBL@NFOD8PK.png)
*   设置插件，七牛域名就是第三步得到的测试域名，前面要加上http://    七牛空间名就是第二步你写的空间名 ACCESS KRY和SECRET KEY下一步说                                          ![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/TMSOST3@QRMH7Q8Y.png)
*   获取ACCESS KEY和SECRET KEY    打开个人面板-密匙管理，然后创建密匙 复制粘贴到上一步就好啦                                                                                                                                                 ![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/H8@5SJ56M_LMZ7YSIV2.png)                                  ![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/OWS22_PQS9R1U6CQ9SPO1.png)
*   保存设置就可以用了，我是国外服务器，加速明显还省流量![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/OCX0S72UA31@CWGZLSE49.jpg)

_**我遇到的问题**_

高高高兴弄完准备测试的时候，打开网站![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/8_Q91CRCI6_AVHUGTG1.png)丫的卡着啊![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/TEJ622NMRFOYJSP5Q7R.jpg)，后来发现问题，把![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/26911VWEX8KASWPYJX1.png)js删掉就好啦![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/0NFPC3GTVEI32KOI.gif)。我用的akina主题 很不错 到这，你是否感觉访问本站快些了呢，快说是 ![](http://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2017/05/CCBG1WK8P7R80ML9PP.gif)
