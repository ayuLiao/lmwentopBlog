---
title: python多版本控制
date: 2017-05-26 09:13:31
tags: python
---

## 简介

有稍微接触python的人就会知道，python中有两个比较热门的版本，一个是python 2.7.x ，一个是python3.x，因为python易用，很多linux系统上都默认集成了某个版本的python，我见的大多数是python2.7.x。因为python2.7.x和python3.x之间存在一些差异，所以使用不同版本开发的python程序并不能完全兼容，伴随着这个问题的出现，python版本控制变得越来越重要。

## 安装pyenv进行版本控制

为了解决python多版本带来的问题，python版本切换工具---pyenv应运而生。
pyenv工具的作用很明显，就是切换不同python的版本，满足相应的开发需求，比如开发一个项目需要用到python3.x，而此时使用的却是python2.7.x，就可以通过pyenv安装python3.x，然后将系统默认的python切换成3.x版本的python。

pyenv工具是github上的一个开源项目（github是一个代码托管网站），所以我们可以直接通过git工具获得pyenv对于的代码，然后修改系统的环境变量，就可以在任意目录下运行pyenv工具，进行python的版本切换了，具体的操作如下

```
$ git clone https://github.com/yyuu/pyenv.git ~/.pyenv     #使用 git 把 pyenv 下载到家目录
$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc     #然后需要修改环境变量，使用 Bash Shell 的输入

$ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc

$ echo 'eval "$(pyenv init -)"' >> ~/.bashrc     #最后添加 pyenv init

$ exec $SHELL -l     #输入命令重启 Shell,然后就可以重启pyenv
```

![](http://onxxjmg4z.bkt.clouddn.com/pyenv%E5%AE%89%E8%A3%85.png)

通过上面的操作后，我们就可以使用pyenv了，使用下面命令来看一下pyenv可以安装那些python版本。
```
$ pyenv install --list
```
![](http://onxxjmg4z.bkt.clouddn.com/pyenv%E7%89%88%E6%9C%AC%E5%88%97%E8%A1%A8.png)

你会发现pyenv支持很多不同版本的python，这些版本都有各自的优缺点，在不同的需求上可以使用不同python版本，比如在数据处理方面，可以使用科学技术的python，也就是anaconda。

此时，我系统中的python版本是2.7.12，这里我们通过pyenv来安装python3.6.1，通过下面命令就可以轻松的安装了
```
$ pyenv install 3.6.1 
```
该命令会从相应的地址上下载我们需要的3.6.1,然后直接为我们安装
![](http://onxxjmg4z.bkt.clouddn.com/pyenv%E5%AE%89%E8%A3%853.png)

这种方式非常简单，但是通过这中方式下载对应版本的python比较慢，所以改变一下方法，首先依旧是使用pyenv install 3.6.1 这条命令，该命令会显示出相应的下载地址，获得下载地址后，可以通过ctrl+c强制停止该命令的运行，然后通过wget或迅雷来下载安装，并将下载好的文件放到 ~/.pyenv/cache文件夹下，cache一开始是不存在的，需要用户自己在~/.pyenv目录下创建，然后再执行下面命令
```
$ pyenv install 3.6.1 -v  #不要忘记-v参数
```

**步骤总结如下：**

**1.执行 pyenv install 3.6.1 获取下载链接**

**2.用wget从下载链接中获取文件 Python-3.6.1.tar.xz**

**3.将安装包移动到 ~/.pyenv/cache/Python-3.6.1.tar.xz**

**4.重新执行 pyenv install 3.6.1 -v 命令。该命令会检查 cache 目录下已有文件的完整性，若确认无误，则会直接使用该安装文件进行安装。**

通过上面的操作，就把python3.6.1版本安装好了，此时最好先更新一下pyenv的数据库，命令如下
```
$ pyenv rehash
```

此时我们来查看一下系统中安装过那些python版本
```
$ pyenv versions
```
![](http://onxxjmg4z.bkt.clouddn.com/pyenv%E6%9F%A5%E7%9C%8B%E7%89%88%E6%9C%AC.png)

从图中可以看出，该系统中有个系统自带的python版本，还有一个就是我们刚刚安装的python3.6.1

如果通过上面的操作，发现python3.6.1并没有出现，也就是安装失败了，很可能系统中没有相应的依赖

pyenv install 3.6.1 -v 命令会检查 cache 目录下的安装包，安装包没有问题就直接进行安装，在安装的过程中，需要对文件进行编译，此时需要相应的依赖，通过下面的命令安装依赖的库

ubuntu下
```
sudo apt-get update
sudo apt-get install make build-essential libssl-dev zlib1g-dev
sudo apt-get install libbz2-dev libreadline-dev libsqlite3-dev wget curl
sudo apt-get install llvm libncurses5-dev libncursesw5-dev
```

Centos/Fedora下
```
sudo yum install readline readline-devel readline-static
sudo yum install openssl openssl-devel openssl-static
sudo yum install sqlite-devel
sudo yum install bzip2-devel bzip2-libs
```

安装完上面的依赖后，再次执行pyenv install 3.6.1 -v 命令，就会安装成功

接着通过下面命令切换系统全局的python版本
```
$ pyenv global anaconda3-4.1.0
```
![](http://onxxjmg4z.bkt.clouddn.com/pyenv%E5%88%87%E6%8D%A2%E7%89%88%E6%9C%AC.png)

还可以直接在命令行中输入python来验证当前全局的python版本，可以发现系统版本已经变为3.6.1了
![](http://onxxjmg4z.bkt.clouddn.com/%E7%B3%BB%E7%BB%9F%E7%89%88%E6%9C%AC%E5%8F%98%E4%B8%BA3.png)

使用同样的命令，将系统全局python切回原来的版本
![](http://onxxjmg4z.bkt.clouddn.com/%E5%88%87%E5%9B%9E%E6%9D%A5.png)

如果你想要将python的某个版本卸载掉，可以使用下面命令
```
$ pyenv uninstall 
```

如果你想更新pyenv，可以使用下面命令
```
$ pyenv update 
```

## 安装virtualenv创建纯净虚拟环境

虽然通过pyenv进行python的版本切换已经不错了，但是每次开发不同python版本的项目时都要切换python版本，有点费力。当我们不同的项目要用到不同版本的库时，pyenv就爱莫能助了，举个具体的例子，Test1和Test2都要使用Request这个第三方库，但是Test1要求使用1.0版本的Request，而Test2要求使用2.0版本的Request，此时pyenv并不能对库进行版本切换，无论pytyon本身的版本如何切换，系统中使用的库默认都是一个版本的，当然可以通过一些配置让不同的项目不从不同的路径上加载需要的库，当不同的项目变得庞大繁杂时，这种方式会显得是否笨拙。

为了解决这种问题，我们可以使用virtualenv，virtualenv的作用是什么？通俗易懂的讲就是创建一个虚拟环境，不同虚拟环境内的python版本和库的版本都可以相互独立、互不影响，同样回到上面的例子，首先我们通过virtualenv创建Test1和Test2这两个虚拟环境，并在这两个环境中配置相应的python版本和库，如在Test1这个虚拟环境中，我们使用python2.7.x和1.0版本的Request，而在Test2这个虚拟环境中使用python3.x和2.0版本的Request，在这两个虚拟环境中分别创建Test1项目和Test2项目，这两项目之间就不会相互影响，使用各自的虚拟环境中使用各自的配置，这样开发不同需求的项目就变得简单而易于管理。

那么首先我们来安装virtualenv，通过下面命令可以简单安装

```
$ pip install virtualenv
```
在我的ubuntu系统中已经默认安装了virtualenv
![](http://onxxjmg4z.bkt.clouddn.com/%E5%AE%89%E8%A3%85virtualenv.png)

安装完后，就可以使用virtualenv创建相应的虚拟环境了，命令如下

```
$ virtualenv test
```

![](http://onxxjmg4z.bkt.clouddn.com/%E5%88%9B%E5%BB%BA%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83.png)

从图中可以看到，test这个虚拟环境被安装到了/home/ayuliao目录下，进入相应的目录，这里是/home/ayuliao/test/bin，可以通过ls命令来查看该虚拟环境下拥有那些文件

运行其中的activate文件，进入相应的虚拟环境
```
$ source activate
```

![](http://onxxjmg4z.bkt.clouddn.com/%E8%BF%9B%E5%85%A5%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83.png)

解释一下这些文件：

1. activate ：这个virtualenv的激活文件，运行该文件就可以进入相应的虚拟环境汇中
2. pip：这个virtualenv独立的pip，与系统的pip相互独立了
3. python：系统python解释器的一个副本，系统全局下使用了哪个版本的python，该虚拟环境就会使用哪个版本的python
4. python2.7：所有的新包会被存放到该文件夹下

接着我们可以通过下面命令来看一下该虚拟环境拥有的包/库
```
$ pip list --format=columns #列的形式显示
```

![](http://onxxjmg4z.bkt.clouddn.com/%E6%9F%A5%E7%9C%8B%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83%E7%9A%84%E5%BA%93.png)

可以发现，虚拟环境中已经有了一些基本的库了

我们可以通过该虚拟环境的pip工具安装我们需要的第三方库，在虚拟环境中安装的任何库都不会污染系统中python的环境和其他的虚拟环境

在虚拟环境中做完一些操作后，就可以通过下面命令退出虚拟环境
```
$ deactivate
```
![](http://onxxjmg4z.bkt.clouddn.com/%E9%80%80%E5%87%BA%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83.png)

但是每次通过virtualenv创建一个虚拟环境后，都要进入该虚拟环境对应的目录启动其中的activate文件，比较麻烦，有没有更方便的方法？毕竟人生苦短嘛。

当然有，我们可以安装virtualenvwrapper，virtualenvwrapper其实就是virtualenv的扩展管理包，使用它可以更加方便虚拟环境

首先进行virtualenvwrapper的安装

```
$ pip install virtualenvwrapper
```
![](http://onxxjmg4z.bkt.clouddn.com/%E5%AE%89%E8%A3%85virtualenvwapper.png)

刚安装完后，virtualenvwrapper是不可以直接使用的，它会默认在/usr/local/bin下生成virtualenvwrapper.sh文件

![](http://onxxjmg4z.bkt.clouddn.com/%E6%9F%A5%E7%9C%8Bvirtualevwapper.png)

你需要运行该文件才行，现在先别急着运行该文件，先看一下文件中写了什么，可以使用vim来查看

![](http://onxxjmg4z.bkt.clouddn.com/%E6%9F%A5%E7%9C%8Bvirtualevwapper%E9%85%8D%E7%BD%AE.png)

可以看见，该文件中写好了我们需要做什么配置，下面按照该文件的提示，配置好相应的环境

将相应的内容写入到~/.bashrc中
```
$ echo 'export WORKON_HOME=$HOME/.virtualenvs' >> ~/.bashrc
$ echo 'source /usr/local/bin/virtualenvwrapper.sh' >> ~/.bashrc
```

再运行bashrc文件
```
$ source ~/.bashrc
```

此时virtualenvwrapper就可以使用了。

通过下面命令创建一个虚拟环境

```
$ mkvirtualenv test2
```

![](http://onxxjmg4z.bkt.clouddn.com/%E5%88%9B%E5%BB%BA%E8%99%9A%E6%8B%9F%E7%A9%BA%E9%97%B4.png)

你会发现，创建完后，直接就进入虚拟环境了，不再用进入相应的目录下手动启动activate文件，非常方便。

同样可以通过deactivate推出虚拟环境，退出后，可以通过workon命令查看到已经创建的虚拟环境

```
$ workon 
```
![](http://onxxjmg4z.bkt.clouddn.com/%E6%9F%A5%E7%9C%8B%E8%99%9A%E6%8B%9F%E7%A9%BA%E9%97%B4.png)

可以看见这里现实出了我们刚刚创建了test2虚拟环境,但是却没有显示直接通过virtualenv创建的test虚拟环境

通过下面命令进入相应的虚拟环境
```
$ workon test2
```

当我们不想在使用该虚拟环境时，可以通过下面命令将该虚拟环境删除
```
$ rmvirtualenv test2
```

![](http://onxxjmg4z.bkt.clouddn.com/%E7%9B%B4%E8%BF%9B%E8%99%9A%E6%8B%9F%E7%A9%BA%E9%97%B4.png)

这里提一下，如果想删除直接通过virtualenv命令创建的虚拟环境，可以将该虚拟环境对于的文件夹直接整个删除，该虚拟环境就从你的计算机上移除了。

## 结尾
后面的开发，我们都会在虚拟环境中进行。