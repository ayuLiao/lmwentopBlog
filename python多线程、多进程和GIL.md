---
title: python多线程、多进程和GIL
date: 2017-07-17 10:16:38
tags: python
---

## 简介

不知道你是否听过在python中无法实现真正的多线程？

通过这个问题，我们来全面的了解一下python中的多线程和多进程以及造成问题的GIL和相关的解决方法

## 线程和进程

线程和进程是操作系统中的概念，比较抽象，我以个人的理解先来解释一下

当我们启动一个程序的时候，相当于启动了**至少一个**进程，也就是说，一个程序至少有一个进程，这里要对程序有个理解，程序其实是静态资源实体，一堆指令(一堆代码)，它是静态的，本身没有任何运行的含义，而进程是动态的，它是程序操作某个数据集时的动态实体，程序创建它时，它就产生了，然后被调度运行，然后对资源进行操作，也就是说进程反应了程序在一定数据集上运行的动态过程，**不同的进程之间是不共享内存空间的**。

一个进程可以包含多个线程，这些线程共同完成该进程的任务，如一个进程要完成任务A，任务A有可分为任务a，任务b，任务c，那么a,b,c这三个小任务就交给进程中线程来完成，如开3个线程来完成a,b,c三个小任务。**线程可以共享属于同一个进程的其他线程的内存空间，也就是一个进程中其内部的所以线程是共享该进程的内存空间的**。

线程之间使用共享的内存空间也是有规则的，如互斥锁机制和信号量机制。

所谓互斥锁就是一个线程在使用一个内存空间时，它拿着一把锁，将这个内存空间给锁起来，其他线程要等待它使用完后，将锁解开才可以使用，使用完的线程将这个互斥锁给谁，谁就可以接着使用该内存空间，其他线程要继续排队

所谓信号量，就是有些内存空间运行多个线程使用，如允许同时4个线程使用，其他线程要使用该内存就要等待其中一个线程使用完成出来后才可以接着使用，那么就需要一个控制的变量，让其他线程知道内存已经被占用完了或者有人使用完这段内存我可以使用了，这就是信号量，如该内存空间允许4个线程使用，那么就在该内存外放4把钥匙，拿到钥匙的线程才可以进入该内存空间执行任务，当执行完后，就把钥匙放回原位，这样其他线程就可以拿钥匙了


可以线程和进程总结一下
1. 一个程序至少有一个进程，一个进程至少有一个线程
2. 进程在执行过程中拥有独立的内存单元，进程中的多个线程可以共享该内存单元
3. 线程使用共享空间时有相应的协调机制

为了加深理解，避免文章出错，我还查阅了线程和进程相关的资料，总结一下

1. 对于一些要求**同时进行**并且又要**共享某些变量**的并发操作，只能用线程，不能用进程。
2. 线程不能够独立执行，必须依存于进程
3. 每个独立的**线程**有一个程序运行的入口、顺序执行序列和程序的出口
4. 进程之间是有层级关系的，而一个进程中的线程是平级的
5. 线程间可以通过共享变量(shared variables)来通信与协调。共享变量相对于消息传递(message passing)和网络(network)来说代价较低。进程间通信主要通过消息传递和网络
6. 进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程产生影响；线程只是一个进程中的不同执行路径，线程有自己的堆栈和局部变量(在运行中必不可少的资源)，但线程之间没有单独的地址空间，一个线程死掉就等于整个进程死掉
7. 线程是调度的基本单位 进程是资源分配的基本单位

加深理解可以继续看下面两篇文章

[线程和进程的区别](https://software.intel.com/sites/default/files/m/5/7/f/a/b/12568-2.1.1_e7_ba_bf_e7_a8_8b_e4_b8_8e_e8_bf_9b_e7_a8_8b_e7_9a_84_e5_8c_ba_e5_88_ab.pdf)

[线程和进程的区别是什么？](https://www.zhihu.com/question/25532384)


## Python使用多线程

Python中可以使用标准库中的**threading**模块来创建多个线程，Python中提供了两个模块，分别是_thread和threading，其中threading对_thread进行了封装，所以绝大多数情况下，Python创建多线程直接使用threading模块，下面来看两种创建方式

1.继承threading.Thread类，重写它的run方法，代码如下

```
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

import threading, time, random
sum = 0 
class AyuThread (threading.Thread):
	# lock 锁对象  threadName 线程名
	def __init__(self, lock, threadName):
		super(AyuThread, self).__init__(name = threadName)
		self.lock = lock
		

	def run(self):
		global sum
		self.lock.acquire()
		for i in xrange(1000):
			sum += 1
		print('thread %s >>> %s' % (threading.current_thread().name, sum))
		self.lock.release()

lock = threading.Lock()
for i in xrange(5):
	AyuThread(lock, 'AyuThread-'+str(i)).start()
# 等待一下子线程的循环
time.sleep(2)
print sum
```

在代码中，我们创建了AyuThread类，该类继承了threading.Thread，重写了父类的run()方法，该方法就是子线程要执行的具体逻辑，这里我们对全家变量sum进行累加，这里注意在方法中使用全局变量时要使用global关键字，不然会被认为是局部变量，这种做法不只是出现在python中，很多其他语言也具有这样的写法，同时在AyuThread类中，我们重写了__init__()方法，在该该方法中**显示的调用了父类的初始化方法**，这很重要，因为最终我们还是要通过threading.Thread这个类来创建线程

注意到在run()方法中我们使用了锁机制，这样使用锁是因为我们创建了5个线程都对全家变量sum进行累加操作，如果没有锁，就会产生数据不同步的问题，也就是最终输出的sum可能不是5000，从这里我们也可以感受到，线程之间使用共享内存空间进行通信虽然比进程之间的通信简单，但是要自己做好共享资源的控制，不然会得到意想之外的结果

代码运行结果如下

![threadingThread多线程1.png](http://obfs4iize.bkt.clouddn.com/threadingThread%E5%A4%9A%E7%BA%BF%E7%A8%8B1.png)

2.直接创建一个threading.Thread对象，在它的初始化函数（__init__）中将可调用对象作为参数传入

代码如下：

```
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

import threading, time, random
sum = 0 
lock = threading.Lock()

def AyuAdd():
	global sum, lock
	lock.acquire()
	for i in xrange(1000):
		sum += 1
	print('thread %s >>> %s' % (threading.current_thread().name, sum))
	lock.release()

for i in xrange(5):
	threading.Thread(target = AyuAdd, args = (), name ='Ayu2Thread'+str(i)).start()

time.sleep(2)
print sum
```

我们创建了AyuAdd()方法，它的作用依旧是累加sum这个全局变量，同样适用了锁机制，保证多个线程之间的数据同步，然后我们将AyuAdd()方法传递给threading.Thread对象，作为初始化函数，再调用start()方法启动线程

那么我通过两个方法都创建了多线程，那么为什么会有人说Python中无法实现真正的多线程呢？先保留这个问题，接着讨论，后面就好明白

其实threading.Thread类中提供了很多方法供我们调用来优雅的控制线程，我常用的有下面结果

Thread.join([timeout])

Thread.join()方法用于阻塞当前线程，只有等待调用了join()方法的线程执行完毕后，当前线程才可以继续执行，通过代码来直观理解

```
# -*- coding: utf-8 -*-
from __future__ import unicode_literals
import threading,time

def AyuWait():
	print '我是子线程',time.strftime('%H:%M:%S')
	time.sleep(2)
	print '我睡了两秒',time.strftime('%H:%M:%S')

t1 = threading.Thread(target = AyuWait,args = (), name='Thead-t1')
print '我是主线程'
t1.start()
time.sleep(1)
t1.join()
print '等死我了'
```

结果如下
![join方法.png](http://obfs4iize.bkt.clouddn.com/join%E6%96%B9%E6%B3%95.png)

可以看到，“等死我了”在子线程Thead-t1执行完后才被执行，也就是说当前线程（主线程）只有等待调用了join()方法的线程(Thead-t1线程)才能继续执行

threading.Condition
通过Condition我们可以控制复杂的多线程数据同步的问题，这个后面的博文再讲

还有很多方法可以看下面的博文
[Python模块学习：threading 多线程控制和处理](http://python.jobbole.com/81546/)


## Python使用多进程

在Linux/Unix的系统内核中会提供fork()函数，该函数就是用于创建进程的，很多高级编程语言都可以调用fork()函数创建进程，Python当然在其中，用法非常简单，引入os库，然后直接调用os库中的fork()方法则可

那在windows下怎么破？可以使用multiprocessing模块(跨平台)，该模块提供一个Process类来表示进程对象，我们可以使用Process来创建一个新进程

```
# -*- coding: utf-8 -*-
from __future__ import unicode_literals
import os
from multiprocessing import Process

def AyuProc(name):
	print '子进程 %s (%s)' % (name, os.getpid())

if __name__ == '__main__':
	print '主进程id: %s' % os.getpid()
	p = Process(target = AyuProc, args=('Proc-1',))
	print '创建子进程'
	p.start()
	p.join()
```

看上面的代码，是否感觉跟创建线程时很相像，**multiprocessing模块支持使用类型threading模块的API生成进程**

这里要注意的是，windows上使用multiprocessing模块创建新进程时，将**创建和启动进程的代码通过 if __name__ == '__main__': 包裹起来**，保证Python解释器能安全的装载主模块，不然会报如下错误

![要在main函数中.png](http://obfs4iize.bkt.clouddn.com/%E8%A6%81%E5%9C%A8main%E5%87%BD%E6%95%B0%E4%B8%AD.png)

更细节的原因可以看[multiprocessing官网解释](https://docs.python.org/3/library/multiprocessing.html#multiprocessing-programming)

但是我们更多的是使用multiprocessing中的Pool来创建可以启动大量子进程的进程池，写了一个简单的例子

```
# -*- coding: utf-8 -*-
from __future__ import unicode_literals
import multiprocessing
import time
def AyuTask(name):
	print 'Task %s is Running' % name
	t = time.time()
	time.sleep(2)
	print 'Task %s is sleep over' % name
	return t

if __name__ == '__main__':
	multiprocessing.freeze_support()
	cpus = multiprocessing.cpu_count()
	pool = multiprocessing.Pool(cpus)
	results = list()
	for i in xrange(cpus+1):
		result = pool.apply_async(AyuTask, args=(i,))
		results.append(result)

	pool.close()
	pool.join()

	for result in results:
		print result.get()
```
我们定义了AyuTask()这个方法，获得一个当前时间的时间戳，接着让进程休眠2秒，然后返回这个时间戳，该方法会被子进程执行

然后在if __name__ == '__main__':中，初始化pool，在windows平台下，需要加上multiprocessing.freeze_support()这句，而且要被if __name__ == '__main__':包裹，这样可以避免我们通过py2exe等工具生成windows下的exe文件爆出RuntimeError错误，官方解释如下

![freeze_support使用.png](http://obfs4iize.bkt.clouddn.com/freeze_support%E4%BD%BF%E7%94%A8.png)

[相关地址](https://docs.python.org/2/library/multiprocessing.html)


为了让多进程发挥最大的作用，使用pool.apply_async()方法，也就是异步调用的方式来执行任务，pool.apply()则是同步的方式，同步表示就是每执行完一个任务才能继续执行下一个任务，这样多进程显得没什么意义，可以将上面的代码改成同步的方法执行，将输出语句改为print result，看一下效果

![python多进程同步执行.png](http://obfs4iize.bkt.clouddn.com/python%E5%A4%9A%E8%BF%9B%E7%A8%8B%E5%90%8C%E6%AD%A5%E6%89%A7%E8%A1%8C.png)

每个任务的执行都要等待上一个任务执行完，不是我们想要的

异步执行的结果如下图

![python多进程.png](http://obfs4iize.bkt.clouddn.com/python%E5%A4%9A%E8%BF%9B%E7%A8%8B.png)

可以看出，我们设定了进程池的大小，通过multiprocessing.cpu_count()获得cpu的核数，是几核就将进程池设置成多大，这样可以最大程度的利用多核的cpu，当进程池满了，下一个进程只有等进程池中的某个进程执行完后才可以进入进程池被执行

如果我是4核的电脑，是否可以将进程池大小设置成5呢？当然可以，但是这样效率并不是最高的，因为有一核在执行两个进程，因为cpu是时间轮序执行任务的，所以这就带来的**两个进程间不停切换的成本，降低的效率**

后面我们调用了join()方法，等待所有的子进程执行完毕，但要注意，调用join()方法前必须先调用close()方法，调用close()方法后就不能往进程池中添加新进程了

因为pool.apply_async之后的语句都是阻塞执行的，所有在最后为所有子进程执行完后，才调用result.get()方法获得其中的值，一般**获取子进程中返回值最后在进程池被回收好，避免阻塞后面的语句**


不知不觉写了这么多，下一篇再讲python的GIL这个大锁吧

用代码编写乐趣---ayuliao