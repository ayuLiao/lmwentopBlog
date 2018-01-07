# python在线编程环境搭建与体验
因为公司项目要求，要在网站上搭建一个用于python开发的Web IDE，问了一下身边的朋友，没什么特别好的结果，因为这个任务不怎么着急，也就没有深入研究，直到昨天无意中发现了codemirror，开始了Python编程环境的折腾

## codemirror
codemirror是一个用js实现的编辑器，有比较好的代码自动识别格式化的功能，功能也比较强大，通过简单的配置就可以支持多种语言和不同的界面风格，可以去[codemirror官网看看](http://codemirror.net/)

codemirror是开源的项目，代码都在github上，[codemirror github](https://github.com/codemirror/codemirror)，可以自己git clone将它拉下来

主要的文件夹结构如下

![](http://obfs4iize.bkt.clouddn.com/codemirror%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84.png)

经过简单研究，要使用，只需要知道lib文件夹中放的是codrmirror的核心JS和CSS，有点提一下，因为我刚从github上拉下来最新的代码时，lib中是没有codemirror.js文件的，在后面使用时才发现，很坑，没有codemirror.js文件完全用不了，所以我就去网上找了一个codemirror.js补充了一下，然后就是mode文件夹，该文件夹中放的是支持各种语言的语法定义，有我需要的python，但似乎好像没有java的，而theme文件夹下就是codemirror支持的各种主题样式

知道了上面这些，就可以轻松的实现一个python的在线编辑器，那么怎么做呢？很简单，直接找到mode/python/index.html

![codemirrorpython.png](http://obfs4iize.bkt.clouddn.com/codemirrorpython.png)

通过浏览器运行一下，出现下面的界面，人家已经帮我们写好一个python web IDE了

![codemirror运行](http://obfs4iize.bkt.clouddn.com/codemirror%E8%BF%90%E8%A1%8C.png)

从图中可以发现，codemirror支持两种python格式，一种是单纯的python格式，另一种是Cython，这个我没用过，不熟，直接看这个index.html的源码，然后将不要的删除，只留下Python mode部分就好了，到这里其实就一下实现了一个python web了，但是感觉有点丑，没有geek的感觉，所以简单的修改一下，改变大小，换个主题，具体代码如下

```
<!doctype html>

<title>Python Web IDE</title>
<meta charset="utf-8"/>

<link rel="stylesheet" href="codemirror/lib/codemirror.css">
<link rel="stylesheet" href="codemirror/theme/dracula.css"/>
<link rel="stylesheet" href="codemirror/addon/fold/foldgutter.css"/>
<script src="codemirror/addon/fold/foldcode.js"></script>
<script src="codemirror/addon/fold/foldgutter.js"></script>
<script src="codemirror/addon/fold/brace-fold.js"></script>
<script src="codemirror/addon/fold/comment-fold.js"></script>
<script src="codemirror/lib/codemirror.js"></script>
<script src="codemirror/addon/edit/matchbrackets.js"></script>
<script src="codemirror/mode/python/python.js"></script>
<style type="text/css">.CodeMirror {border-top: 1px solid black; border-bottom: 1px solid black;}</style>
<article>
<h2>Python Web IDE</h2>

<div><textarea id="code" name="code">
if __name__ == '__main__':
    print('Hello, ayuliao')
</textarea></div>

    <script>
    var editor = CodeMirror.fromTextArea(document.getElementById("code"), {
        mode: {name: "python",
               version: 3,
               singleLineStringErrors: false},
        lineNumbers: true,  //显示行号
        theme: "dracula", //设置主题
        lineWrapping: true, //代码折叠
        foldGutter: true,
        gutters: ["CodeMirror-linenumbers", "CodeMirror-foldgutter"],
        matchBrackets: true,  //括号匹配
    });

    editor.setSize('700px', '800px'); 
    
    </script>
  </article>
```

简单看看这堆代码，首先要使用codemirror，当然是要引入lib下的codrmirror.css和codrmirror.js

```
<link rel="stylesheet" href="codemirror/lib/codemirror.css">
<script src="codemirror/lib/codemirror.js"></script>
```

为了让codemirror可以让python代码语法高亮且可以自动缩进，需要将mode/python/python.js导入

```
<script src="codemirror/mode/python/python.js"></script>
```

然后换了个geek点的主题，dracula，所以就需要将theme/dracula.css导入

```
<link rel="stylesheet" href="codemirror/theme/dracula.css"/>
```

让编辑器可以支持代码折叠

```
<!--支持代码折叠-->
<link rel="stylesheet" href="codemirror-5.31.0/addon/fold/foldgutter.css"/>
<script src="codemirror-5.31.0/addon/fold/foldcode.js"></script>
<script src="codemirror-5.31.0/addon/fold/foldgutter.js"></script>
<script src="codemirror-5.31.0/addon/fold/brace-fold.js"></script>
<script src="codemirror-5.31.0/addon/fold/comment-fold.js"></script>
```

创建编辑器界面，其实就是一个textarea，里面可以写一些默认的代码片段

```
<textarea id="code" name="code">
if __name__ == '__main__':
    print('Hello, ayuliao')
</textarea>
```

接着就是将textarea变成Web IDE了，这段js最好放在最后，这样才能使用前面导入js文件中的内容

```
var editor = CodeMirror.fromTextArea(document.getElementById("code"), {
        mode: {name: "python",
               version: 3,
               singleLineStringErrors: false},
        lineNumbers: true,  //显示行号
        theme: "dracula", //设置主题
        lineWrapping: true, //代码折叠
        foldGutter: true,
        gutters: ["CodeMirror-linenumbers", "CodeMirror-foldgutter"],
        matchBrackets: true,  //括号匹配
    });

    editor.setSize('700px', '800px'); 
```

这段js比较简单，很多配置内容可以去codemirror上找，也可以之间翻看源文件，最后通过setSize()方法改变了一下编辑器的大小

最终效果如图

![](http://obfs4iize.bkt.clouddn.com/pythonwebide.png)

## cloud9
通过上面的简单操作已经实现了一个编辑器，但是功能还是不是特别强，因为没办法编译运行啊，虽然我昨天知道了codrmirror+Skulpt可以搭建一个简单的在线python编辑环境和运行环境，但是没有去尝试，就被cloud9给惊艳了，先上一个安装好的效果图

![cloud9](http://obfs4iize.bkt.clouddn.com/cloud9%E6%95%88%E6%9E%9C.png)

cloud9是基于node.js实现的一个Web IDE，没错，是一款IDE，而不是简单编辑器，它可以编辑调试运行python代码，在上面开发项目没有什么大毛病，虽然我没有深入使用过，但也可以体会出cloud9的不简单，cloud9除了支持Python的在线编辑运行外，还支持JavaScript、PHP、Go等语言，更牛掰的是它还支持数据库，包括MySQL、MongoDB、Redis、SQLite，因为它是国外公司开的，所以在国内直接访问使用它会显得比较慢，而且cloud9的注册强制要求添加个人的国外信用卡信息，抱歉，我木有，所以没法直接用，不过cloud9的在线开发环境是开源在Github上的，所以可以在自己的服务器上搭建一个cloud9，[Cloud9 Github](https://github.com/c9/core)

下面来看看如何安装它,其中涉及了一点Docker的知识

因为一步步的安装配置cloud9比较麻烦，不是我这种懒人喜欢的，有没有一键安装使用的啊？有，这就需要使用docker容器技术了，docker方面的内容我个人近期也在学习，以后会尝试分享些这方面的内容

因为cloud9安装配置都比较麻烦，所以有人(sapk大神)已经帮我们把cloud9所需要安装的配置的所有内容都安装配置好了，并生成了docker镜像让我们可以直接使用，不用再去配置一遍

首先在centos7上安装Docker，yum一句话安装

```
# yum install docker
```

安装完后，启动一下，并设置成开机自启

```
# service docker start
# chkconfig docker on
```

这里提一下，如果你是境内服务器，因为众所周知的原因，在你安装境外的一些docker镜像时会非常的慢，这时你可以使用国内的docker镜像仓库，加快镜像的下载速度，因为我的服务器在境外，就不配置了

安装配置完docker后，就可以安装Cloud9了，这里使用sapk大神的docker源，sapk的源使用量最大，星最多，应该是最靠谱的一个源了

```
docker pull sapk/cloud9
```

等待安装，安装完后，可以通过 docker images命令查看一下，出现下面内容就表示安装成功了

```
docker.io/sapk/cloud9   latest   0cd8ecd4fbbf   13 days ago   485 MB
```

安装完后，cd 到想运行的工作区的目录下，直接运行

```
docker run -d -v $(pwd):/workspace -p 8181:8181 sapk/cloud9 --auth user:123456
```

这条命令就会在你cd到的目录中创建名为cloud9的文件夹，这里会存放你编辑的代码文件

```
[root@izj6c1jjxlrlkc6d56dnl7z cloud9]# ls
env  test.py
```

然后就可以通过IP:8181来访问cloud9了

![](http://obfs4iize.bkt.clouddn.com/cloud9%E8%BF%90%E8%A1%8C%E6%88%90%E5%8A%9F.png)

但是这样只是安装了，想要运行相应编程语言，需要安装对应的环境，比如要运行python，就要安装python的环境，那么这种比较麻烦的工作也有人帮我们写好了对应的脚本，我们只需要将脚本下载下来运行一下，就可以安装好所有的环境了

```
git clone https://github.com/izuolan/env.git

cd env
//给予脚本运行权限
chomd +x run.sh
./run.sh
```

运行效果如下：

![web.png](http://onxxjmg4z.bkt.clouddn.com/web.png)

第一次运行你要安装基础软件包，所有选择0

然后你就可以安装python环境了，选择3

这样配置好一个在线的python开发环境了

## 结尾
随着对在线编辑的了解，发现越来越多的Web IDE，同时也再次体会到了Docker的强大之处，然配置环境的痛苦见鬼去吧，这里推荐一篇，通过Docker搭建在线Android开发环境，怎么说我以前也是搞Android的，哈哈

[Docker 实现浏览器里开发Android应用的功能](http://www.jb51.net/article/97221.htm)

编程创造乐趣---ayuliao

