## 简介

因为要在极客学院录课，录课的要求比较繁杂，总而言之就是暂时不能用Ubuntu来录，没办法，为了赚钱生存，只能将台式装回win10，不过固态硬盘就是快，30个钟就搞定了，那么在windows上怎么让python2和python3完美共存呢？

## virtualenv
在windows上依旧可以安装virtualenv，因为工作原因，依旧使用着python2.7，但是感觉python3使用的人越来越多了，但是想要在公司大面积使用，python3就要稳点，动不动就升个级，应该让很多公司暂时不愿使用python3，当然这是我个人所见，也可能是我一叶遮秋了


windows上安装virtualenv
```
pip install virtualenv
```

但是virtualenv并不能解决不同版本python的问题，特别是有时候要使用python3时，我们都知道pyenv只支持linux环境，所以在windows下其实不适应

这个需求其实很常见，很多都是下载好两个版本的python，然后在添加环境变量时修改一下添加时python的名词，如python2.7的python.exe可以改为python27.exe，这当然不失为一个办法，但是并不好，你这样一改，你后面就会发现很多东西就报错或者装不上了，因为他们默认使用的python.exe

其实还有更加简单的方法

## windows下python2和python3完美共存
同样，直接安装python2和python3

python3其实已经考虑到这种情况了，它为我们提供了py这个命令

所以当我们使用python这个命令时，使用的是python2.7，而使用py这个命令使用的是python3，简直不要太简单

![py2和py3.png](http://obfs4iize.bkt.clouddn.com/py2%E5%92%8Cpy3.png)

那么怎么分别使用python2和python3下的库呢？

同样可以使用py命令

这里以pip命令为例

在安装了python2和python3的win10环境下，直接使用pip，默认是使用python2.7，而我们使用下面命令

```
py -3 -m pip --version
```

就可以使用python3的pip
![pip和pip3.png](http://obfs4iize.bkt.clouddn.com/pip%E5%92%8Cpip3.png)


在进行项目开发的时候，就更简单了，使用pycharm，换环境，分分钟，直接打开pycharm的setting，进入如图处，就可以更换python开发环境了，同时还可以通过pyCharm来安装第三方库，当然pyCharm安装第三方库的方式也是通过pip来实现的，但是有时候会安装失败，我懒得深究，直接通过命令行来安装，如果事事深究，那你就完了

![pycharm环境.png](http://obfs4iize.bkt.clouddn.com/pycharm%E7%8E%AF%E5%A2%83.png)

## python3创建虚拟环境

因为有环境洁癖，所以我喜欢创建不同的python虚拟环境来开发不同的项目，那么问题来了，我们都知道可以使用virtualenv来创建虚拟环境，但是python2和Python3的环境下怎么使用？

首先我天真的使用

```
py -3 -m pip install virtualenv
```

来安装py3下的virtualenv，然后我就有点奇怪，使用py -3 -m 来调用，会报错，直接通过virtualenv，依旧是创建出了python2的虚拟环境

Google一下，发现python3同样已经给出了解决方法

python3.3以上的版本可以使用venv这个模块来创建虚拟环境，这个可以直接代替我们经常使用的python virtualenv

venv这个模块是python3.3以上内置的，看了大部分python开发人员都有环境洁癖，该模块提供创建**轻量级**的python虚拟环境，虽然现在我也搞不懂它有什么轻量级的，但是在windows上的效果跟virtualenv几乎一样

venv提供与系统python相独立的一个环境，每个环境中有自己的python二进制文件，同样可以拥有独立的一套python包，也就是第三方包同样可以独立安装到这个虚拟环境中，但是在python3.3中使用venv创建的虚拟环境不包含pip，所以你需要自己手动安装，但是，在python3.4中改进了这个缺陷，也就是说，在python3.6中是没有毛病的

![py3创建虚拟环境.png](http://obfs4iize.bkt.clouddn.com/py3%E5%88%9B%E5%BB%BA%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83.png)

使用方式更virtualenv一样

用代码编写乐趣---ayuliao
