---
title: CentOS7搭建配置Nginx+PHP+MySQL
date: 2017-03-15 10:59:13
tags: Linux
---

## 简介
本章内容教大家在CentOS7上分别安装上Nginx、PHP和MySQL，熟悉安装的过程和一些基本的配置，理解当用户进行PHP请求时，Nginx做了什么，对Nginx响应用户请求的流程有个整体把握

## Nginx与Apache的区别

Nginx和Apache最核心的区别是：Apache是同步多进程模型，一个连接对应一个进程，所以在高并发情况下，Apache消耗的资源比较多，而Nginx是异步多进程模型，多个连接可以对应一个进程，在Nginx中一个进程可以处理万级别的连接数

### Nginx相对Apache的优势
1.比Apache占用的内存和资源更少（轻量级）
2.Nginx处理请求是异步非阻塞的，所以适合高并发的情景，而Apache处理请求是阻塞的，在高并发的情景下Nginx能保持低资源低消耗却保证了较高的性能
3.高度模块化，可以比较轻松的编写并使用模块
4.Nginx配置简洁

### Apache相对Nginx的优势
1.模块非常多，已经可以满足大部分的需求了
2.稳，非常稳，稳如泰山（超稳定）
3.Apache的rewrite比Nginx的强大（rewrite主要功能是实现URL的跳转）
4.发展时间旧，bug相对Nginx要少

现在一般前端使用Nginx作为反向代理，用于抗压（高并发情况），而后端使用Apache处理动态请求

如果想更深一步去理解，可以看下这篇文章
[Nginx为什么比Apache Httpd高效：原理篇](http://www.mamicode.com/info-detail-1156329.html)

## 安装Nginx
为了保证正常安装Nginx和安装后Nginx的正常使用，在安装前，需要安装如下软件
1.GCC编译器
GCC（GNU Compiler Collection）可用于编译C/C++语言程序，我们下载的Nginx不是可以直接在Linux中可以运行的二进制程序（虽然1.2.x版本已经在某些操作系统上提供相应的二进制包了，但是这里学习最广泛使用的方式，将源码编译成二进制程序进行使用）

通过下面两句命令来安装GCC编译器
```Linux
yum install -y gcc

yum install -y gcc-c++
```

2.PCRE库
PCRE（Perl Compatible Regular Expressions，Perl兼容正则表达式）是由Philip Hazel开发的函数库，这个库支持正则表达式。如果我们在配置文件nginx.conf中使用了正则表达式，就必须装上PCRE库，因为Nginx的HTTP模块需要靠PCRE库来解析正则表达式，如果你认定自己不会使用正则表达式，那么可以不装，这里建议装上，以防以后需要使用到正则表达式

命令如下
```
yum install -y pcre pcre-devel
pcre-devel是使用PCRE做二次开发时所需要的开发库，包括头文件等，这也是编译Nginx所必须使用的。
```

3.zlib库
zlib库用于对HTTP包的内容做gzip格式的压缩，来减少网络流量传送，当我们在nginx.conf中配置了gzip on，并指定了某些类型（content-type）的HTTP响应使用gzip来进行压缩以减少网络传输量，就必须在编译时将zlib库编译进Nginx

命令如下
```
yum install -y zlib zlib-devel
zlib是直接使用的库，zlib-devel是二次开发所需要的库。
```

4.OpenSSL开发库
如果想让Nginx在SSL协议上传送HTTP，也就是所谓的HTTPS，就需要装上OpenSSL，此外，如果我们还想使用MD5、SHA1等散列函数，那也必须安装上这个开发库

命令如下
```
yum install -y openssl openssl-devel
```

上面的4个库只是Nginx实现Web服务器最基本功能所需要的库。

Nginx是高度自由化的Web服务器，它的功能有许多模块来支持，这些模块可以根据我们自身的需求来选择使用，此时要注意的是使用的模块是否需要安装其他第三发库来支持，类似于zlib库或OpenSSL库，如果需要，就必须先将这些第三方库给安装好，才可以正常使用该模块

准备工作做了这么久，接下来正式进行Nginx的安装

你可以去官方下载最新版的nginx的tar包，或者直接使用wget命令来下载，这里我选择我常用的版本作为演示，其他版本的安装十分雷同

获得并解压Nginx对应的tar包
```
获得Nginx对应的tar包
wget http://nginx.org/download/nginx-1.11.9.tar.gz

解压tar包
tar zxvf nginx-1.11.9.tar.gz
```
进入解压好的nginx-1.11.9文件夹
![解压tar包](http://obfs4iize.bkt.clouddn.com/%E8%A7%A3%E5%8E%8Btar%E5%8C%85.png)

运行configure文件，对nginx进行编译前的配置
```
sudo ./configure --with-http_ssl_module       添加SSL支持模块,HTTPS必须的
```

最后通过make命令进行编译和安装
```
make && make install
```
这样就完成了Nginx的安装了

但是因为不是直接通过yum命令安装的（yum源中没有Nginx），所以启动Nginx的命令比较麻烦，要进入到对应的文件夹中其中Nginx，这里Nginx安装了默认的路径中，如果你不知自己装在哪个路径中，可以看一下刚通过make install函数安装时命令行中的内容，在最后安装成功后，会显示Nginx安装的具体路径
![nginx](http://obfs4iize.bkt.clouddn.com/nginx.png)

```
进入nginx/sbin目录启动nginx
cd /usr/local/nginx/sbin
启动nginx
./nginx
```

Nginx中其他简单的操作也类似
```
cd /usr/local/nginx/sbin
重新载入配置
./nginx -s reload
重新启动Nginx
./nginx -s reopen
停止Nginx
./nginx -s stop
```

启动完Nginx后，你就可以通过主机IP来访问nginx了，出现下面这个界面表示Nginx就搞定了（我对默认界面进行了一些修改），如果不能访问，可以查询一下自己防火墙的配置，是否开放了Web服务所需要的端口，如果你的服务器上在腾讯云、阿里云或者某个云上的，可以看看自己为该服务器配置的安全策略是否允许Web服务所需要的端口可以接收和发送相应的数据

可以看一下CentOS7中，配置防火墙的命令
```
允许相应的端口
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --zone=public --add-port=9000/tcp --permanent

重新载入配置
firewall-cmd --reload
```

![访问nginx.png](http://obfs4iize.bkt.clouddn.com/%E8%AE%BF%E9%97%AEnginx.png)

但是每次要进行到相应的路径来启动Nginx太麻烦了，这里可以在etc/init.d目录下创建一个启动脚本，通过这个脚本来启动Nginx，这样启动Nginx会方便很多

```
在etc/init.d目录下创建nginx脚本
vim /etc/init.d/nginx
```

然后将下面内容复制并保存到这个nginx脚本文件中
```
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemin
#
# chkconfig:   - 85 15
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /usr/local/nginx/conf/nginx.conf
# pidfile:     /usr/local/nginx/logs/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/usr/local/nginx/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"

lockfile=/var/lock/subsys/nginx

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```

接着为这个脚本添加执行权限
```
chmod +x /etc/init.d/nginx
```

并将Nginx添加到系统自启动组中
```
chkconfig --add /etc/init.d/nginx
```

最后将开启自启动
```
chkconfig nginx on  
```

经过上面的配置后，就可以通过下面的命令来操作nginx了
```
service nginx reload	重新加载配置

service nginx start		启动Nginx
```

## 安装MariaDB (MySQL)
很多比较旧的教程中使用如下3句命令来安装MySQL
```
yum install mysql
yum install mysql-server
yum install mysql-devel
```
但尴尬的是，mysql和mysql-devel都安装成功了，mysql-server却无法通过yum安装，造成这个现象的原因就是在CentOS7中，mariaDB取代了MySQL，CentOS7中yum命令的源中是没有mysql-server的

### MySQL和mariaDB区别
mariaDB其实是MySQL源代码的一个分支，原本MySQL是一个开源的数据库，但是在2009年4月20日，Oracle公司收购了SUN公司，MySQL也从SUN公司转手到Oracle公司，那么MySQL以后是否继续开源变成了一个未知数，在意识到Oracle公司可能会对MySQL的许可做什么后， mariaDB从MySQL中分离了出来，mariaDB作为MySQL的"向下替代品"，mariaDB包含了一些优于MySQL的新特性，所以以后看见要求安装MySQL作为环境要求的，其实可以安装mariaDB，一般不会出现错误，所以下面我们直接安装mariaDB

通过下面命令来安装mariaDB
```
yum -y install mariadb*  
```

通过下面命令来启动mariadb服务
```
systemctl start mariadb.service  
systemctl enable mariadb.service
```

初次安装mariaDB和MySQL情况是一样的，root用户没有密码（MySQL与mariaDB命令配置几乎没有差别）
所以可以通过下面命令为root用户配置密码
```
mysqladmin -u root password "你的密码"
```

也可以在进入MariaDB后，通过修改mysql表来配置密码
```
MariaDB [mysql]> update user set password=password("你的密码")where user='root';
```

我在写这篇博文前已经配置好我的MariaDB，但是我忘记密码，我这就叫贵人多忘事，这里顺便提一下MariaDB的root用户忘记密码的解决方法，该方法同样适用于MySQL

首先，你要确定服务器中数据库是安全的，没有人可以随意连接上这个数据库，因为在重置MariaDB的root密码时，MariaDB数据库完全处于没有密码保护状态，如果此时其他用户也可以随意访问数据库并修改数据库中的信息（此时任何人登录数据库都是不需要密码的）

确认环境是安全的，就修改etc目录下的my.cnf配置文件，在[mysqld]的段中添加上下面这句代码
```
skip-grant-tables
```

具体操作命令如下
```
vim /etc/my.cnf
```
![配置mycnf.png](http://obfs4iize.bkt.clouddn.com/%E9%85%8D%E7%BD%AEmycnf.png)


接着重启MariaDB
```
systemctl stop mariadb.service 	停止mariadb服务
systemctl start mariadb.service 再次开启mariadb服务，让修改的配置生效
```

然后通过下面命令直接登录MariaDB，不需要输入密码
```
mysql -u root 
```

然后通过如下命令，为root用户修改密码
```
USE mysql ; 
UPDATE user SET Password = password ( 'new-password' ) WHERE User = 'root' ; 
flush privileges ; 
quit
```
效果如图
![MariaDB配置密码.png](http://obfs4iize.bkt.clouddn.com/MariaDB%E9%85%8D%E7%BD%AE%E5%AF%86%E7%A0%81.png)

然后再次修改etc/my.cnf，将skip-grant-tables这行代码注释掉，重启一遍MariaDB服务，让MariaDB数据库再次被密码保护
![进入MariaDB1.png](http://obfs4iize.bkt.clouddn.com/%E8%BF%9B%E5%85%A5MariaDB1.png)

## 安装PHP
最后我们来安装配置PHP
首先通过yum安装PHP
```
yum install php
```

接着为了让Nginx可以操作PHP请求，还需要安装php-fpm
```
yum -y install php-fpm    php与nginx连接软件
```

随后为了让MariaDB与PHP相连接，需要安装php-mysql，php-mysql还可以让MySQL与PHP相连接
```
yum -y install php-mysql    php与mysql连接软件
```

为了使PHP不出现中文乱码，再安装上php-mbstring
```
yum  -y install php-mbstring   php的中文编码库
```

你还可装上php-xml，使得PHP可以解析XML
```
yum install php-xml    php与xml连接软件
```

然后我们启动php-fpm，并设置php-fpm开机自启
```
service php-fpm start
chkconfig php-fpm on
```

在相应的目录下找到nginx.conf配置文件，这里是在默认目录下，开启Nginx支撑PHP的模块
```
cd /usr/local/nginx/conf	进入Nginx配置目录
vim nginx.conf 		配置nginx.conf文件
```

修改内容如下
```
修改前
#location ~ \.php$ {
#    root           html;
#    fastcgi_pass   127.0.0.1:9000;
#    fastcgi_index  index.php;
#    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
#    include        fastcgi_params;

修改后
 location ~ \.php$ {
    root           html;
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    include        fastcgi_params;
}
```
首先将注释，也就是#号去掉，接着将fastcgi_param对应的**/scripts**$fastcgi_script_name改成**$document_root**$fastcgi_script_name

保存完修改后的配置，重启一下nginx，让配置生效，我们就可以通过nginx来处理PHP请求了

可以自己在nginx的网页根目录写一个php文件要验证Nginx是否已经可以处理PHP请求了，这里在/use/local/nginx/html（Nginx默认位置）中创建ayu.php文件，然后写上简单的php代码

```php
<?php
phpinfo();
?>
```
然后通过相应的URL访问该php文件，如果Nginx不支持PHP请求，浏览器就会弹出下载ayu.php文件提示框，若Nginx支持PHP请求，就会相应相应的PHP界面
![nginx访问php.png](http://obfs4iize.bkt.clouddn.com/nginx%E8%AE%BF%E9%97%AEphp.png)

### php-fpm与Nginx的关系
从上面的配置中，我们知道了Nginx需要安装php-fpm后就可以响应PHP请求，但是其中Nginx与PHP是如果通过php-fpm进行协同工作的呢？

要弄明白上面的问题，首先就要知道CGI (Common Gateway Interface) 和 FastCGI 这两个协议

CGI是Web Server与后台语言交互的协议，通过这个协议，开发者可以使用任何语言来处理Web Server发来的请求，动态生成相应的内容，但是CGI对每个请求都需要生成出一个全新的进程来处理，处理完后又会将对应的进程关闭，随着Web的发展，高并发的情景越来常见，CGI这种方式已经不能满足这样的需求了，所以FastCGI就诞生了，看名字就知道，就是快速的CGI，它可以在一个进程内处理多个请求，而不会在请求处理完后结束进程，这样在性能上有很大的提高

而PHP-fpm是FastCGI协议PHP的实现，也就是说，任何实现了FastCGI协议的Web Server都可以与之通信（Nginx就实现了FastCGI），PHP-fpm对应标准的FastCGI还提供了一些增强功能

PHP-fpm是一个PHP进程管理器，它包含master进程（主进程）和worker进程（工作进程），其中master进程负责监听端口，默认配置是监听9000端口，接收来自Web Server的请求，而worker进程一般有多个，具体要多少个，可以对配置文件进行配置，默认是6个，每个worker进程中都内嵌了一个PHP解释器，是PHP代码真正执行的地方
可以通过下面命令查看系统中php-fpm的情况
```
ps -ef | grep fpm
```
![fpm1.png](http://obfs4iize.bkt.clouddn.com/fpm1.png)

从Nginx角度来讲，Nginx处理可以进行HTTP请求的代理还可以进行其他许多协议请求的代理，其中就有与php-fpm相关的FastCGI协议，为了能够让Nginx理解FastCGI协议，Nginx通过FastCGI模块来将HTTP请求映射为对应的FastCGI请求

Nginx的FastCGI模块提供了fastcgi_param指令来处理相应的映射关系，可以看一下Nginx中FastCGI的配置文件，就发现该模块的主要工作就是将Nginx中的变量映射为PHP中能理解的变量
![fastcgi_params.png](http://obfs4iize.bkt.clouddn.com/fastcgi_params.png)

到这里再仔细看一下nginx.conf这个配置文件
![server.png](http://obfs4iize.bkt.clouddn.com/server.png)
![nginx配置.png](http://obfs4iize.bkt.clouddn.com/nginx%E9%85%8D%E7%BD%AE.png)

可以看出，Nginx新建了一个虚拟主机，监听80端口的请求，根目录为html(/usr/local/nging/html)，默认主页为index.html或index.htm

接着就看到开启PHP支持的配置，可以看出将所有以.php结尾的请求都交给FastCGI模块处理，其监听端口为9000，而就像上面讲的，FastCGI模块将变量映射成PHP变量，将请求交给了PHP—fpm来处理

理顺一下思路，当用户对Nginx发起PHP请求时，Nginx首先从80端口接收到PHP请求，然后将该请求发送给9000端口，经FastCGI模块处理交给了在监听9000端口的PHP-fpm，PHP-fpm的master进程将PHP执行任务交个下面的worker进程，worker进程使用内嵌的php解释器来执行PHP并将获得的结果层层返回，最终通过Nginx发出，结果就显示在用户眼前了

到这里CentOS7配置Nginx+PHP+MySQL就完成了
