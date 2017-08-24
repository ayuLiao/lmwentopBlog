---
title: github项目创建、上传、删除、常见错误
date: 2016-07-14 08:46:50
tags: github
---
总结一下github常见的操作，记录一下，方便自己

## 创建项目
直接点击new repository
![新建项目](http://obfs4iize.bkt.clouddn.com/gitbuh%E6%96%B0%E5%BB%BA%E4%BB%93%E5%BA%93.png)

输入一个之前自己创建过的项目中没有哦使用过的名字，最好使用英文名

## 上传项目

	git init

这个命令会在本地文件夹内创建一个.git目录


	git add .

将本地目录中的所有文件都添加进上传列表中来

![git初始化](http://obfs4iize.bkt.clouddn.com/%E4%B8%8A%E4%BC%A0%E9%80%9A%E7%9F%A5%E5%BF%AB%E6%89%8B_1.png)

	git commit -m "要提示的内容"
输入要提示自己的内容
![注释](http://obfs4iize.bkt.clouddn.com/%E4%B8%8A%E4%BC%A0%E9%80%9A%E7%9F%A5%E5%BF%AB%E6%89%8B_2.png)

	git remote add origin https://github.com/ayuLio/你的仓库名.git
连接你之前在github上面创建的仓库

	git push -u origin master
将本地项目推送到github创建的项目的master分支上

![上传](http://obfs4iize.bkt.clouddn.com/%E5%B0%86%E6%9C%AC%E5%9C%B0%E9%A1%B9%E7%9B%AE%E6%8E%A8%E9%80%81%E5%88%B0master%E4%B8%8A.png)

但是我们发生了错误
Faild to connect to gitubu.com port 443:Connection refused
拒绝连接

具体原因我也不清楚，上网一查，可能是地址写错了会出现这样的报错，但是我地址根本没有写错

我这个NoticeQuick仓库是之前就创建好的，本地代码也是之前就初始化和上传过的，所以出现了这样的错误

我尝试着用了另一个链接地址https://github.com/ayuLiao/NoticeQuick

报出另外一个常见错误
remote origin already exists
表示远程源已经存在，也就是说已经用命令
	git remote add origin 你的url
是不能再使用的

此时要使用
	git remote rm origin
![常见错误](http://obfs4iize.bkt.clouddn.com/%E5%87%BA%E7%8E%B0remote_origin_already_exists%E9%94%99%E8%AF%AF.png)
再次git remote add origin就不会出现保持了

此时再git push就可以了

但是上传到最后
![报错](http://obfs4iize.bkt.clouddn.com/%E4%B8%8A%E4%BC%A0%E9%A1%B9%E7%9B%AE%E8%BF%87%E5%A4%A7%E6%8A%A5%E9%94%99.png)
报错
error:failed to push some refs to "你的仓库的URL"

看了一下报错原因，我的通知快手的word文档大小大于github的100M限制
所有报错

在本地文件内，我将**通知快手.doc**删除

在此进行git push
依旧报错，同样的错误，同样说通知快手.doc文件过大

因为之前git add .已经将通知快手.doc加入上传列表了，所有删除本地依旧报错

我开了一新文件夹，将要上传且大小小于100M的文件放入文件中，重新git init
初始化git

按照上面的步骤，一次成功
![成功](http://obfs4iize.bkt.clouddn.com/%E4%B8%8A%E4%BC%A0%E9%A1%B9%E7%9B%AE%E6%88%90%E5%8A%9F.png)

## 删除项目
进入到要删除的项目，点击setting
![进入项目的setting](http://obfs4iize.bkt.clouddn.com/%E5%88%A0%E9%99%A4github%E9%A1%B9%E7%9B%AE.png)

![setting内](http://obfs4iize.bkt.clouddn.com/%E5%88%A0%E9%99%A4github%E9%A1%B9%E7%9B%AE2_2.png)


直接点击删除，删除时要输入项目的名称，只有输入正确才可以删除

![删除项目](http://obfs4iize.bkt.clouddn.com/%E5%88%A0%E9%99%A4github%E9%A1%B9%E7%9B%AE_2.png)

![输入删除项目名](http://obfs4iize.bkt.clouddn.com/%E5%88%A0%E9%99%A4github%E9%A1%B9%E7%9B%AE_3.png)
