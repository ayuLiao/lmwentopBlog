## 简介
上个月，也就是7月，对python开发者来说是一个好日子，因为Python终于将第一语言的位置给拿下了，将常年位居第一个的Java挤到第二位，正如一篇博文中描述的场景，在语言圆桌会议上，Python眼神慵懒、身体轻微的靠在椅子上，看着桌上其他的语言，说‘对不起！我不是针对谁，我是说，在座的各位都只能挣第二’

但是Java这么多年老大也不是白当的，Java依旧在工业界大量的被使用，它有丰富的第三方组件库为Java提供全面而且强大的功能。

因为项目原因，我不得不使用Python来调用HanLP语言工具包，而HanLP是通过Java来实现的，所以我就必须要通过Python来调用Java

## python调用Java的方式
上网搜索Python调用Java的方法，大致有五种，分别是使用Pyjnius、JCC、Javabridge、JPype和Py4j

下面来分分别介绍一下，Pyjnius和Jpype会讲细一点，因为这两个库使用过，其他三个没有实践过，就大致的讲讲

### Pyjnius
Pyjnius是一个比较活跃的python调用Java库，项目在github上进行了开源，也是一个比较新的库

[PYjnius Github地址](https://github.com/kivy/pyjnius)

这个库有着比较完善的文档，当然都是英文的，国内没有关于该库讲的比较深的问题

可以直接通过pip来安装这个库，要注意的是，这个库依赖于Cython库，所以在安装pyjnius前，要将Cython通过安装上

具体如果使用可以先自行阅读一下[官方文档](https://pyjnius.readthedocs.io/en/latest/)

在使用pyjnius前，你必须告诉pyjnius你系统中jdk的位置，它才能使用jvm来解析java代码

官网中是这样写的(当然这是针对windows的，写这篇博文时，我使用的windows10)
![pyjnius配置.png](http://obfs4iize.bkt.clouddn.com/pyjnius%E9%85%8D%E7%BD%AE.png)

跟配置java环境变量一样，简单的配置一下就好了
![添加环境变量.png](http://obfs4iize.bkt.clouddn.com/%E6%B7%BB%E5%8A%A0%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F.png)

这里要注意的是，**文件路径的中最后的斜杠不要忘了**，不然pyjnius就会报找不到JVM的错误

配置完后，就可以直接进行简单的使用了

![使用pyjnius.png](http://obfs4iize.bkt.clouddn.com/%E4%BD%BF%E7%94%A8pyjnius.png)

从图中可以看出，使用pyjnius时，导入的是jnius这个名字，然后使用其中的autoclass方法，加载java中的类，这里加载 java.lang.System 这个类，这里加载了java JDK中自带的java类，有点java基础的人都会感觉熟悉，就是用于输出的

我们可以看，当我们输入 System.out 时，返回的一串内容，包含的信息就有这个java所属的类，具pyjnius官方文档介绍，让python中显现java中的类，是通过java的发射原理实现的，发射在Android中也常用获得类中变量或值

最后通过System.out.println('hello ayuliao')输出一句话

输出方式跟Java一样，其实可以简单的理解为，我们通过autoclass将这个Java类导入到python中来了，也就是代码中System变量，该变量具有加载的java.lang.System类的所以方法和属性

为了更好的理解，这里我们来调用HanLP这个第三方Java工具包

要加载第三方的库，官方文档中也提及

![pyjnius加载第三方库.png](http://obfs4iize.bkt.clouddn.com/pyjnius%E5%8A%A0%E8%BD%BD%E7%AC%AC%E4%B8%89%E6%96%B9%E5%BA%93.png)

但是通过研究，有三种方法来加载，在官方上我暂时只看到一种

1.将.class文件打包成jar，然后将CLASSPATH指定jar的路径
2.将.class文件路径指向CLASSPATH
3.通过jnius.config修改CLASSPATH（官方文档提及的方法）

我常使用第一种方法，通过下面简单使用来理解

![pujnius](http://obfs4iize.bkt.clouddn.com/pujnius.png)

从代码中，可以简单看出，首先引入os库，然后将CLASSPATH变量的路径改为我们要加载的HanLP工具包的路径，修改完后，就直接引入jnius下的autoclass，然后将该第三方库中我们要使用的类加载进来，这里加载 com.hankcs.hanlp.HanLP 类，通过该类就是实现分词、关键词抽取等NLP相关的功能

使用也分词简单，Java中这么使用这个类中的方法的，这里就怎么使用，这里通过其中的segment方法获得句子的分词，分词结果的类型是java.util.List类型，那么我们就可以使用java.util.List类型中的任何方法，如size()方法获得list的长度，get()方法获得其中的内容

那么我们可以通过for循环来获得其中的内容，但是直接通过for循环来获得List中的内容是不可行的（有些又好像可以）返回的依旧是HanLP中定义的类型

![接着for取值.png](http://obfs4iize.bkt.clouddn.com/%E6%8E%A5%E7%9D%80for%E5%8F%96%E5%80%BC.png)

如何获得分词后的值呢？

可以阅读HanLP的源码，来看看，HanLP中是怎么使用分词结果的

![HanLP分词结果使用.png](http://obfs4iize.bkt.clouddn.com/HanLP%E5%88%86%E8%AF%8D%E7%BB%93%E6%9E%9C%E4%BD%BF%E7%94%A8.png)

发现它直接使用System.out.println来输出，所以我们也可这样做
通过autoclass导入java.lang.System类，然后使用它来打印分词结果

![HanLP正常显示.png](http://obfs4iize.bkt.clouddn.com/HanLP%E6%AD%A3%E5%B8%B8%E6%98%BE%E7%A4%BA.png)


到这里pyjnius的基本使用就讲完了，因为没怎么深入研究，所以还不知道怎么在调用第三方方java工具包时，同时导入它需要的配置文件，大致看了一下文档，没怎么发现，如果你熟练使用pyjnius，可以gmail告诉我

这里讲讲pyjnius我个人觉得有点欠缺的地方

1. pyjnius调用java中的方法后，返回的类型是java中的类型而不是python中的，如java.util.List这里的java类，那么就无法通过python来直接操作这些类型，文档中似乎是使用重写的方式来实现返回python类型，具体在文档中的[pyjnius API](https://pyjnius.readthedocs.io/en/latest/api.html)，其实我没怎么看懂

2. pyjnius如果使用第三方Java工具包，似乎没有方法可以直接将该Java工具包所需的配置文件传入

优点：
1. Github 讨论氛围很不错，作者也积极回答使用者的问题，可以看出这个项目是很活跃的项目
2. 如果不考虑什么各种复杂的调用方式，Pyjnius是一个简单的实现方式

### JCC
JCC是一个C++代码生成器，它通过C++作为中间层，让Python通过调用C++提供的相应接口来调用Java

JCC会通过Java的Native Interface（JNI）来生成C++对象接口，生成的这些接口符合Cpython这个解释器，这样Python就可以通过C++接口来调用Java了，但这需要编译所有可能的调用C++代码（C++是编译后才可运行的代码）

[JCC 介绍](http://lucene.apache.org/pylucene/jcc/index.html)

[JCC 下载](https://pypi.python.org/pypi/JCC/)

JCC我没用过，Google查了其使用方法，Python JCC Document 为关键词，结果没有发现

可能没有了解JCC，但是直觉告诉我，它可能很毒，所以不推荐大家使用

### Javabridge
让Python包可以轻松地从Python启动Java虚拟机（JVM）并与之交互。Python代码可以使用低级API或更方便的高级API与JVM进行交互。

没用过 :)

### Jpype
这个库算是我使用最多的库了，用Jpype来调用Java非常简单，重要的是，它的返回类型可以直接通过python使用，这点与pujnius有比较大的不同

可以简单的使用一下

![jpype](http://obfs4iize.bkt.clouddn.com/jpype.png)

这里通过getDefaultJVMPath()获得系统中JVM的路径，然后通过startJVM()来开启JVM，这样就可以解析Java代码了，这里我简单的输出Hello ayuliao

需要注意的时，当我们使用完了，就要将其关闭，避免浪费系统资源

可以看出Jpype其实非常好用，但是这个库比较老了，而且Github并不活跃，此前还查找过Jpype社区不活跃的原因，原来是作者都不更新了，Jpype库最新的是2013的版本，0.5.4这个版本，此前我在window7上使用Jpype一点问题都没有，但是在window10上，就呵呵

我电脑升上了window10，然后就被Jpype毒了一个下午

以前可以正常运行的代码，现在出现这样的问题，而且只给我一个内存码，我内心是崩溃的

![jpype报错](http://obfs4iize.bkt.clouddn.com/jpype%E6%8A%A5%E9%94%99.png)

我不信，写段最简单的使用Jpype代码试一下

![python直接崩溃](http://obfs4iize.bkt.clouddn.com/python%E7%9B%B4%E6%8E%A5%E5%B4%A9%E6%BA%83.png)

好吧，python直接崩溃，Very interesting!

遇到问题，首先让自己不要慌，Google一下，找到Jpype的Github，发现大家在window10上使用都有这样的问题，可是，似乎都没解决。

我将讨论里的所有方法都尝试了

安装各种dll支持库			 ---- 失败
Python32+JPype32+JVM32       ---- 程序崩溃
Python32+Jpype64+JVM64/JVM32 ---- 程序崩溃
Python64+Jpype64+JVM64       ---- 程序崩溃

看来Jpype不支持window10，想想也是，2013年的东西，一直没有更新

但是经验告诉我，还有一招

首先，通过pip将jpype卸载掉

此前使用的Jpype都是通过pip install Jpype1来安装的

![卸载pip安装的JPype.png](http://obfs4iize.bkt.clouddn.com/%E5%8D%B8%E8%BD%BDpip%E5%AE%89%E8%A3%85%E7%9A%84JPype.png)

现在我们去LFD这个第三方网站下载Jpype

[LFD](http://www.lfd.uci.edu/~gohlke/pythonlibs/)

这里注意，**使用32位的Jpype还是64位的Jpype取决于你的Python是32位还是64位的**

所以这里我下载32位的Jpype，JVM也使用32位的

![去LFD下载Jpype.png](http://obfs4iize.bkt.clouddn.com/%E5%8E%BBLFD%E4%B8%8B%E8%BD%BDJpype.png)

使用这里的Jpype，需要Numpy库的支持，后面我运行后，报错提示我的，所以这里你先装上

看了这里的Jpype被修改过了，这个网站果然良心

![去LFD下载Numpy.png](http://obfs4iize.bkt.clouddn.com/%E5%8E%BBLFD%E4%B8%8B%E8%BD%BDNumpy.png)

下载完后，通过通过pip将其安装都虚拟环境中，安装完后，就可以在window10上正常使用Jpype了

### Py4J
Py4j是个很强的库，但是我还没研究

为什么说它强，因为Jpype作者不搞Jpype了，说Jpype底层设计就有一些问题，懒得从头再改，反而弄了个Py4J

Py4J的社区就非常活跃了，而且文档也比较全面

Py4J同样可以通过Python来直接使用Java中的一些数据类型，比如Java中的集合

[Py4J官方文档](https://www.py4j.org/)

