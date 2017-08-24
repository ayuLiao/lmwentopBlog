---
title: 搭建自己的wordpress博客
date: 2017-05-31 11:06:20
tags: wordpress
---

## 简介
我本来就有一个博客，[lmwen.top](lmwen.top)，这个博客是一个静态界面，搭建在github上，每个github用户都可以在github page上搭建一个静态界面，很多人将它弄为自己的博客，挺geek的。

静态界面的优势就是在于加载速度快，放在github page上的好处是，界面怎么布局随你搞，不会想在CSND博客上固定的模板，而且还有很多CSND上的广告，虽然在上面创建一个博客只需要注册一个账号则可，但是限制还是比较大的，对应爱自由的我们应该选择一个更好的。

如果搭建[lmwen.top](lmwen.top)这样的博客，我已经写过了，有兴趣的可以看[hexo+github给自己搭建一个博客空间](http://lmwen.top/2016/06/09/hexo-github%E7%BB%99%E8%87%AA%E5%B7%B1%E6%90%AD%E5%BB%BA%E4%B8%80%E4%B8%AA%E5%8D%9A%E5%AE%A2%E7%A9%BA%E9%97%B4/)

这次我打算使用wordpress来搭建一个英文博客，所以使用的东西都是针对国外的。wordpress是一款CMS，简单的理解就是内容管理后台，通过全世界的努力，它已经比较完善了，它最强大的优势在于庞大的插件和主题库，你想要什么功能，一般都有插件将其实现了，你根本不必再编写代码，你想要怎么样的布局，基本上都可以在主题库中找到自己想要的效果，而且很酷很专业

本wordpress搭建的环境
使用lnmp在搬瓦工vps上搭建wordpress，系统是centos7，在Godabby上购买域名并进行域名解析

## 环境介绍
可能不是所有人都知道lnmp、搬瓦工和Godabby，下面来简单介绍一下

### lnmp
lnmp其实就是一键式集成环境，l代表Linux，n代表Nginx，m代表MySQL，p代表PHP
其实我并不怎么喜欢集成环境，更喜欢自己逐个安装Nginx、MySQL和PHP，应该在centos7上安装这些都很简单，当然在centos7把MySQL从官方源中去除了，变成了Mariadb，其实这两个没差，想深入了解一下可以看我另外一篇博文[CentOS7搭建配置Nginx+PHP+MySQL](http://lmwen.top/2017/03/15/CentOS7%E6%90%AD%E5%BB%BA%E9%85%8D%E7%BD%AENginx-PHP-MySQL/)，我之所以不喜欢集成环境不是因为它不好，有时它封装的很好，而我又没有详细的看过它的官方文档，导致要修改配置文件的时候，不知道集成环境将相应的配置文件放到哪里去了，很尴尬，而且很多优秀的集成环境封装的太好，让我们使用起来十分方便，但是高度封装导致的问题就是用户如果没有深入的了解该集成环境就很难比较细致的控制他（控制粒度比较大）

但是这里我建议使用lnmp是因为它对wordpress很友好，如果你分别安装Nginx、MySQL和PHP后，然后自己搭建wordpress，这也是可行的，但是会出现一些让认头痛的问题，wordpress的路径只能使用最原始的解析，不然Nginx会报错，你需要通过比较繁杂的配置让Nginx可以正常的解析wordpress的路径

![Nginx路径报错.png](http://obfs4iize.bkt.clouddn.com/Nginx%E8%B7%AF%E5%BE%84%E6%8A%A5%E9%94%99.png)

如果使用Apache是完全没有问题的，如果想继续使用Nginx，要没配置Nginx（推荐），要么修改wordpress路径，下面展示修改wordpress路径的方法，如果要配置Nginx比较复杂，本文就不介绍了，因为我也没试过配置Nginx

![朴素url.png](http://obfs4iize.bkt.clouddn.com/%E6%9C%B4%E7%B4%A0url.png)

然而你使用lnmp，它已经帮你搞定这个头痛的问题了

### 搬瓦工
如果你有动念头搭建自己的VPN（这个后面有机会可以写一篇博文），你就应该听过搬瓦工，搬瓦工是一个知名的vps商家，出售各种vps，应该是国外商家，而且支持支付宝（很多国外商家并不支持支付宝），所以很多国人使用，同时因为搬瓦工有一键搭建梯子的服务，所以购买一个搬瓦工，自己搭建翻墙服务器非常简单，我使用搬瓦工最不爽的是，通过xshell连接上搬瓦工服务器后，敲命令的反应速度真是慢啊，每个命令字符几乎都要等1秒，把我的梯子换了多个线路还是不行，最终我认怂，就这样用着先，一开始还担心搭建的wordpress反应速度太慢，但是搭建成功，发现自己的担忧是多余的

在vps这个圈子，其实有很多玩法，很多圈内次，我也接触不深，就大概知道一些，但是感觉这行的黄牛做的真是风生水起，想了解跟多vps可以看看这个博客，职业搞基，感觉也是一个职业vps黄牛，[vps大全](http://www.vpsdaquan.cn)

### GoDabby
其实就是一个外国的域名商，其实网上有些时候会有所谓的免费域名和很低价钱的虚拟空间，但大多数时候都是一个坑。

我之所以选择GoDabby，而不选择万网啊，新网啊，西部数据之类的国内知名域名商户是因为懒的备案，我西部数据还有12个域名，都是当时脑子一热觉得是好域名买下来等着倒卖的，结果显示留在西部数据的硬盘里吃灰。

在GoDabby购买域名一个是因为它也支持支付宝，而且GoDabby在全世界也是很有名气的域名商家，同时因为做英文网站，针对外国用户，如果是通过国内商家购买，做域名解析可能会影响到国外流量访问网站的速度（这个只是个人猜测觉得会有影响），所以就直接使用GoDabby了，GoDabby对比国内的域名商家的一个特定就是，真他么贵，以com结尾的域名要70元，国内是50多元，但是我还是忍痛买了，反正自己迟早也会富成废人，所以没必要在乎小钱！

## 环境搭建
这里我认为你已经购买了搬瓦工的vps，并通过ssh工具连接上了，这里我使用的是xshell，购买细节，搬瓦工后台设置等等内容可以去找其他教程。

通过xshell连接上后，就可以安装lnmp了，安装细节官网已经讲的比较详细了 [lnmp官网安装教程](https://lnmp.org/install.html)，这里我再提一下要注意的

首先，通过wordpress搭建一个网站，一般都会选用占内存最小的系统，这里就是最小版的centos，里面很多东西都没有，因为我们用不到，搭建网站有4个点最重要：**内容、布局、安全、响应速度**，系统占的内存小那么wordpress就有跟多可用内存，这样网站响应的速度相应会快一些，如果想要跟快，除了花多点钱换台牛逼的服务器还有跟办法就是使用CDN服务，压缩自己的图片文件，合并css和js减少网页的请求等等

因为一般使用的是最小版的系统，所以先安装screen和vim（vim可以不安装，但是相对vi我更喜欢vim，各位随意），Screen是一款由GNU计划开发的用于命令行终端切换的自由软件。用户可以通过该软件同时连接多个本地或远程的命令行会话，并在其间自由切换。目的就在于，避免我们SSH连接远程Linux服务器时要执行长时间任务，如果我们连接Linux的会话窗口关闭了，那么执行远程耗时任务的线程也被关闭了，下次又要重头开始，非常麻烦，所以使用screen开启一个新的会话，当我们连接断了，该会话还在

只要Screen本身没有终止，在其内部运行的会话都可以恢复。远程登录的用户即使网络连接中断，用户也不会失去对已经打开的命令行会话的控制。只要再次登录到主机上执行screen -r就可以恢复会话的运行。同样在暂时离开的时候，也可以执行分离命令detach，在保证里面的程序正常运行的情况下让Screen挂起（切换到后台）
更多screen命令的内容，可以看这篇博文[linux screen 命令详解](http://www.cnblogs.com/mchina/archive/2013/01/30/2880680.html)

安装好screen后，执行
```
screen -S lnmp
开启一个新的会话窗口
```
然后你可能还需要安装wget，命令如下
```
yum install -y wget
```

然后通过wget来下载稳定版的LNMP一键安装包则可
```
wget -c http://soft.vpser.net/lnmp/lnmp1.3-full.tar.gz && tar zxf lnmp1.3-full.tar.gz && cd lnmp1.3-full && ./install.sh lnmp
```

其他安装细节看这[lnmp官网安装教程](https://lnmp.org/install.html)

安装成功后会出现下面的界面
![lnmp安装成功.png](http://obfs4iize.bkt.clouddn.com/lnmp%E5%AE%89%E8%A3%85%E6%88%90%E5%8A%9F.png)

## 添加虚拟机
当你成功安装lnmp后，你就可以通过lnmp来添加虚拟主机了，这里的虚拟主机其实就是一个网站的根目录，当然其中lnmp为我们做了一些控制。一个虚拟主机对应一个网站，当我们创建完虚拟主机后，就可以将网站相关的代码上传上来，聪明的你应该知道了，lnmp很方便让你一个主机有多个网站

可以通过如下命令来添加虚拟主机
```
lnmp vhost add
```
然后会出现交互界面，让你配置一些信息

1.首先会让你输入域名，你输入自己注册的域名，如果还没注册，可以在这里停一些，将域名注册好再回到这里继续操作，当然这些配置操作都会写入相应的配置文件，所以后面是可以修改的，不用过于谨慎，通过与这些界面进行交互，形成的配置文件会在这里
```
/usr/local/nginx/conf/vhost/域名.conf
```

当你输入网域名后，它还会提示你是否添加其他域名，我这里选择n，不添加其他域名，然后通过修改配置文件，利用301重定向将不带www的域名指到带www的域名，如新网站的域名是www.goenjoychina.com，如果不配置一下，goenjoychina.com就访问不到，这样当然不好

2.接着会让你输入网站的根路径，这里用默认的就好

3.然后询问是运行Rewrite，这里最好写y，有些人可能不明白什么是Rewrite，这里的Rewrite表示的其实是URL Rewrite，也就是URL重写，作用就是将传入web的某个请求重定向到另一个URL，URL Rewrite最常用的用法就是URL伪静态，这个我在很多PHP框架中都见过，如ThinkPHP，它可以将动态页面显示未静态界面，wordpress也会使用这种技术

进一步了解[URL Rewrite](http://blog.csdn.net/shimiso/article/details/8594885)

4.再下来就会问你是否开启访问日志，这个自己看情况考虑，有时差一些错误可以看log来解决，但是硬盘空间不是非常大的服务器就不建议开了，因为大多数时候是没必要的

5.接着就是创建数据库了，这里要注意是Backspace是不会删去命令行中的字符的，所以写错了就从头配吧，感觉比数据库找回密码快很多

6.到这里，就完成了虚拟机的创建了

可以在/home/wwwroot/目录下看见你创建的虚拟主机

## 安装wordpress
在安装wordpress前，建议将default文件中的一些敏感文件给删除，default这个文件夹是系统默认的，里面有一些信息，如phpmyadmin和探针（p.php），为了安全，建议上传这些东西，如p.php、phpinfo.php、index.html等等，然后将phpmyadmin文件夹改名，给一个不容易被猜到的名称，不然你的数据库被人爆了，你网站就费了，数据备份很重要

![删除default内容.png](http://obfs4iize.bkt.clouddn.com/%E5%88%A0%E9%99%A4default%E5%86%85%E5%AE%B9.png)

修改完成后，我们进入刚刚创建的虚拟主机目录，通过ftp将将wordpress传到该目录下，建议传压缩包，这样传输速度回快一些，然后再在vps上进行解压则可，这里要注意的是，**wordpress的文件要直接放到该虚拟主机的目录下**，不然会报403错误

![vps根目录.png](http://obfs4iize.bkt.clouddn.com/vps%E6%A0%B9%E7%9B%AE%E5%BD%95.png)

然后就是按部就班的安装wordpress了

## 购买域名
这里没什么好说的，先购买域名是因为后面的步骤需要域名，我在GoDaddy上购买的，在这里购买的域名是免备案的，方便些，虽然贵一点
![微信截图_20170531095807.png](http://obfs4iize.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20170531095807.png)

然后在GoDaddy设置一下域名解析，将该域名解析到搬瓦工VPS对应的IP则可

## 安装wordpress
此时通过域名进入wordpress的安装界面，然后一步步的安装，安装完后，就会出现一个界面，**将后台密码和用户名设复杂一点**，如果只是单纯的设置复杂点，其实还有点欠缺，因为可以通过脚本来跑字典，虽然比较难跑到你的密码，但是让你的服务器连接数据库查询登录密码是挺消耗资源的，这里一个比较好的方法就是通过Google验证码机制将恶意登录用户挡在门外，这样没有输入正确的验证码，就无法登录，虽然效果似乎一样，但是省了服务器连接数据库查询用户输入是否正确的过程

Google验证码是目前我知道的最难搞的验证码，因为人有时都会弄错，感觉除了极其牛逼的人，很少有人能弄

要使用Google验证码，首先要在[Google reCAPTCHA](https://www.google.com/recaptcha/intro/invisible.html)将你的网站注册，要使用这个，你就需要Google账号，这个就不多说了

在Google reCAPTCHA为自己的网站申请相应的密钥
![添加验证码.png](http://obfs4iize.bkt.clouddn.com/%E6%B7%BB%E5%8A%A0%E9%AA%8C%E8%AF%81%E7%A0%81.png)

然后登录wordpress的后台，添加Google Captcha这个插件，该插件可以轻松的让你的wordpress网站使用Google验证码机制，你可以通过设置该插件的一些参数，让Google验证机制出现在不同的界面，如评论界面、登录界面，这样就可以减少恶意用户对你的网站造成的伤害
![google验证.png](http://obfs4iize.bkt.clouddn.com/google%E9%AA%8C%E8%AF%81.png)

如果想看一步步申请Google验证码的文章，可以看[教你如何使用Google的reCAPTCHA驗證碼keys申請](http://zfly9.blogspot.com/2015/07/20150703a.html)

## 一些问题
### 插件无法直接下载安装
因为你将wordpress搭建在centos上，所以就会有文件目录权限造成的问题，你需要给虚拟主机目录下的文件开启写权限，不然你无法直接通过wordpress后台在线安装插件，只能通过本地上传，就算你每次都通过本地上传再使用插件，有些优秀的插件也因为目录没有写权限而报错，我这里为www.goenjoychina.com目录下的所有文件都开放了写权限
```
chown www:www -R /home/wwwroot/www.goenjoychina.com/*
```

### goenjoychina.com无法访问
如果你没有修改过配置文件，goenjoychina.com是无法直接访问的，因为虚拟主机没有绑定goenjoychina.com这个域名，你可以通过修改该虚拟主机对应的conf文件来实现goenjoychina.com重定向到www.goenjoychina.com的功能

lnmp集成环境下，配置文件默认位于
/usr/local/nginx/conf/vhost/ 目录下

你可以通过vim把新的内容修改一下

将
```
server_name www.goenjoychina.com;
```
改为
```
server_name www.goenjoychina.com goenjoychina.com;
if ($host != 'www.goenjoychina.com' ) { 
rewrite ^/(.*)$ http://www.goenjoychina.com/$1 permanent;}
```

然后重启一下nginx就好了，在lnmp集成环境下，重启nginx的方法如下
```
/etc/init.d/nginx reload
```

这里贴一下官方控制lnmp状态的命令
```
LNMP状态管理命令：

LNMP 1.2+状态管理: lnmp {start|stop|reload|restart|kill|status}
LNMP 1.2+各个程序状态管理: lnmp {nginx|mysql|mariadb|php-fpm|pureftpd} {start|stop|reload|restart|kill|status}
LNMP 1.1状态管理： /root/lnmp {start|stop|reload|restart|kill|status}
Nginx状态管理：/etc/init.d/nginx {start|stop|reload|restart}
MySQL状态管理：/etc/init.d/mysql {start|stop|restart|reload|force-reload|status}
Memcached状态管理：/etc/init.d/memcached {start|stop|restart}
PHP-FPM状态管理：/etc/init.d/php-fpm {start|stop|quit|restart|reload|logrotate}
PureFTPd状态管理： /etc/init.d/pureftpd {start|stop|restart|kill|status}
ProFTPd状态管理： /etc/init.d/proftpd {start|stop|restart|reload}
Redis状态管理： /etc/init.d/redis {start|stop|restart|kill}

如重启LNMP，1.2+输入命令：lnmp restart 即可；单独重启mysql：/etc/init.d/mysql restart 也可以 lnmp mysql restart ，两个是一样的。

LNMPA状态管理命令：

LNMPA 1.2+状态管理: lnmp {start|stop|reload|restart|kill|status}
LNMPA 1.2+各个程序状态管理: lnmp {httpd|mysql|mariadb|pureftpd} {start|stop|reload|restart|kill|status}
LNMPA1.1状态管理： /root/lnmpa {start|stop|reload|restart|kill|status}
Nginx状态管理：/etc/init.d/nginx {start|stop|reload|restart}
MySQL状态管理：/etc/init.d/mysql {start|stop|restart|reload|force-reload|status}
Memcached状态管理：/etc/init.d/memcached {start|stop|restart}
PureFTPd状态管理： /etc/init.d/pureftpd {start|stop|restart|kill|status}
ProFTPd状态管理： /etc/init.d/proftpd {start|stop|restart|reload}
Apache状态管理：/etc/init.d/httpd {start|stop|restart|graceful|graceful-stop|configtest|status}

LAMP状态管理命令：

LAMP 1.2+状态管理: lnmp {start|stop|reload|restart|kill|status}
LAMP 1.2+各个程序状态管理: lnmp {httpd|mysql|mariadb|pureftpd} {start|stop|reload|restart|kill|status}
```
原内容在这里，[LNMP状态管理命令](https://lnmp.org/tag/%E9%87%8D%E5%90%AF/)

### wordpress只有一个主题
刚安装完lnmp，我就发现wordpress只有一个主题，我通过ftp上传上去的新主题都没有显示，不明原因

![只显示一个主题2.png](http://obfs4iize.bkt.clouddn.com/%E5%8F%AA%E6%98%BE%E7%A4%BA%E4%B8%80%E4%B8%AA%E4%B8%BB%E9%A2%982.png)

查了一下资料，方向lnmp将scandir方法禁用了，认为是危险函数，但是要显示多主题就必须要使用该函数，你可以修改/usr/local/php/etc下的php.ini，删去scandir就ok了

![只显示一个主题.png](http://obfs4iize.bkt.clouddn.com/%E5%8F%AA%E6%98%BE%E7%A4%BA%E4%B8%80%E4%B8%AA%E4%B8%BB%E9%A2%98.png)

重启一下php-fpm，就可以了
```
/etc/init.d/php-fpm restart
```

此时显示出多主题的样子
![多主题了.png](http://obfs4iize.bkt.clouddn.com/%E5%A4%9A%E4%B8%BB%E9%A2%98%E4%BA%86.png)

## 结尾
到这里，一个wordpress网站就被搭建起来，但是也仅仅是搭建起来了，网站要搞起来，还是前面提过的4个重点：**内容、布局、安全、速度**，每个方面都可以讲的很深，我也刚刚入了个门，大家共勉！

用代码创造乐趣---ayuliao