---
title: wordpress推荐安装的插件
date: 2017-06-06 10:16:44
tags: wordpress
---

## 简介
为了搭建出一个比较OK的英文网站，经过我几天的摸索和实际，觉得应该总结一下这几天看到的东西，记录一下觉得很OK的插件

## 什么是插件
如果你也是用wordpress搭建一个网站，却没有使用功能强大的插件，那真的太可惜了，利用好一些优秀的插件，可以提高你网站的搜索排名和优化你网站的内部结构（在一些网站上也将插件称为外挂）。

wordpress的插件都会放到下面的目录
```
\wp-content\plugins
```

一般有两种方法来安装插件，一种就是通过在线安装，这种方法的好处就是简单，但是如果你的服务器在国内，在线安装会显得非常的慢，而且在线安装界面会比较卡，这让人很不爽

![Addplugins](http://obfs4iize.bkt.clouddn.com/Addplugins.png)

另外一种方法就是在网上下载好，然后将同名的文件夹解压放到该目录下，我常用这种方法下载一下破解的付费插件，体验一下付费插件是否有那么牛逼，但是这种方法有个坏处就是，你不是从官方下载的，你下载的插件很可能有恶意代码，所以还是不要这样做比较好，因为恶意代码对网站的伤害太大

下面讲讲我常用的插件

## Akismet Anti-Spam
Akismet鼎鼎大名的反垃圾邮件的插件，刚安装完wordpress后就会有，是默认安装的插件之一，我的网站刚开没多久，没什么人访问，所以也就没有垃圾邮件来骚扰我，所以我并没有感受它的强大，但是盛名之下，应该还是不错的，从2015年起，Akismet就开始收费了，是不是很伤心，但是其实并不一定要交钱，看我操作。

首先安装并激活插件，激活成功后，它会提醒你需要一个API Key，你需要去相应的网站上去申请它，直接通过提示进入相应的界面，最后你会看到一个交钱的界面

![akismet](http://obfs4iize.bkt.clouddn.com/akismet.png)

其实它这个只是建议你捐点钱，你可以将$36拖到最左边，不给钱直接，直接申请获得一个API Key

![akismet2](http://obfs4iize.bkt.clouddn.com/akismet2.png)

这样就搞定了

## All in One SEO
这个插件就真牛逼了，SEO虽然不是特别难，但是也要一定的学习成本，我对SEO圈内印象最深的话就是，内容为王，一篇原创内容会更让搜索引擎青睐你的网站，所有想真正做一个OK的网站，还是不能单纯的复制粘贴的，虽说内容为王，但是网站的布局混乱还是很不好的，像以前的站长那是必会一些SEO，现在搭建wordpress就不必那么麻烦了，直接使用All in One SEO，将所有SEO有关的内容都帮你集成好了，该插件有付费版和免费版，我现在穷，所以用的是免费版。

![SEO优化](http://obfs4iize.bkt.clouddn.com/SEO%E4%BC%98%E5%8C%96.png)

All in One SEO功能强大，而且完善的文档，但是如果你对SEO一点概念都没有，看的还是像天书，我看的时候对于一些概念也不明白，但没关系，一般文档里都有详细的介绍。

你可以进入Feature Manager将All in One SEO的一些功能激活，如XML网站地图和Robots.txt文件编写功能等，都是比较重要的功能。

All in One SEO中需要设置的东西比较多，有点耐心将它设置好，让我们的网站对搜索引擎更加友好。同时All in One SEO可以帮助我们将[Google Search Console](https://www.google.com/webmasters/)和[Google Analytics](https://analytics.google.com)整合进来

Google Search Console，通过名字就可以知道是Google搜索控制中心，通过这个控制中心你可以查看Google为你提供的详细数据，如搜索结果、有哪些搜索流量等等，这样你就可以对你网站的流量有个大致的了解，好针对性的写文章，同时你可以主动提及网站地图，让你的网址对搜索引擎更加友好，可惜我这个是新站，所以没什么流量。

![GoogleSearchConsole](http://obfs4iize.bkt.clouddn.com/GoogleSearchConsole.png)

Google Analytics，同样通过名字就可以知道是Google分析工具，它主要用于分析你网站的流量，如用户数、跳出率之类的，这些数据也要好好利用起来，这样才能针对性的提高自己网站

![GoogleAnalytics](http://obfs4iize.bkt.clouddn.com/GoogleAnalytics.png)

为何不用统计性的插件？
其实也不是不可以使用统计型的插件，使用统计插件更加方便，直接安装启用一下，就能很直观的看到网站的流量数据了，而且这些统计插件都比较酷，如有个世界地图告诉你你的用户来自哪里等等，但是这些都功能都比较耗费资源，也就是会让你网站的相应速度变慢，这是我不能忍的，而且在wordpress相关的论坛潜水了一段时间，发现使用统计插件好像不怎么被推崇，耗资源一说，而且统计插件统计出来的数据比Google统计出来的高几倍，同时一些出名的统计插件还被爆出有漏洞，所以我是不推荐使用的，如果你想体验一下，这里推荐Slimstat Analytics这个统计插件，这个统计插件是后起之秀，感觉新一点的插件可能没漏洞（当然只是单纯的感觉），当然好资源还是肯定的，因为它必须要跟踪浏览你网站的每个用户，才能记录出数据。

完成了All in One SEO的总体设置后，你还可通过Feature Manager安装一些必要的功能，这里推荐安装XML Sitemaps，也就是网站地图，它会告诉搜索引擎这个网站的整体构造方便搜索引擎去爬数据，同时也安装一下Robots.txt，这个文档是一个规范，用于建议搜索引是否要搜索该网站中的每个界面，但是这也仅仅是建议，搜不搜还是看搜索引擎本身，比如微信当然不希望百度的爬虫搜索自己，但是百度的爬虫敢不搜吗？它当然会搜，这样体系出一个道理，如果你的网站已经被很多人认可，访问量很大时，搜索引擎会主动来收录你

![SEO优化2](http://obfs4iize.bkt.clouddn.com/SEO%E4%BC%98%E5%8C%962.png)

## Contact Form 7
直接看Contact Form 7这个插件的简介就可以知道该插件的作用，主要用于管理多个联系表单，可以通过简单的标记灵活地自定义表单和邮件内容。大部分wp站长用通过该插件实现收集访客反馈的信息，因为有些恶意用户会发垃圾话，所以Contact Form 7表单Akismet垃圾邮件过滤，而且是通过AJAX加载的，所以并不会严重影响网页的加载速度。

## Kindeditor For Wordpress
kindeditor是一款轻量级的在线编辑器，替代Wordpress原生的富文本编辑器，比较受推崇，我已经见我关注的一些人都推荐使用这个编辑器，so我觉得下水试一下，感觉还ok，也没什么惊艳的感觉，因为我更喜欢使用Markdown来编写博客，其实Wordpress中也有几个不错的Markdown插件，不过我还没体验过

![在线编译器.png](http://obfs4iize.bkt.clouddn.com/%E5%9C%A8%E7%BA%BF%E7%BC%96%E8%AF%91%E5%99%A8.png)

## Relevanssi
是一个功能比较牛逼的搜索插件，可以将wordpress默认的搜索插件，Relevanssi同样有免费版和付费版，你可以在后台设计搜索模板，还可以自定义颜色来显示搜索的关键词，该插件还会保留访客所有的搜索结果都wordpress的日志中，这就真牛逼了，一般访客进行搜索的话都是他们迫切想要的内容，这样我们就可以获得访客的需求，然后在写出具有针对性的文章，效果应该很棒，其实我是想将Google站内搜索引入到我的网站中的，毕竟我是Google党

如果你的网站意见有比较高的流量了，可以入手付费高配版

大部分配置都可以通过修改relevanssi.php这个文件实现

![relevanssi](http://obfs4iize.bkt.clouddn.com/relevanssi.png)

## UpdraftPlus - Backup/Restore
一个牛逼的备份还原插件，对一个网站来说，数据是最重要的，如果哪天你的网站被人攻破了（虽然很少有人闲的蛋疼来搞你），只要你有数据，你完成可以Copy出一摸一样的网站。

有备无患，有备无患，所以立刻将UpdraftPlus装上，平时可以不激活使用，但是还是建议一直激活，因为它可以设置定时任务，来完成定时保存。

UpdraftPlus可以将整个wordpress网站中所有的数据，包括完整的数据库、log、插件主题和wordpress核心文件等都可以备份，然后再通过还原功能一键还原，当然UpdraftPlus有免费版和付费版，其实免费版就够用了。

使用UpdraftPlus进行备份的操作非常傻瓜式

![备份所有.png](http://obfs4iize.bkt.clouddn.com/%E5%A4%87%E4%BB%BD%E6%89%80%E6%9C%89.png)

这里我选择备份在云端，这里我选择Dropbox作为存放备份文件的云端，UpdraftPlus插件可以直接支持Dropbox，配置方式也很简单，几乎不用配置什么，你只需注册一个Dropbox则可

![远程备份.png](http://obfs4iize.bkt.clouddn.com/%E8%BF%9C%E7%A8%8B%E5%A4%87%E4%BB%BD.png)

UpdraftPlus会提示你是否备份成功，如果成功了，你直接可以在Dropbox看到相应的备份文件，你可将它们下载到本地保留一份

![wordpress备份.png](http://obfs4iize.bkt.clouddn.com/wordpress%E5%A4%87%E4%BB%BD.png)

## WP Authenticity Checker (WAC)
WP Authenticity Checker是一个检测主题或插件代码是否安全的插件，但是这种类型的安全插件并不会十分智能，它会认为通过base64方法加密的内容是不安全的，当然对于一些比较明显的漏洞还是有一点识别能力的，我们可以安装这个插件，然后开启来检查一下，看看插件有没有严重问题，如果主题、插件都是wordpress在线安装的，那么应该是没问题的，该插件检查完后就可以停用了，节省一些资源

![wac安全.png](http://obfs4iize.bkt.clouddn.com/wac%E5%AE%89%E5%85%A8.png)

## WP Cerber
Cerber也是一个很有必要的安全插件，它的主要功能就是让你的网站避免暴力攻击，如python爬虫跑字典模拟登陆，一些黑客的攻击，因为wordpress登陆的路径都是默认的   你的域名xxx/wp-login.php，如果不做点措施，如现在登陆出错次数或添加验证码区分人机，你的网站很容易就被搞死，就算他一直登陆不了，但是爬虫一直在尝试登陆也会消耗你大量服务器资源，但只要装上Cerber，搞定上面的问题，Cerber可以建立黑名单来现在IP登陆，可以记录登录成功的人的IP、登录时间、登录用户名等等，可以通过reCAPTCHA（Google验证码）来区分人机

总之该插件功能强大，为了提高一点网站的安全，牺牲一些系统资源也是值得的

![记录登录.png](http://obfs4iize.bkt.clouddn.com/%E8%AE%B0%E5%BD%95%E7%99%BB%E5%BD%95.png)

![登录验证.png](http://obfs4iize.bkt.clouddn.com/%E7%99%BB%E5%BD%95%E9%AA%8C%E8%AF%81.png)

还有一个问题，就像前面提到的，登录文件总是  网站域名/wp-login.php，我们可以将login.php改为其他人猜不到的文件名，有两种方法，第一种就是修改源代码，第二种就是通过插件来实现，我使用的是第一种方法

具体的操作如下

1.将login.php修改成其他不易被猜到的文件名，如ayuliao.php

2.打开ayuliao.php，将文件里的wp-login.php全部替换为ayuliao.php，可以通过vim命令来替换，如下

```
# vim ayuliao.php
然后在vim中输入如下一次性替换命令
:1,$s/wp-login.php/ayuliao.php/g
```

没使用vim这个命令的可以Google一下，vim真的很强大，很好用

3.打开wp-includes/general-template.php，同样将wp-login.php全部替换为ayuliao.php，我使用跟上面一样的方法

但是这样修改完后，你会发现非常鸡肋，虽然wp-login.php确实没了，但是只要它输入 网站域名/wp-admin.php，如果你没有登录，wordpress会自动跳转到你的登录界面，那么前面的修改就没有什么效果了，虽然登录界面有Google验证码拦着大部分恶意机器人，但是改的这么辛苦竟然一定卵用都没有，让我非常郁闷

直到后面我发现了这个插件

## BulletProof Security
这个安全插件功能非常牛，而且很全面，可以防范很多安全工具，保护你的 .htaccess 文件，记录HTTP错误，数据库备份，数据库前缀的修改，还可以防各种各样的攻击，在登录安全方面也做的比较全面，通过它，我们可以修改登录文件的名称，也就是wp-login.php名称，可以隐藏wp-admin.php，我虽然开启了它比较多的功能，但是对目前我来说最有用的就是隐藏wp-admin.php，后面我停用了这个插件，主要出于网站性能考虑，虽然停用了它，但是隐藏wp-admin.php的效果还在，这说明该插件做了一些源码上的修改。如果你有兴趣，你也可以安装来玩弄一下

## WP Fastest Cache
Fastest Cache又是一款名气很大的插件，分免费和付费，你应该猜到了，我使用的是免费版，一句话概括它的强大功能就是使用缓存、压缩等技术让你的网站更快，这样当你下次再次打开网站时，就不必重头加载网站了，提高了用户打开网页的速度。

![搞定缓存.png](http://obfs4iize.bkt.clouddn.com/%E6%90%9E%E5%AE%9A%E7%BC%93%E5%AD%98.png)

## WP Super Cache
提高网站响应速度的方法除了生成缓存，还有就是使用CDN，优秀的CDN会让你的网站打开速度有明显的提升，但是大部分的CDN都是需要付费的，如果你要需要使用付费CDN，这里推荐MaxCDN，它可以与WP Fastest Cache结合使用，你可以直接安装MaxCDN这个插件，它什么都好，就是需要付费

![选择一个CDN.png](http://obfs4iize.bkt.clouddn.com/%E9%80%89%E6%8B%A9%E4%B8%80%E4%B8%AACDN.png)

这里推荐使用WP Super Cache，该插件的主要功能依旧是缓存网站，我一般与WP Fastest Cache一同使用，让两者功能互补，反正又不会冲突，但是WP Super Cache最牛逼的提供免费的CDN，你可以直接使用

![CDN支持.png](http://obfs4iize.bkt.clouddn.com/CDN%E6%94%AF%E6%8C%81.png)

用代码创造乐趣---ayuliao