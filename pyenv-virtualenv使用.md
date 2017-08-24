---
title: pyenv-virtualenv使用
date: 2017-07-30 12:32:16
tags: python
thumbnail: http://obfs4iize.bkt.clouddn.com/pyenv_vertualenv%E4%BD%BF%E7%94%A8.jpg
---

## 简介
今天在看《精通Python网络爬虫》这本书，其实对于Python爬虫我是有一点经验的，自己在网上看了很多Python爬虫的教程，也跟着实现了一些python爬虫，但是有点野蛮生长的感觉，所以想着系统化的去看看这方面的内容，我觉得看书是很好的一种方式，花点前去买本技术书，虽然这些内容并不一定只能在书上看，网上肯定也有相关的内容，但是重要的是，技术书籍帮你整理好了，免费的东西虽有，但是好坏难分，找也要花费时间，所以我更偏向于买本书看看。

所以我就开始看《精通Python网络爬虫》，想知道自己在Python爬虫方面是否还有些自己没听过没使用过的套路。看这本书的时候，也给我带来了一个问题，因为我使用Python其实是偏向于Python2.x版本的，写Python3.x版本的代码比较少，虽说Python2.x和Python3.x在语法方面的差别并不是非常大，但是Python2.x和Python3.x在urllib库上的用法还有有较大差别的，python2.x有两个版本urllib和urllib2，python3.x将两个版本合成了urllib，所以在用法上有点不同，所以我想跟随的书的步伐也在Python3.x下开发爬虫，可是我的系统环境其实是Python2.x，因为个人的习惯，我不想让很多Python第三方库污染我的系统Python的环境，所以一般都会使用virtualenv、virtualenvwrapper来进行版本控制，但virtualenv工具会根据系统默认的Python版本来形成一个虚拟环境，现在我系统默认的是Python2.x，但是我想创建Python3.x的虚拟环境，光是virtualenv无法满足的我想需求，所以我一开始尝试通过Pyenv下载3.6.2的系统环境，然后将默认系统的环境通过Pyenv切换到3.6.2下，不懂的可以去看[python多版本控制](http://lmwen.top/2017/05/26/python%E5%A4%9A%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6/)，切换后，再通过virtualenv来创建虚拟环境，猜想这样virtualenv根据此时系统的默认版本Python3.6.2来创建出具有相同版本的虚拟环境，我尝试后，发现，是我太天真了。

## pyenv-virtualenv介绍
难道就没有办法实现我的需求了吗？不可能，因为我感觉这是一个很普遍的需求，有点经验的Python开发者对Python环境都是有洁癖的，我也同样有，毕竟以前在大杂锅的Python环境下，出过莫名其妙的问题，然后通过Google一搜，就发现了pyenv-virtualenv这个神器。

你没猜错，pyenv-virtualenv就是用于创建指定Python版本的虚拟环境，这样无论是系统默认是什么环境，你都可以指定要创建什么版本的Python，当然这个版本的Python已经通过Pyenv下载了

首先你要明白，pyenv-virtualenv是基于pyenv，其实就是pyenv的一个插件，所以要使用pyenv-virtualenv，就要先装上pyenv，具体可看[python多版本控制](http://lmwen.top/2017/05/26/python%E5%A4%9A%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6/)

[pyenv-virtualenv的github地址](https://github.com/pyenv/pyenv-virtualenv)

## 安装配置pyenv-virtualenv

1.我们需要从Github是将其clone下来，或者自己去https://github.com/pyenv/pyenv-virtualenv上下载

```
git clone https://github.com/pyenv/pyenv-virtualenv.git
```

2.将pyenv-virtualenv拷贝到pyenv的插件目录下

pyenv插件目录，可以看到我已经将pyenv-virtualenv复制到该目录下了
![pyenv配置文件默认路径.png](http://obfs4iize.bkt.clouddn.com/pyenv%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E9%BB%98%E8%AE%A4%E8%B7%AF%E5%BE%84.png)

3.然后将 pyenv的virtualenv-init添加到你的shell，这样可以让pyenv-virtualenv使用更加方便，通过shell的相关命令就可以使用了，这个是可选的，你可以不加到shell的配置中，但是建议你还是加

通过下面的命令就可以将pyenv的virtualenv-init添加到你的shell

```
$ echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile
```

4.然后重启shell，让它重新加载自己的配置文件，让我们可以启用pyenv-virtualenv

```
$ exec "$SHELL"
```

通过上面4布就完成了pyenv-virtualenv的配置了

## 使用pyenv-virtualenv

先看一下我的系统，ubuntu16.0.4下默认的python版本
![系统默认python版本.png](http://obfs4iize.bkt.clouddn.com/%E7%B3%BB%E7%BB%9F%E9%BB%98%E8%AE%A4python%E7%89%88%E6%9C%AC.png)

我们先安装python3.6.2
```
pyenv install 3.6.2
```

安装完后，我们就可以创建Python3.6.2的虚拟环境了

```
pyenv virtualenv 3.6.2 ayuliaotestpy3
```

![创建py3虚拟环境.png](http://obfs4iize.bkt.clouddn.com/%E5%88%9B%E5%BB%BApy3%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83.png)

如上图，我们就安装成功了

接着就可以通过pyenv activate使用了
```
pyenv activate ayuliaotestpy3
```

![使用ayuliaotest3.png](http://obfs4iize.bkt.clouddn.com/%E4%BD%BF%E7%94%A8ayuliaotest3.png)

可以发现，现在虚拟环境的python已经是python3.6.2了

可以通过下面命令来退出虚拟环境
```
pyenv deactivate
```
![退出虚拟环境.png](http://obfs4iize.bkt.clouddn.com/%E9%80%80%E5%87%BA%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83.png)

查看系统中创建了多少个虚拟环境
```
pyenv virtualenvs
```

![创建虚拟环境的个数.png](http://obfs4iize.bkt.clouddn.com/%E5%88%9B%E5%BB%BA%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83%E7%9A%84%E4%B8%AA%E6%95%B0.png)

仔细看途中，其实只创建了两个虚拟环境，一个是ayuliaotestpy3，一个是WebCrawlerPy3

删除虚拟环境
```
pyenv uninstall ayuliaotest3py3
```

![删除虚拟环境](http://obfs4iize.bkt.clouddn.com/%E5%88%A0%E9%99%A4%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83.png)

到这里，pyenv-virtualenv的基本用法就介绍完了，这些就够用了，其他更多用法，可以看pyenv-virtualenv官方介绍

用代码编写乐趣---ayuliao
