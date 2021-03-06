---
title: 理解python编码问题
date: 2017-06-20 09:48:40
tags: python
---

## 简介
在刚玩python爬虫的那段时间，时常获取到乱码内容，索然通过Google暂时解决了问题，但是一不留意，下次又会在其他地方出现这样的问题，感觉每次解决的办法都有点不同，所以花了段时间去了解原理，后来跟同学讨论，觉得有必要花点时间记录一下，避免以后自己思路混乱了

## 背景知识
首先要明白几个概念，字节、字符、字符集、字符码、编码和解码，有了这几个背景概念，就很好搞了

### 比特
比特，计算机中最小的单位了，其实就是0和1，在电气层面上解释，计算机存储东西都是通过电流改变硬盘中的磁极磁性，也就是改变正负极，那么抽象成数学上的意义就是0或1，可能你也听过，真正的将计算机中的内容完全删除，要将硬盘低格7次以上，为的就是不让他人使用工具将磁盘的磁性还原回去，当然直接电磁炉里转两圈更靠谱

### 字节
字节非常好理解，一个字节等于8个比特，如00000001，计算机中的数据都是以字节为单位来存储的，无论是文字、图片或者视频等等，网络传输中常说的字节流也是字节

### 字符
现在这篇文章就是有字符组成的，字符一般是个信息单位，字符由字节组成，但是不同的字符由不同数量的字节组成，如英文字符a，由一个字节则可表示，但是博大精深的汉字则要多个字节（2~3个）

### 字符集
简单的理解为某个范围内字符的集合，如ASCII，就是128字符的集合，里面包含的数字、字母等，很明显ASCII这个字符集太小了，根本不够世界上所有的国家使用，如中国、日本等，ASCII中根本没有这些国家字体的表示，所以这些国家就自己弄了字符集，这样不同地区的计算机交流就会出现乱码，比如在ASCII字符集中，第65个表示A这个字符，但是在GBK这个字符集中就不是A了，那么就出现了乱码

### 字符编码
字符编码就是给每个字符在对应的字符集中一个特定的编号，如A在ASCII字符集中的编号为65，ASCII字符集中的这种对应关系就叫ASCII编码，同样还有GBK编码、UTF-8编码等等，为什么字符集中要相应的编号呢？？？因为将编号转成对应的字节，也即是相应的比特，存入计算机存储中，如65，其字节为00100001，其实就是一种抽象，将A抽象成65，不同的字符编码就是不同的抽象规则

### 编码、解码
了解了上面的内容，这两个词的含义就好理解了，编码就是将字符抽象成相应的编码号转成字节进行存储的过程，至于该字节抽象成哪个编码号，就看是那种字符编码了，如果是ASCII编码，那就更加ASCII字符集中对应的关系进行抽象，如果UTF-8编码，就通过UTF-8字符集中对应的关系进行抽象，解码也同样，就是编码的过程反过来了，如果你是ASCII编码，那么就通过ASCII字符集中对应的关键就像具化（也就是获得对应的字符）

## 造成乱码的核心原因
其实很简单，就是编解码不一致，比如你解码使用UTF-8，编码却用ASCII，那么出来的当然是乱码，举个比较常见的例子，你的python文件编码格式是utf-8，可中文windows系统本地默认编码是gbk，此时你通过控制台打印字符串时必然是乱码，具体代码如下

简单的输出一下
```python
# -*- coding:utf-8 -*-
print "帅比"
```

错误界面
![python乱码.png](http://obfs4iize.bkt.clouddn.com/python%E4%B9%B1%E7%A0%81.png)

修改一下代码，看看系统的编码

```python
# -*- coding:utf-8 -*-
import sys
type = sys.getdefaultencoding() 
print type
print "ayuliao"
```

从下图中可以看出，系统编码为GB2312(GB2312编码是第一个汉字编码国家标准，由中国国家标准总局1980年发布，1981年5月1日开始使用。GB2312编码共收录汉字6763个，其中一级汉字3755个，二级汉字3008个)

![系统编码.png](http://obfs4iize.bkt.clouddn.com/%E7%B3%BB%E7%BB%9F%E7%BC%96%E7%A0%81.png)

解决办法就是先将要输出的内容解码，再进行相应的编码，因为文件本身是utf-8编码，所以要先解utf-8编码，然后在进行GB2312编码，这样在控制台输出的内容就不会是乱码

```python
# -*- coding:utf-8 -*-
import sys
type = sys.getdefaultencoding() 
print "帅比".decode('utf-8').encode(type)
```

控制台正常显示
![python编码正常.png](http://obfs4iize.bkt.clouddn.com/python%E7%BC%96%E7%A0%81%E6%AD%A3%E5%B8%B8.png)

我们再修改一下代码
```python
# -*- coding:utf-8 -*-
import sys
type = sys.getdefaultencoding() 
reload(sys)
sys.setdefaultencoding('utf8')
type2 = sys.getdefaultencoding()
print type
print type2
print "帅比".encode(type)
```

结果如下
![改变系统编码.png](http://obfs4iize.bkt.clouddn.com/%E6%94%B9%E5%8F%98%E7%B3%BB%E7%BB%9F%E7%BC%96%E7%A0%81.png)

为什么这样就不用将“帅比”通过utf-8解码了呢？这里不用感到奇怪，因为python中存在隐式解码，也就是在print输出到控制台之前，如果你没有使用decode()函数进行显示界面，而你又调用了encode()方法进行编码，那么就会发送隐式解码，隐式解码会使用系统的字符编码，如果系统的字符编码为gb2312，那么"帅比".encode(type)相当于"帅比".decode(gb2312).encode(type)，这样的结果当然会是乱码，因为文件本身是utf-8编码的，你却使用gb2312解码，此时就是乱码了，你在通过encode()方法怎么编码都没用，这也是很多人遇到的坑，看见出现编码问题了，立刻使用encode()方法，然后一个个字符编码的尝试，先来utf-8，什么，竟然不行，那再来gbk，什么还是不行.....

那python为什么要有隐式解码这个坑比呢？那就要讲讲编码历史了，在很久很久以前，有很多中编码，如ASCII、GBK等等，这个大家都知道，这些编码是不能相互交流的，因为会产生乱码，但是过多的编码使得世界变得混乱，所以就出现了unicode，它可以包含所有不同种类的编码，正式因为所有编码在unicode都不会出现混乱，也就是乱码，所以**python就使用unicode作为编解码中的中间编码**，这样GDK要转ASCII时，因为GDK直接转ASCII会出现乱码，所以就通过GDK-->Unicode-->ASCII的形式，当然细心的人可能会问ASCII也就127位，GDK远超于ASCII，超出的部分怎么办？？？凉拌....没事GDK转ASCII干啥，一般都是ASCII不包含汉字，直接输出会有乱码，所以要进行编解码，将ASCII转成包含汉字的GDK。

扯远了，但是从上面也可以看出python为什么要弄隐式解码，因为两种编码不能直接转码，直接转码很有可能出现乱码，所以要先将他们都解码成Unicode，然后再进行编码，所以python如果看你使用解码方法decode()就会偷偷的帮你用，使用系统默认的字符编码帮你解码，因为python认为你的文件中的内容跟系统应该是一样的，但是在中文世界中，经常不一样....

通过python隐式编码的特性，我们可以让系统重新加载一个字符编码，写法如下，但是在python3中，好像就不存在这些问题了，所以python3才是未来

```python
import sys
reload(sys)
sys.setdefaultencoding('utf8')
```

## 网页乱码
上面的讨论都是在知道编码的情况下解决乱码，因为都是本地字符输入输出，但是当你去获取网页上的内容时，出现乱码怎么办，其实也简单，同样的思路，出现乱码问题的原因就是编解码不同，那么就要知道编码是使用什么字符编码，解码是使用哪个字符编码，本地我们可以轻易的知道使用了什么字符编码，但是网页使用了什么字符编码我们却不知道，一种方法就是查看网页源代码，一般该网页使用了什么编码会在head标签中声明出来的

![查看网页编码格式.png](http://obfs4iize.bkt.clouddn.com/%E6%9F%A5%E7%9C%8B%E7%BD%91%E9%A1%B5%E7%BC%96%E7%A0%81%E6%A0%BC%E5%BC%8F.png)

大多数网页都会使用utf-8，但是并不是所有网页，那每次编写一个网页爬虫都要看网页相关的编码太麻烦了，那更简单的方法就是chardet这个字符编码判断包，你可以通过pip安装它，用法也非常简单，我们可以使用urllib包将对应网页的内容爬取下来，然后将内容作为参数传入chardet包的detect()方法中，它会给你一个它认为与之最相近的字符编码，confidence表示可能是该字符编码的概率。这样获得了网页的字符编码后，你就可以进行我上面讲的编解码了，这样爬取下来的网页内容就不会出现乱码了

```
>>> import urllib
>>> rawdata = urllib.urlopen('http://www.google.cn/').read()
>>> import chardet
>>> chardet.detect(rawdata)
{'confidence': 0.98999999999999999, 'encoding': 'GB2312'}
```

## 结尾
本章没有讲到python2和python3的编码区别，在python3中，对编码做了比较大的改动，后面有时间会写写

用代码创造乐趣---ayuliao