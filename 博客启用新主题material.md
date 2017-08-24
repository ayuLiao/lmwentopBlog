---
title: 博客启用新主题material
date: 2017-07-17 08:42:10
tags: 随笔
thumbnail: http://obfs4iize.bkt.clouddn.com/material.png
---

如果你以前浏览过我的博客，可能发现，它变美了。

其实我在审美上比较偏向于简单、扁平，之前使用的主题NexT很好，但是过于扁平简单又感觉没有直击心灵的美感，所以我一直想尝试一下其他的主题，结实NexT是看到知乎上高人气回答，NexT主题排第一，而且看了一下其他主题，并没有特别的冲动，所以就使用了被用户一致好评的NexT，但是用久了，审美就疲劳了，感觉人都是这样，喜新厌旧。这是人性，用久了的手机不会再爱惜，穿久了的鞋子不会再心疼，在一起久了的女朋友，我却还是一样的疼爱，哈哈哈。

言归正传，说句良心话，Material真的是超美的，图文结合又不失简单，有时候还特别羡慕有这样设计感的美工，但是我太懒了，换平时我肯定是懒的换的，毕竟最近也不是特别有空，但，我最近重装了电脑，原本的博客的主题配置没有留下来，所以准备重新陪一个，结果Google一搜，发现了Material，看到这个主题，一个字，酷！

![Material主题](http://obfs4iize.bkt.clouddn.com/material%E4%B8%BB%E9%A2%98.png)

## 快速开搞

安装Material非常简单，直接通过git或者通过npm都OK

git
```
git clone https://github.com/viosey/hexo-theme-material.git
```

npm
```
npm install hexo-material
```

该方式会把 Material 主题下载到 hexo 目录下的 node_modules 文件夹中

![themes文件夹下.pngs](http://obfs4iize.bkt.clouddn.com/themes%E6%96%87%E4%BB%B6%E5%A4%B9%E4%B8%8B.png)

找到下载的material文件夹，将它复制到themes文件夹中

然后打开 站点配置文件，找到 theme 字段，并将其值更改为 material

再将 material 文件夹中的 _config.template.yml 复制一份并重命名为 _config.yml


将站点配置文件中的 language 设置成zh-CN (简体中文)

就这样，没毛病

## 配置

所有基本的配置可以去官方上看

[Material配置介绍](https://material.viosey.com/intro/)

如果想给每篇博客都设置一个缩率图，直接在md文件中使用thumbnail标签，如我这篇文章

![使用缩率图.png](http://obfs4iize.bkt.clouddn.com/%E4%BD%BF%E7%94%A8%E7%BC%A9%E7%8E%87%E5%9B%BE.png)

更详细内容可以看Hexo的相关文档
[Front-matter - 官方介绍](https://hexo.io/zh-cn/docs/front-matter.html)

### 每日图片
Material的首页有个每日图片，这里可以改成bing的图片（默认是写死的）
找到
E:\blog\themes\material\layout\_partial\daily_pic.ejs文件，进行如下修改

html
```
<div class="mdl-card__media mdl-color-text--grey-50" style="background-image:url(<%= theme.img.daily_pic %>)">

修改为

<div class="mdl-card__media mdl-color-text--grey-50" style="background-image:url(https://api.i-meto.com/bing)">
```

但是我觉得有时候bing的自然图片跟我博客有种突兀的感觉，我就没有使用了

### 不蒜子
使用不蒜子来统计访问你站点的人数，找到E:\blog\themes\material\layout\_partial\footer.ejs，添加下面的代码，因为我直接申请使用过，所以直接加上这段代码就OK了

可以访问[不蒜子官网](http://busuanzi.ibruce.info/)了解更多


```
<div>
<script async src="https://dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>
<div>
本站总访问量 <span id="busuanzi_value_site_pv"></span> &nbsp&nbsp&nbsp
你是第<span id="busuanzi_value_site_uv"></span>个来到的小伙伴
</div>
</div>
```

### 代码高亮
material主题还有个很酷的功能就是，它支持代码高亮，首先你需要安装Hexo-Prism-Plugin这个Hexo插件

```
npm i -S hexo-prism-plugin
```

然后在站点配置文件(_config.yml)中加上下面的配置

```
prism_plugin:
  mode: 'preprocess'    # realtime/preprocess
  theme: 'default'
  line_number: false    # default false
```

接着再将hexo中默认代码高亮给关闭，默认的代码高亮真的 没什么卵用。。。

```
highlight:
  enable: false
```

运行下面两条命令，重新生成一下
```
hexo c
hexo g
```

Metarial还有很多集成服务，如评论系统、搜索系统，都可以去Metarial的官网去了解，我还没有装那么多东西

[Metarial集成服务](https://material.viosey.com/services/)

用代码编写乐趣---ayuliao