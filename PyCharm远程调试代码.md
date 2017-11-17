# PyCharm远程调试代码

## 简介
很多时候我们使用PyCharm都是进行本地开发，但是当你要调试服务器上的代码时怎么办？一种蠢方法就是将服务器上的代码下载到本地，编写完成后，再上传，看效果，这种方法是可行的，因为我此前就这样，很难受，感觉一天都在下载和上传，虽然麻烦，但是依旧可以解决问题，我就忍了，直到公司开发的项目要集成微信支付，因为微信支付要验证服务器的合法性，所以在配置微信时，要配置线上服务器的地址，而不能是本机地址，那么此时线上报错了，你将代码拉到线下是无法复现这个错误的，所以必须在线上对代码进行调试。

这也是本章就系统的介绍如何通过PyCharm远程调试代码

## PyCharm SFTP服务
要远程调试代码，那么就先要将代码上传到服务器上，那么上传的方式就有很多了，可以自己去网上下载相应的FTP上传软件，在Mac/Linux下也可以直接通过**scp**命令来将文件或文件夹上传都远程服务器，可以自行去[scp官网](http://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/scp.html)了解一下，基本命令如下

```
复制文件到远程服务器
$scp local_file remote_username@remote_ip:remote_folder
复制文件夹到远程服务器
$scp -r local_folder remote_username@remote_ip:remote_folder

从远程复制文件到本地目录
$scp root@10.6.159.147:/opt/soft/demo.tar /opt/soft/
从远程复制到本地
$scp -r root@10.6.159.147:/opt/soft/test /opt/soft/
```

其实PyCharm也提供了将本地代码上传到远程服务器的功能，PyCharm可以使用它的SFTP服务将我们需要上传的文件上传，可以在 Tool--->Deployment 进行配置

![配置远程连接.png](http://onxxjmg4z.bkt.clouddn.com/%E9%85%8D%E7%BD%AE%E8%BF%9C%E7%A8%8B%E8%BF%9E%E6%8E%A5.png)

从图中可以看出，Upload相关的选项是灰色，这是因为你没有配置，我们选择Configuration，对PyCharm提供的SFTP进行简单的配置

![配置SFTP服务.png](http://onxxjmg4z.bkt.clouddn.com/%E9%85%8D%E7%BD%AESFTP%E6%9C%8D%E5%8A%A1.png)

点击左上角的加号，创建一个配置，将SFTP要链接的服务器路径、服务器用户名和密码都填上，最下面的Web server root URL 字段PyCharm会自动帮我们填写，简单的配置后，可以点击 Test SFTP connection... 按钮，判断SFTP是否可以成功链接服务器，如果返回Success，则表示服务器链接成功

接着还要配置一下本地代码上传到服务的路径，配置好这个，代码才会上传到相应的路径

![上传到服务的路径](http://onxxjmg4z.bkt.clouddn.com/%E9%85%8D%E7%BD%AE%E4%B8%8A%E4%BC%A0%E5%88%B0%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%9A%84%E8%B7%AF%E5%BE%84.png)

配置完后，我们就可以进行上传了

![](http://onxxjmg4z.bkt.clouddn.com/%E5%8F%AF%E4%BB%A5%E4%B8%8A%E4%BC%A0.png)

PyCharm上传有个比较大的坑，**就是你上传时要选中要上传的文件，如果要上传整个项目，就要选中整个项目，如果是上传单独的文件，就选中这个单独的文件**

## PyCharm远程Debug
光上传代码是不能进行调试，因为我们在PyCharm中运行项目使用的依旧是本地的Python解释器，依旧是在本地运行，那么此时对代码进行Debug跟线上的代码没有关系，线上的代码也没有启动，如果要远程Debug，要完成两个配置

1.配置PyCharm的Python解释器，使用服务器上的Python解释器
2.配置PyCharm的Debug功能，让PyCharm运行服务器上的代码

这里先做第一步，进入PyCharm的配置界面，选择Prjoect XXX---> Project Interpreter，配置Python解释器，点击⚙，如下图

![Python解释器1](http://onxxjmg4z.bkt.clouddn.com/%E9%85%8D%E7%BD%AEpython%E8%A7%A3%E9%87%8A%E5%99%A81.png)

然后选Add Remote，如果以后要选中本地不同的Python解释器，就选中Add Local

![Python解释器2](http://onxxjmg4z.bkt.clouddn.com/%E9%85%8D%E7%BD%AEPython%E8%A7%A3%E9%87%8A%E5%99%A82.png)

接着选中SSH Credentials，将远程服务器的地址、服务器的用户名和密码填上，然后在服务器中选择Python解释的路径，也就是 Python interpreter path

![Python解释器3](http://onxxjmg4z.bkt.clouddn.com/%E9%85%8D%E7%BD%AEPython%E8%A7%A3%E9%87%8A%E5%99%A83.png)

选中完后，点击OK，PyCharm就好将远程服务器上相应Python解释器的相关文件都同步下来，你需要等待一下，然后就会出现下面界面，观察Project Interpreter，它已经是远程服务器的Python环境了


![Python解释器4](http://onxxjmg4z.bkt.clouddn.com/%E9%85%8D%E7%BD%AEpython%E8%A7%A3%E9%87%8A%E5%99%A84.png)

这样第一步就完成了，接着来配置PyCharm中的Debug，如果不配置，那么Debug依旧会使用本地的Python解释器

打开PyCharm Debug的配置界面（Run-->Debug-->Edit Configuration...），然后配置一下当前项目下的Python interpreter，确保Debug使用远程服务器的Python解释器来运行代码

![配置Debug.png](http://onxxjmg4z.bkt.clouddn.com/%E9%85%8D%E7%BD%AEDebug.png)

这里有点要注意，**Debug配置界面中，Host要配置为0.0.0.0，有时候我们会将它配置成远程服务器的IP，如果配置成远程服务器的IP，会出现IP已经被分配的错误提示，如下**

![无法启动.png](http://onxxjmg4z.bkt.clouddn.com/%E6%97%A0%E6%B3%95%E5%90%AF%E5%8A%A8.png)


所有都配置完后，就可以进行Debug了，此时使用的就是远程的Python解释器，Debug的也是远程的Python代码，注意观察Debug的第一行，可以发送其通过SSH连接远程服务器

![Debug启动使用远程服务器.png](http://onxxjmg4z.bkt.clouddn.com/Debug%E5%90%AF%E5%8A%A8%E4%BD%BF%E7%94%A8%E8%BF%9C%E7%A8%8B%E6%9C%8D%E5%8A%A1%E5%99%A8.png)

到这里我们就可以在远程调试代码了

## 同步远程数据库
那么有些Python项目，特别是web项目，不只是代码要上传到服务器，也要将数据库中的表同步到服务器，这里介绍一下使用 Navicat Premium 来快速完成这个需求

首先选中一个要同步数据的数据库，选择数据传输

![选择数据传输1.png](http://onxxjmg4z.bkt.clouddn.com/%E9%80%89%E6%8B%A9%E6%95%B0%E6%8D%AE%E4%BC%A0%E8%BE%931.png)

进入数据传输界面，我们可以选择本地要传输的具体内容，比如是将所有的数据表都传输过去还是选择部分表，接着就是选择接受本地数据传输的接收端，也就会服务器对于的数据库

![数据传输2.png](http://onxxjmg4z.bkt.clouddn.com/%E6%95%B0%E6%8D%AE%E4%BC%A0%E8%BE%932.png)

然后就可以进行传输了，速度非常快

![数据同步.png](http://onxxjmg4z.bkt.clouddn.com/%E6%95%B0%E6%8D%AE%E5%90%8C%E6%AD%A5.png)

## 结尾
到这里，你应该可以愉快的调试远端代码了

代码创造乐趣---ayuliao

