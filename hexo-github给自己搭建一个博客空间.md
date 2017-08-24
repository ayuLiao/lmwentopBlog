---
title: hexo+github给自己搭建一个博客空间
date: 2016-06-09 17:02:24
tags: hexo
---
## 为何自己要写博客？
我在自己打算要写博客前问过自己这样的问题，并不是自己十分热爱写作，也并不是自己有特别想要表达的感悟，只是觉得，我想说一些话，自己可以记录下来，学过的东西，自己可以总结出来，我觉得很多人可能都觉得自己是一个情感丰富的人，是一个有很多智慧和感慨的人，我曾经也常常这样认为，觉得自己看见很多现象，有很多话想说，想表达出自己对此的看法，让他人觉得自己很有思想之类的，我也以此作为我的写博客的动机，我会特有去看书，特意去想一些有的没的感慨来书写，似乎自己看透了很多东西，但是回头去看之前写过的文字，觉得有种十分逞强的感觉！而自己并不喜欢这种感觉，所以觉得重新开一个博客，重新写点东西，总结点技术。

我有一点感触，感触自己其实没有什么料，因为之前的博客我要求自己日更，但是一个星期下了，就不知道要写什么了，我惶恐，所以我会特意去想些感慨，很多事情其实没有必要写出一篇博客，寥寥几字便可以说明白，却硬要写出点味道来，让自己觉得自己有点高深，我没有关注除技术之外的博客，所以不明白如何写出一点新的生活感悟，到后来才有点理解，生活哪有这么多感悟，对一些事哪有那么深的感触可以动笔写文。

但是我依旧选择重新开始，不在是为了一个目标，一个任务，而是为了有一个可以与自己对话的思考过程，我之所以觉得自己很虚，就是因为我之前在写些东西的时候，与自己对话，与自己进行交流，发现其实自己是很空的，并没有此前那种自己很有思想的感觉，感觉你的思想不过就拿一两句大白话，我也试过，将一个大白话用一个故事来引出，但是这实在让我自己都有点不开心，因为我没有经历过什么故事，写的东西是空洞的。

只有开始写点东西，浮躁的心才会开始安静，你会明白，此前的好高骛远是没有根基支持的，你感觉你懂了很多，当要写出来，要进行一次与自己的对话时，却连自己都说服不了，可是此时的我们却总是想说服他人，所有他人说你浮躁了。

同时，写点东西也还带有之前的一个目标，锤炼自己的思想，我至今依旧相信，觉得一个人人生高度的是一个人的思想高度，而不是一个人财富，或一个的技术。这样有思想高度的人，是会被人尊敬的。

## 怎么搭建博客？
### 1.安装前准备
a.安装Node.js(可以去[中文官网](http://nodejs.cn/)下载按照)
Node.js:基于 Chrome V8 引擎的 JavaScript 运行环境，是JavaScript运行在服务端的一项技术，Node.js是单线程的。Node.js 的包管理器 npm，是全球最大的开源库生态系统。

b.安装git或github客户端(用于跟github那边进行数据的传输)
### 2.利用npm安装hexo
Hexo 是一个基于nodejs 的静态博客网站生成器，作者是来自台湾的 Tommy Chen

特点：
> *不可思议的快速 ─ 只要一眨眼静态文件即生成完成
> *支持 Markdown
> *仅需一道指令即可部署到 GitHub Pages 和 Heroku
> *已移植 Octopress 插件
> *高扩展性、自订性
> *兼容于 Windows, Mac & Linux
进一步入门了解可以看这篇博客
[hexo —— 简单、快速、强大的Node.js静态博客框架](https://segmentfault.com/a/1190000000370778)

c.安装：
```JavaScript
	C:\Users\ayu> npm install -g hexo
```
会在将hexo安装到全局模式下，也就是说，任何目录下的cmd（window命令控制）都可以使用hexo

进入一个文件夹（这个文件夹是你想让它成为你博客的根目录）
在该目录下打开git命令行
![git命令框](http://obfs4iize.bkt.clouddn.com/git%E5%91%BD%E4%BB%A4%E5%88%9B%E5%BB%BA%E5%8D%9A%E5%AE%A2%E6%A0%B9%E7%9B%AE%E5%BD%95.png)

d.初步设置
初始化hexo，hexo就会在你刚刚所选的目录下创建一系列相关文件
```JavaScript
$hexo init
```

生成静态界面：
```JavaScript
$hexo g
```

生成好界面后，启动hexo的服务：

启动hexo服务：
```JavaScript
$hexo s
```

如果你们没有其他软件会占用4000端口的话，就而已成功打开
在浏览器上输入：http:/localhost:4000/
此时，浏览器会显示hexo的初始界面，因为我的已经配置了相应主题，所以会有所不同
![ayu](http://obfs4iize.bkt.clouddn.com/localhost%E5%8D%9A%E5%AE%A2%E7%95%8C%E9%9D%A2.png)

e.配置到github
为了让其他人也可以看见的开启的博客，要将它配置到github上

aa.创建一github账号，注意自己github账号的用户名

bb.在github上建立一个**用户名.github.io**的仓库
![用户名.github.io仓库](http://obfs4iize.bkt.clouddn.com/%E5%88%9B%E5%BB%BAgithub%E5%8D%9A%E5%AE%A2%E9%A1%B5%E9%9D%A2%E7%A9%BA%E9%97%B4.png)
我github的用户名为ayuLiao，所以创建的域名为ayuLiao.github.io，因为我之前已经创建过了，所以这里报错了

cc.配置SSH，让本地主机可以通过ssh来上传项目到github上

![设置](http://obfs4iize.bkt.clouddn.com/%E8%AE%BE%E7%BD%AEssh.png)
进入设置界面
![选择SSH](http://obfs4iize.bkt.clouddn.com/%E8%AE%BE%E7%BD%AEssh_2.png)
选择SSH设置，我这里已经上传过一个SSH了

dd.用git命令行在本地生成一个新的SSH
```JavaScript
$ssh-keygen -t rsa -C "邮箱地址@youremail.com" 
```
然后会出现下面内容
```JavaScript
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/your_user_directory/.ssh/id_rsa):<回车就好>
```
然后会让你输入密码，直接空格，不必输入，这里的密码是用git工具对github进行项目传输时是否要输入的密码
出现这个，就表示ssh创建成功了
![SSH创建成功](http://pic.yupoo.com/vankos_v/DKi6S7PO/lpjsl.png)
我的在C:\Users\啊毓\.ssh中，这个路径在生成ssh key是命令行是有提示的，我们默认哪里空格，所有事默认路径
id_rsa.pub文件是刚刚生成的密钥
![密钥](http://obfs4iize.bkt.clouddn.com/%E6%9C%AC%E5%9C%B0ssh%E6%96%87%E4%BB%B6.png)

复制其中的内容到github的ssh配置处
![ssh复制](http://obfs4iize.bkt.clouddn.com/%E5%9C%A8github%E5%85%81%E8%AE%B8%E5%88%9B%E5%BB%BA%E7%9A%84ssh.png)

ee.测试一下ssh
在git命令行下进行测试
```JavaScript
$ssh -T github用户名@github.com
```
![ssh测试](http://obfs4iize.bkt.clouddn.com/%E6%B5%8B%E8%AF%95%E4%B8%80%E4%B8%8Bssh.png)

f.修改hexo主目录下的配置文件_config.yml
找到配置文件中的deploy，进行如下修改
```JavaScript
deploy:
  type: git
  repository: https://github.com/用户名/用户名.github.io.git
  branch: master
```
![_config.yml](http://obfs4iize.bkt.clouddn.com/hexo%E4%B8%BB%E9%A2%98%E9%85%8D%E7%BD%AE.png)
g.修改hexo的默认主题
可以百度hexo主题，会有很多hexo主题网站,推荐可以去这个[知乎问答](http://www.zhihu.com/question/24422335)上看看，这个知乎问答给具有很高人气的hexo排了先后

首先获得自己喜欢的hexo主题，大多数人气主题都有对应的github，而且大部分也有对应的中文文档
这里获取两个比较常见的主题
```JavaScript
$ git clone https://github.com/cnfeat/cnfeat.git themes/jacman
```
然后修改hexo主目录下的配置文件_config.yml
```JavaScript
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: jacman
```

h.上传和更新文章
配置好后，先生成静态页面，不生成，就无法显示之前配置的效果
```JavaScript
$hexo g
```
然后直接配置到github上，因为之前设置了ssh，所有可以直接上传，因为ssh是加密的数据流，具有一定的安全性
```JavaScript
$hexo d
```

此时可以打开
用户名.github.io来欣赏自己的搭建博客了

如果自己要写新文章，而不是让博客里只有一个简单的hello world
```JavaScript
$hexo n 'newPage name'
```
这样就可以在你的hexo主文件夹\source\_posts(我新生成的文章文件在E:\blog\source\_posts中)
然后利用markdown语法，对md文件进行博客内容的编写，markdown可以是一种提高写作效率语法，目的让我们更加专注于内容，可以看看这篇博客[Markdown语法说明](http://www.markdown.cn/)

i.作为一个有追求的装逼人，当然要有自己的域名
用自己的域名跟自己创建的github博客进行绑定

方法一.可以在用户名.github.io这个项目上创建一个CNAME文件，里面写上自己域名，这里我写的是我的域名
![github中的CNAME](http://obfs4iize.bkt.clouddn.com/CNAME_1.png)
![CNAME内容](http://obfs4iize.bkt.clouddn.com/CNAME_2.png)

这种方法在下次hexo d后，github创建的CNMA文件因为本地没有，所有为了同步，就将github上的CNAME给删除了，这样你绑定的域名就失效了

方法二.在本地创建CNMA文件，在source文件下创建CNMA文件（hexo主目录->source文件夹->CNAME)
![本地CNAME](http://obfs4iize.bkt.clouddn.com/CNAME_3.png)
在CNAME中输入跟方法一中同样的内容
这样每次更github同步都会带上CNAME
当然第一次在本地创建CNAME要进行$hexo g

自己的域名哪里来？为何是CNAME解析呢？

我的域名在[西部数据](http://www.west.cn/)那里注册来

CNAME解析相当于将域名解析成你现在的域名

![CNAME解析](http://obfs4iize.bkt.clouddn.com/CNAME_4.png)

这样就大功告成了

当上面的配置配置完啦，可以进行一些主题的优化
可以加入第三方插件，如评论插件：多说，人数访问插件：不蒜子