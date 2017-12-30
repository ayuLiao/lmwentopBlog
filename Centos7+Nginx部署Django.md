# Centos7+Nginx部署Django

本篇文章主要记录一下在Centos7下通过Nginx部署Django的过程，因为项目使用的是Django REST Framework框架，它基于Django，帮我们处理很多细节，适合前后端分离的项目，那么它的部署过程，其实就是Django项目的部署过程

使用环境：
Centos7
Nginx 1.11.9
Python3.5.4
Django1.11.6

如果环境不同，配置的过程可能有所出入，但是不离本质

首先，推荐看uWSGI官方文档，[使用uWSGI和nginx来设置Django和你的web服务器](http://uwsgi-docs-zh.readthedocs.io/zh_CN/latest/tutorials/Django_and_nginx.html)，网上很多搭建文章应该都有参考该文档，本篇文章除了讲搭建的过程，还会提一下，途中踩到的坑

首先看一下如果Nginx+uWsgi+Django搭建起来会是什么样子

```
the web client <-> the web server <-> the socket <-> uwsgi <-> Django
```

这里web server就是Nginx，然后通过socket的方式与uwsgi链接，uwsgi在与Django链接，让客户端可以与Django进行交互

什么是uwsgi呢？

**uwsgi是wsgi的具体实现**，wsgi是Web Server Gateway Interface的简称，直译就是一个Web服务器网关接口，同时wsgi也是python的一种标准。一个Web服务器，如Nginx，它面对的是外部世界，它可以从服务器中返回各种静态文件，如HTML、图片、js、CSS等等，**但是它无法与Django直接通信，所以需要uwsgi作为中间级来链接Django**，Nginx与PHP等也是需要中间件的，可以看我的另外一篇文章[CentOS7搭建配置Nginx+PHP+MySQL](http://lmwen.top/2017/03/15/CentOS7%E6%90%AD%E5%BB%BA%E9%85%8D%E7%BD%AENginx-PHP-MySQL/)

有了概念后，就理一理配置的步骤，首先会配置Nginx，让Nginx可以正常工作，这样就打通了 the web client <-> the web server

然后再配置uwsgi与Django，打通uwsgi<->Django

最后在分别配置Nginx与uwsgi，让两者通过socket来链接

## Centos7安装Nginx

你可以直接通过yum来安装

```
yum install nginx
```

但是本文通过nginx源码进行安装，相对来说比较复杂，但是我已经习惯了，可以明确知道通过源码安装后nginx的各个文件会在那个位置

因为在此前文章中写过，可以直接看
[CentOS7搭建配置Nginx+PHP+MySQL](http://lmwen.top/2017/03/15/CentOS7%E6%90%AD%E5%BB%BA%E9%85%8D%E7%BD%AENginx-PHP-MySQL/)

使用的是nginx1.11.9版本的nginx，如果你也是使用这个版本，那么你对nginx的操作肯定是与我相同的

启动nginx

```
cd /usr/local/nginx/sbin

启动nginx
./nginx
重新载入配置
./nginx -s reload
重新启动Nginx
./nginx -s reopen
停止Nginx
./nginx -s stop
```

通过IP访问一下
![](http://obfs4iize.bkt.clouddn.com/%E8%AE%BF%E9%97%AEnginx.png)

如果没问题，我们就已经将 the web client <-> the web server打通了

## 使用uWSGI
我使用了pyenv-virtualenv进行python的版本控制，进入虚拟环境，我默认你已经安装好了pyenv-virtualenv并安装了python3.5.4，如果不知怎么整，可以看我这一篇博文[pyenv-virtualenv使用](http://lmwen.top/2017/07/30/pyenv-virtualenv%E4%BD%BF%E7%94%A8/)

```
pyenv activate ayu(python虚拟环境名)
```

然后安装uwsgi

```
pip install uwsgi
```

安装完后，首先来测试一下，看看uwsgi能不能与Django打通，其实就是执行Django项目下的wsgi.py文件，一般Django项目中都会自动生成该文件，如图，我的项目名叫wk

![](http://obfs4iize.bkt.clouddn.com/djangowsgi.png)

进入到Django项目的根目录，执行下面语句，wk改成你们自己的项目名

```
uwsgi --http :8000 --module wk.wsgi
```

然后访问 IP:8000，如果返回正常的内容，就说明 the web client <-> uWSGI <-> Django这条路打通了

## Nginx与uWSGI连通
到这里我们已经将两端都打通了，接下来要做的就是将Nginx和uWSGI打通完成Django的部署

这一步也比较简单，首先要做的就是配置Nginx，让Nginx可以与uWSGI通信

因为你安装Nginx的方式可能跟我不同，所以我们Nginx配置文件的位置就可能不同，但是该配置文件的名称一般都是nginx.conf，那么找到它，我的在/usr/local/nginx/conf/目录下

```
[root@izj6c1jjxlrlkc6d56dnl7z conf]# ll
total 64
-rw-r--r-- 1 root root 1077 Nov 17 11:00 fastcgi.conf
-rw-r--r-- 1 root root 1077 Nov 17 11:00 fastcgi.conf.default
-rw-r--r-- 1 root root 1007 Nov 17 11:00 fastcgi_params
-rw-r--r-- 1 root root 1007 Nov 17 11:00 fastcgi_params.default
-rw-r--r-- 1 root root 2837 Nov 17 11:00 koi-utf
-rw-r--r-- 1 root root 2223 Nov 17 11:00 koi-win
-rw-r--r-- 1 root root 3957 Nov 17 11:00 mime.types
-rw-r--r-- 1 root root 3957 Nov 17 11:00 mime.types.default
-rw-r--r-- 1 root root 3187 Dec 26 12:48 nginx.conf
-rw-r--r-- 1 root root 2656 Nov 17 11:00 nginx.conf.default
-rw-r--r-- 1 root root  636 Nov 17 11:00 scgi_params
-rw-r--r-- 1 root root  636 Nov 17 11:00 scgi_params.default
-rw-r--r-- 1 root root  664 Nov 17 11:00 uwsgi_params
-rw-r--r-- 1 root root  664 Nov 17 11:00 uwsgi_params.default
-rw-r--r-- 1 root root 3610 Nov 17 11:00 win-utf
-rw-r--r-- 1 root root  626 Dec 10 22:01 wkshop.conf
```

通过vim编辑nginx.conf，在该目录下，还发现了uwsgi_params，在修改nginx.conf时需要使用，具体的配置内容如下

```
upstream django {
    # server unix:///path/to/your/mysite/mysite.sock; # for a file socket
     server 127.0.0.1:8001; # for a web port socket (we'll use this first)
     }

    server {
        listen       8000;
        
        server_name  你的域名或者IP;
        charset utf-8;
        
        client_max_body_size 75M;

        location /media {            
            alias /root/wk/media;
            
        }
        location /static {
            alias /root/wk/static;
        }

        location / {
            include  uwsgi_params;
            uwsgi_param UWSGI_PYHOME /root/.pyenv/versions/wkpy35
            uwsgi_pass django;
        }
       
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
       
    }
```

解释一下上面的配置代码

upstream django 只是取了个名字，它表示nginx与uwsgi的socket通信端口

在server中，配置Nginx的监听端口为8000，server_name可以配置为自己的域名，没有域名可以配置成IP，client_max_body_size客户端最大请求大小

接着将Django中存放资源的media和存放静态文件的static文件夹配置给Nginx，让Django中这两个文件夹的路径与Nginx对应，然后就是配置根路径，首先将于nginx.conf文件同一目录的uwsgi_params引入，接着配置使用uWSGI的python环境，最后就是配置uwsgi_pass，将前面设置好的django配置给它

这样就完成了Nginx的配置了，重启一下Nginx，让其重新加载nginx.conf配置文件，然后以socket的方式启动一下uWSGI，在Django项目的根目录下运行下面的代码

```
uwsgi --socket :8001 --module mysite.wsgi
```

需要注意的是，uWSGI启动的接口是8001，也必须是8001，因为在nginx.conf文件中，我们配置了Nginx与uwsgi通信的接口为8001，正常启动后，就可以访问一下你的服务器，IP:8000，注意，访问的是8000端口

![django访问succ.png](http://obfs4iize.bkt.clouddn.com/django%E8%AE%BF%E9%97%AEsucc.png)

可能有人被8001和8000搞混了

仔细看Ningx的配置文件，其实是可以明白的，Nginx的监听端口是8000，那么我们要访问Nginx就需要将请求服务器的8000，Nginx收到请求后，会进行转发，转发的端口同样配置在了nginx.conf上，也就是8001端口，为了让uwsgi可以正常接收到Nginx转发过来的数据，那么uwsgi也就需要在8001端口开启，那我们之间请求8001，不通过Nginx直接访问uWSGI不行吗？不行，因为uWSGI是以socket形式启动的，不会相应http请求，所以必须通过Nginx才能访问uWSGI，不然就会出现如下的跳过log

```
invalid request block size: 21573 (max 4096)...skip
invalid request block size: 21573 (max 4096)...skip
invalid request block size: 21573 (max 4096)...skip
```

其实到这里我们已经将**the web client <-> the web server <-> the socket <-> uwsgi <-> Django**打通了，为了方便，将uWSGI的相关设置都写到一个以ini结尾的文件中，比如我创建了wk.ini文件，然后将下面配置写到文件中

```
[uwsgi]
chdir = /root/wk
socket = :8001
wsgi-file = /root/wk/wk/wsgi.py
master = true
uid = root

processes = 2
threads = 4

chmod-socket = 666
chown-socket = root:nginx

buffer-size = 32768
vacuum = true 
```

chdir：Django项目的路径
socket：uWSGI运行的接口
wsgi-file：Django项目wsgi文件的位置

processes：进程数
threads：线程数

chmod-socket：权限
chown-socket：(centos用户名):(centos组)

写完后，然后运行一下

```
uwsgi --ini wk.ini
```

访问IP:8000，正常访问就表面配置文件没问题

## 结尾
到这里就完成了Centos7上通过Nginx配置Django的所有过程，可能在上面的步骤中，你有些步骤不能成功，这时看看你有无开防火墙限制端口，如果本地防火墙没有限制端口，需要去运营商如阿里云上看看有无将8000端口和8001端口开放出来


