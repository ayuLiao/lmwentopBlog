# python浅拷贝和深拷贝

在python中浅拷贝和深拷贝是比较容易混淆的内容，虽然平时好像用不到，但是一不小心用到了却不知道就会掉进大坑里，所以本节深入的讨论一下python中的浅拷贝、深拷贝，主要让大家可以回答下面几个问题：

+ 1.在python中，直接赋值有什么特征？
+ 2.什么是浅拷贝？什么是深拷贝？
+ 3.如何使用浅拷贝？如何使用深拷贝？
+ 4.两种拷贝有什么特征？

## 直接赋值
在python中，直接将一个变量赋值给另外一个变量，其实就是让新变量指向了旧变量的内存地址，通俗的说，新变量就是旧变量的别名

通过代码简单验证一下，首先创建ayu这个list类型的变量，让该变量赋值给byu变量，然后通过id()方法判断一下两个变量的内存地址是否相同

```
In [1]: ayu = ['abc',123,['I','Love','You']]

In [2]: byu = ayu

In [3]: print(id(ayu))
4536573832

In [4]: print(id(byu))
4536573832

In [5]: print([id(i) for i in ayu])
[4507311944, 4504983360, 4538250504]

In [6]: print([id(i) for i in byu])
[4507311944, 4504983360, 4538250504]
```

代码非常简单，可以看出通过id()方法获得两个变量的内存地址是一致的，都是4536573832，迭代ayu和byu，使用id()方法获取其中变量的内存地址，发现也是其内存地址也是一致的，这也就说明，ayu其实就是byu，两者操作的同一个东西

**在python中，我们可以通过对象的内存地址、对象类型、对象存储的值来标识对象的唯一身份，所以多个对象的内存地址相同并不一定表示这些对象是同一个对象**，这句话似乎与上一句冲突，其实不然

比如我们修改ayu这个list中第一个元素的值和第三个元素的值，看看其中的变化

```
In [7]: ayu[0] = 'new_abc'

In [8]: ayu[2].append('ayuliao')

In [9]: print(id(ayu))
4536573832

In [10]: print(id(byu))
4536573832

In [11]: print([id(i) for i in ayu])
[4538413328, 4504983360, 4538250504]

In [12]: print([id(i) for i in byu])
[4538413328, 4504983360, 4538250504]
```

看到上面代码相应的输出，我们改变了list中的第一个元素，将abc修改成了new_abc，同时改变了list中第三个元素，为其添加多了ayuliao这个元素

通过上面的修改，发现ayu和byu的内存地址与没有修改前一致，没有发生变化，但是当遍历ayu和byu时，发现list中第一个元素的内存地址由4507311944变成了4538413328，但是同样修了的第三个元素其内存地址依旧是4538250504，没有发生改变

这样的变化其实并不奇怪，在python中list是可变对象，它相当于一个容器，容器内存可以存东西，容器内部存储内容的改变并不影响整个容器在内存中的位置，也就是整个list虽然其中存储的内容在改变，但是该list的内存地址是不会改变的，但是像int、str等不可变对象，当它发生改变时，内存地址就改变了。

可是在编写python代码时，这些不可变对象依旧可以赋值，在操作上给人直观的感受是可变的，但是当这些不可变对象改变内容时，其实将新的内存存放在了新的内存中，而以前存放的内容依旧还在旧的内存中，而变量指向了新的内存，通过下面代码理解一下

创建变量a，将变量a赋值给变量b，然后修改变量a的值，将python改为java，可以发现变量a的内存地址改变了，但是python这个字符串本身的内存地址依旧是没有改变的，可以看变量b的内存地址，发现是没有改变的，说明str类型是不可变的，**不要将变量b认为是str类型对象本身，变量其实只是一个存储内存地址的空间，这个空间内的内存地址是可以改变的**

```
In [13]: a = 'python'

In [14]: b = a

In [15]: print(id(a))
4508235120

In [16]: print(id(b))
4508235120

In [17]: a = 'java'

In [18]: print(id(b))
4508235120

In [19]: print(id(a))
4507409632
```



