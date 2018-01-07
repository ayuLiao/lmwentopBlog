# python爆破加密的zip

## 简介
python爆破加密zip程序的逻辑很简单，就是遍历密码字典，使用密码字典中的密码一个个去试着解密zip，其实所谓的爆破都是这个逻辑，比如MD5爆破也类似，将字典中的值加密成MD5，然后与要解密的MD5进行比较，如果MD5值相同，那么就知道这个MD5对应的值了，当然在2004年的时候，山东大学的王小云教授“破解”了MD5算法，此“破解”不是真正意义上的破解，这方面更多的内容可以自行Google

## show code
通过python简单实现一下上面的逻辑，十几行代码搞定

```
#-*- coding:utf-8 -*-
import zipfile
import optparse
from threading import Thread

def extractFile(zFile, password):
    try:
        if isinstance(password, str):
            password = password.encode(encoding='utf-8')
        # extractall()传入密码则可解密
        zFile.extractall(pwd=password)
        print(zFile+'的密码为:'+password)
    except:
        pass

def main():
    # 定义命令格式，得到zip文件逻辑和密码字典路径
    parser = optparse.OptionParser('usage%prog'+"-f <zipfile> -d <dictionary>")
    parser.add_option('-f', dest='zname', type='string', help='需破解的zip文件')
    parser.add_option('-d', dest='dname', type='string', help='密码字典')
    options, args = parser.parse_args()
    if (options.zname == None) or (options.dname == None):
        print(parser.usage)
        exit(0)
    else:
        zname = options.zname
        dname = options.dname
    #  获得zip对象
    zFile = zipfile.ZipFile(zname)
    passFile = open(dname)
    for line in passFile.readlines():
        password = line.strip('\n')
        t = Thread(target=extractFile, args=(zFile, password))
        t.start()

if __name__ == "__main__":
    main()
```

简单解释一下上面的代码，
1.通过optparse定义命令格式，让使用者通过命令就可以获得zip文件的路径和密码字典的路径
2.实例化zipfile对象
3.逐行读取密码字典中的密码
4.开启多线程，将zipfile对象和密码传入
5.extractFile()方法就是具体的解密zip文件的方法
6.通过zipfile对象的extractall()方法就可以解密zip，**extractall()方法要传入的bytes类型的值，在python3下，默认是str类型，所以要转换一下，如果是python2，默认就是bytes，不必继续转换

接着可以创建一个加密的zip来试验一下

```
mkdir ayuzip
cd ayuzip
vim ayu.txt
```

文艺点在ayu.txt文件中写上凉州词

然后通过zip命令加密一下

```
zip -r ayuzip.zip ayuzip -P ayu1234
```

接着创建密码文件

```
vim password.txt
```

随意写上多个密码，记得要包含真正的密码，不然就会爆破不出来

```
123
23455
1234
121233
123434
ayu123
ayu1234
```

接着运行一下刚刚写好的python代码就好

```
python hackzip.py -f ayuzip.zip -d password.txt
```

一首凉州词送给大家

![](http://onxxjmg4z.bkt.clouddn.com/hackzip.png)

## 结尾
代码非常简单，大家应该发现了，代码其实不是重点，密码字典才是重点，你有个高质量的密码字典你的爆破程序才有用，所以在hack某人时，第一步都是社工，收集这个人的信息，然后通过这些信息加上自己的hack经验定制出针对这个人的密码字典，比如这个人的名字+电话多种组合，当然现在的人也有了点安全意识，这种方法也不一定成功，所以猥琐的思路很重要，比如收集到这个人的信息后，发现他喜欢金融股票类的内容，那么可以做个相关网站（自己做当然太麻烦，通过软件将人家的网站全网爬下来），然后发个他注册，或者中间人劫持他的流量让他访问这个网站并注册，最终的目的都是让他注册，得到密码，因为很多人都是一码通的，所以你就可以。。。嘿嘿嘿了




