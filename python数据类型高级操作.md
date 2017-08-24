---
title: python数据类型高级操作
date: 2017-05-26 09:15:30
tags: python
---

## python数据类型基础

python中常见的数据类型有list(列表)、tuple(元组)、dict(字典)和set，当然我们可以引入一些库，使用这些库中的数据类型，方便我们实现一些特殊的需求，如可以映入collections库的namdtuple类，来实现一个带名字的元组。

下面来看一下list、tuple、dict和set这四个数据结构一些基础用法

### 安装IPython

当我们编写python程序时，想使用一些自己不熟悉的语法，想写一些小代码来测试自己的想法时，可以使用IPython这个工具来编写代码，该工具不适合编写一下比较大型的python项目，但是对于平常开发，还算很OK的。

IPython是Python的交互式Shell，提供了代码自动补完，自动缩进，高亮显示，执行Shell命令等非常有用的特性。

特别是它的代码补完功能，例如：在输入**list.**之后,按下Tab键，IPython会列出zlib模块下所有的属性、方法和类。完全可以取代python自带的bash。

![](http://onxxjmg4z.bkt.clouddn.com/%E4%BD%BF%E7%94%A8ipthon.png)

从上图可以看出，直接在命令行输入ipthon，就可以进入ipython进行python代码的编写了，哦！忘了说如何安装了，没错，已经可以直接通过apt-get命令来安装IPython

![](http://onxxjmg4z.bkt.clouddn.com/%E5%AE%89%E8%A3%85ipthon.png)

安装完了相应的工具，就正式来看看python中基础数据类型的基础用法

### list基础用法

list是python内置的一种数据类型，它是一个有序的集合，里面可以放不同类型的元素，可以通过list类型自带一些方法轻松的使用元素的添加和删除

可以通过两种方式来创建一个list，一个是直接创建、一个是通过list()这个工厂函数

![](http://onxxjmg4z.bkt.clouddn.com/list%E7%9A%84%E4%BD%BF%E7%94%A8.png)

从图中可以看出，我们可以通过append()方法在list尾部追加数据，或者通过insert()方法将元素添加到list对应的下标中

接着我们通过pop()将s1这个list中的元素给删除，如果pop()方法不传参数，就会删除list最后的那个元素

![](http://onxxjmg4z.bkt.clouddn.com/pop%E6%96%B9%E6%B3%95.png)

我们还可以通过len()方法获得list对应的长度，如果list中没有元素了，len()方法会返回0

```python
In [1]: len(s1)
Out[1]: 3
```

当我们想获取list中的一些数据时，还可以通过切片的方式来获取，切片操作获取第一个参数，不获取最后一个参数

![](http://onxxjmg4z.bkt.clouddn.com/Selection_002.png)


### tuple基础用法

Python中除list外，还有另一种有序的集合，它叫tuple，元组。tuple和list非常类似，但是**tuple一旦初始化就不能修改**，这个特性非常重要。

创建元组同样可以直接创建和通过tuple()这个工厂函数创建，不过很多时候通过tuple()函数来创建一个元组意义不大，因为该函数创建的元组时不会存值，而元组在初始化后又不能修改，所以就是创建了一个空元组

在直接创建时，跟list一样，但是在只有一个数时
![](http://onxxjmg4z.bkt.clouddn.com/Selection_003.png)

元组虽然不能修改其中的元素，但是可以对整体进行+或×的运算，+运算可以将多个元组合并成一个新元组，×运算可以将一个元组复制成一个新元组

![](http://onxxjmg4z.bkt.clouddn.com/Selection_004.png)

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%A4%9A%E7%89%88%E6%9C%ACSelection_002.png)

在元组中，同样可以使用切片操作，操作方式和效果都和list中一样，说到list，我们可以通过tuple方法将一个list转成一个元组

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%A4%9A%E7%89%88%E6%9C%ACSelection_003.png)

如果想要获得元组中的最大值和最小值，可以通过max()方法和min()方法

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%A4%9A%E7%89%88%E6%9C%ACSelection_004.png)

如果想要比较两个tuple的大小，在python2中可以使用cmp(tuple1,tuple2)函数进行比较，但是在python3中，cmp()函数已经被舍弃掉了，所以要比较两个tuple，可以使用operator模块，operator模块是python中的内置的操作符函数接口，通过c实现，所以执行速度比一般的python代码快，通过该模块中的一些方法可以轻松实现不同数据类型之间的比较，其中就可以比较不同tuple，如下图

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C9.png)

其中lt,le,eq,ne,ge,gt分别表示等价于<、<=、==、!=、>=和>的表达式语法,operator还可以用于比较不同的list

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C10.png)

在实际的项目中，代码量比较巨大，如果使用了大量的元组，却只通过下标来访问元组中的元素，久了就会忘记相应下标对应值的意义，这样就显得不太方便，我们可以像list那样，使用具有一定含义的命名作为下标，这里可以通过2中方法来实现，一种就是直接为这些下标数字附上意义

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%A4%9A%E7%89%88%E6%9C%ACSelection_005.png)

通过上面的方式，我就清楚的知道了对于下标存的值所拥有的含义，这样就提高的代码的可读性，接着来看第二种方式，使用namedtuple，namedtuple这个数据容器可以实现带名称的下标，这样代码就可以一目了然

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%A4%9A%E7%89%88%E6%9C%ACSelection_006.png)

从图中可以看出，namedtuple()方法需要传入两个参数，分别表示要定义的元组类型（People）和该类型所包含的属性（name,sex,age,phone），我们可以通过最简单的方式为People类型的元组赋值，也就得到了p1，当然也可以直接使用合适的list，通过_make()方法来得到一个元组，也就是p2，获得这些元组后，我们都可以通过点下标的方式获得该下标对应的值

获得一个namedtuple后，还可以通过_replace()方法修改其中的值，嗯？？元组不是不可以修改其中的值吗？没错，使用_replace()方法的本质其实是重新获得一个namedtuple，将原来那个namedtuple给覆盖

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%A4%9A%E7%89%88%E6%9C%ACSelection_007.png)

我们修改了岁数，将青葱岁月的妹子变成了满脸沧桑的大婶，哎，叹息岁月之无情！注意，**p2的值并没有改变**

顺带说一下，可以通过_adict()方法轻松的将namedtuple转成OrderedDict，也就是字典

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%A4%9A%E7%89%88%E6%9C%ACSelection_008.png)


### dict基础用法

dict同样是python内置支持的一种数据类型，dict全称dictionary，使用key-value形式来存储相应的数值，这种数据结构具有很快的查找速度，它同样可以直接创建或通过工厂函数来创建

```python
d1 = {}
d2 = dict()
```

在初始化dict时，容易犯一个错误，如下

```python
key = 'name'
d1 = {key : 'value'}  # ----> 结果为 {'name' : 'value'} 正确
d2 = dict(key = 'value') # ----> 结果为 {'key' : 'value'} 错误
```

其实dict还可以通过list来创建，此时就需要fromkeys函数的帮助了

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C11.png)

我们通常使用这种方法来对dict进行初始化

在dict中，虽然可以直接使用中括号的方式来获得该key对应的值，写法如下，但是不推荐这种写法

```python
d1['a']  # ---->获得key为a对应的value
```

推荐使用get()方法来获得dict中key对应打value，使用get()方法的好处在于，就算dict中没有这个key，程序也不会抛出错误，而是返回一个空，还可以在get()方法中设置一个默认的值，当get()找不到该key时，就会返回默认的值，增加代码的友好度

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C12.png)

dict是可变的，我们可以随时往dict中添加新的key-value，如果key已经存在，新赋的值就会替换掉原来的value

若要删除dict中的key，可以使用pop(key)方法，传入key，那么对应的key-value就会从dict中删除

```python
d1.pop('a')
```

使用dict时，需要注意几点

1.dict查找的速度虽然快，但是会占用大量的内存
**2.dict中的key必须时不可变对象**

key必须为不可变对象，不然就会报错

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C13.png)

这是因为，dict中，查找key对应的值，是通过对key进行相应的运算，得出一个内存地址，该地址就存放着对应的value，其实就是哈希表的实现原理，如果key都是可变的，那每次使用同一个key进行运算，都会得到不同的value，这显然是不正确的，所以为了保证dict内部不发生混乱，就规定了key必须是不可变对象

最后讲一下同时迭代key和value，其实也简单，就是使用iteritems()

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C14.png)

### set基础用法

set和dict类似，这种类似表现在set的原理和dict几乎一样，其中同样只可以存放不可变对象，来保证set中没有重复的值，可以将set看成只有key的dict

**在set中是不可以存可变对象的，因为它要通过对元素的运算来达到set内部不会有重复元素的目的**，同时还要注意，set和dict一样都是无序的

创建一个set需要一个list作为输入，这里可能就懵了，不是说，set中不可存可变对象吗？list是一个可变对象啊，同学，神目如炬啊

输入的list只是告诉set，其中有什么数据，因为set不能直接输入要存入的数据

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C15.png)

所以要通过list来存入数据，但是这只是一种规定写法，并不是说明set中可以存可变对象，我们尝试存一下就知道了，如下

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C16.png)

报错信息很明显的告诉你是不能这样创佳set的

但是你可能会发现setky存一个变量，其实set并没有存该变量，set会将变量对应的数值取出来存入set中（这个数值是不可变对象）

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C17.png)

我们可以通过add()方法来添加set中的元素，重复添加不会报错，但是不会有效果，还可以通过remove()来删除set中的元素，如果set中已经没有该元素来，还用remove()方法来删除，就会报错，其实使用add()或remove()方法时已经隐含的做了判读一个元素是否在set中的操作了。

因为set是无序的，而且不像dict有key作为索引，所以在访问set中某个值时，就判断来该元素是否在set中了

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C18.png)

set还可以看成数学意义上的无序、不重复的集合，我们可以对set做数学意义上打交集或并集

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C19.png)

当我们在遍历set时，因为set是无序的，所以遍历得到的顺序很可能跟输入的顺序不同，而且，在不同的机器上遍历结果也可能不同


### 优雅操作

有了list、tuple、dict、set的基础知识，可以来看看更加优雅的用法

如使用合适的模块、列表生成式、函数编程等

1.如何在列表中根据相应的条件筛选出合适的数值？

最基础的想法就是遍历该列表，然后对其中的每个元素都进行判断，符合要求的就添加进一个新列表，最后将该列表输出，这样写的逻辑没有错误，但是人生苦短，我们需要使用更优雅的写法，这里使用列表生成式的写法，这种方法速度几乎是简单通过遍历判断的2倍

这里获得列表中的偶数

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C20.png)

首先我们引入random模块来获得一个随机的list，然后通过列表生成式的方式来获得list中符合条件的值

那如何在字典（dict）中根据相应的条件筛选出合适的数值

其实路数和上面的一样

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C21.png)

同样的问题，换到set数据类型上，同样，你仔细观察就会发现，跟list操作的唯一不同，就是在写生成式时，有中括号变成了大括号

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C22.png)


2.如何获得英文文章中出现频率最高的5个单词？

这个问题可以分解成两个问题，一个是如何将文章中的单词分割出来存放到相应的数据结构中？另一个是如何得到该数据结构中出现最高的5个元素？

很多人看到第二个问题，就会想到循环累加，循环数据结构中的元素，出现一次，相应的值累加一次，但是这样太麻烦，其中一种优雅的解决方法就是使用collections模块的Counter，Counter就是解决该类问题的

对于第一个问题，我们可以使用re模块，利用正则表达来切割出其中的单词

首先来看一下我们要统计词频的文件，这是我写的一篇介绍中国古镇旅游的英文文章

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C24.png)

首先我们分别引入collections模块打Counter和re，然后通过open()方法打开相应的英文文章，接着使用re中的split()方法来分割出单词，其中传入一个正则表达式，这里使用\W+表示以非字母为分割标识，然后将获得的值传入Counter()方法，这样就获得来这篇英文文章所有单词对应出现的频率，因为要求是出现频率最高的5个词，所以还需要使用most_common()方法，传入5表示频率最高的5个

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C23.png)

3.如何对字典进行排序？

因为字典是通过哈希计算来获得key对应的值的，所以查询速度非常快，我们喜欢用它来存储一些要经常查询的数据，但是dict内部是无序的，可是有时我们又需要有序打印出来，我们该如何优雅的对字典进行排序？现实中的问题如高考，数据量比较巨大，一个学生通过身份证来查询自己打成绩，如果使用list进行遍历，对服务器的负担比较大，而且查询速度也慢，用好体验不好，此时使用dict就是一个好选择，但是官方还需要公布前100的排名，那么就要对dict进行排序了

这里我们可以使用sorted()方法对dict进行排序，sorted()内部是通过c实现的，运行速度比原生的python要快，而且sorted()方法作为python内置的函数，其排序算法已经经过了各大高手的优化，比自己实现的排序算法要优秀，但是大家都知道sotred()是对list进行排序的，如图

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C25.png)

其实sorted也可以对dict进行排序

首先我们构造出一个随机的dict，前面的表示学生的学号，后面的表示成绩

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C26.png)

如果直接使用sorted()对dict进行排序，那么sorted就只会比较dict中打key，也就是学号，这样明显是不对的

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C27.png)

我们可以为sorted()方法的key参赛传入一个值，key参数的作用就是让sorted()根据哪个值进行排列，这里要使用匿名函数的方式，指定使用value进行排序，x下标从0开始，x[0]表示key，x[1]表示value，这样就可以得到正确的答案了

![](http://onxxjmg4z.bkt.clouddn.com/python%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C22.png)


python数据类型还有很多优雅的操作，这篇就暂时讲到这里