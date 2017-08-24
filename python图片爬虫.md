---
title: python图片爬虫
date: 2016-06-18 19:03:45
tags: python
---
## 简介
python图片爬虫，其实在网上有很多相关的代码，但是自己看见比较热的都是用比较老的库了，于是自己写了一个
前提要有点熟悉requests和BeautifulSoup的用法
BeautifulSoup可以去看一下其[开发文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/)或者一些[BeautifulSoup入门博客](http://www.cnblogs.com/yupeng/p/3362031.html)
requests比urllib2简洁很多，可以去[官方网站](http://docs.python-requests.org/en/master/)了解，因为最新的文档是英文的，所有可以去看旧一点的[中文文档](http://requests-docs-cn.readthedocs.io/zh_CN/latest/)，基本教程没有什么差别

## 方法
利用requests来爬取源码，BeautifulSoup来解析出图片地址，然后进行下载
目标网站：http://www.jdlingyu.com/，是一个非常“绅士”的网站

## 网页分析
![首页](http://obfs4iize.bkt.clouddn.com/python%E5%9B%BE%E7%89%87%E7%88%AC%E8%99%AB_1.png)
为了得到高清的图，首页的缩小图，我们当然没有兴趣，但是通过点击它，可以进入二层页面，观看高清图片，这里面的图片才是我们想要的。

由此我们的思路就是，先得到第一层界面进入第二层界面的入口url（也就是那些小图），然后再得到第二层的高清图片的url，在进行下载就好了。

通过chrome的开发者工具分析（按F12），我们知道，入口url的规律
被class为pin-coat的div给包裹着的a标签的href属性里的值就是入口url
![入口url](http://obfs4iize.bkt.clouddn.com/python%E5%9B%BE%E7%89%87%E7%88%AC%E8%99%AB_%E5%85%A5%E5%8F%A3.png)

点击缩略图，进入第二层，一大波高清绅士图铺面而来，而我内心平静如水，帅的人都这样子

通过chrome开发工具分析，可以发现，我们的目标高清图为class为main-body的div中的a标签的href属性的值
得到这个url，程序就基本完成了
![高清图片url](http://obfs4iize.bkt.clouddn.com/python%E5%9B%BE%E7%89%87%E7%88%AC%E8%99%AB_2.png)

## 代码
```python
#coding:utf-8
import re
import requests
import bs4
if __name__ == "__main__":
    root_url = "http://www.jdlingyu.com/page/"
    #为下载的照片命名，看有多少张绅士图
    k = 0
    for i in range(1,10+1):
        url = root_url + str(i) + "/"
        r = requests.get(url)
        #为了避免乱码
        html_cont_1 = r.content.decode('utf-8')
        soup_div_in = bs4.BeautifulSoup(html_cont_1,'html.parser',from_encoding='utf-8')
        div_in = soup_div_in.find_all('div',class_='pin-coat')
        #BeautifulSoup返回的对象的类型不是String，而是数组，为了让BeautifulSoup进行二次索引，将其转换为String类型
        div_in_string = ''
        for x in div_in:
            div_in_string = div_in_string + str(x)
        #div_in_string里的值要为string类型    
        soup_link_in = bs4.BeautifulSoup(div_in_string,'html.parser',from_encoding='utf-8')
        link_in = soup_link_in.find_all('a',href=re.compile(r"http://www.jdlingyu.com/\d"))
        for link in link_in:
            url_in = link['href']
            r_in = requests.get(url_in)
            html_cont_2 = r_in.content.decode('utf-8')
            soup_div = bs4.BeautifulSoup(html_cont_2,'html.parser',from_encoding='utf-8')
            div = soup_div.find_all('div',class_='main-body')
            div_string = ''
            for x in div:
                div_string = div_string + str(x)
            soup_link = bs4.BeautifulSoup(div_string,'html.parser',from_encoding='utf-8')
            link_img = soup_link.find_all('a')
            for link in link_img:
                url_img = link['href']
                print(url_img)
                img_get = requests.get(url_img)
                #打开文件进行保存
                with open('img/'+str(k)+'.jpg','wb') as f:
                    f.write(img_get.content)
                k = k+1
```
运行这个代码要在代码的相同目录下创建存放图片的文件夹，不会会报没有该文件的错误
![创建文件](http://obfs4iize.bkt.clouddn.com/python%E5%9B%BE%E7%89%87%E7%88%AC%E8%99%AB%E6%96%B0%E5%BB%BA%E6%96%87%E4%BB%B6%E5%A4%B9.png)

制作爬虫，重要的是网页页面代码的分析，当然，要制作一个比较复杂的爬虫，可能还有分模块

对应BeautifulSoup返回类型方法参考[stackoverflow的一个问答](http://stackoverflow.com/questions/20968562/how-to-convert-a-bs4-element-resultset-to-strings-python)

爬虫的一些教程慕课网爬虫实战用了urllb2和BeautifulSoup，极客学院爬虫实战用了request和正则表达式，综合一下，就更加简洁完美了！！

运行完程序后，就会得到满满的成就感！！
![绅士图](http://obfs4iize.bkt.clouddn.com/python%E5%9B%BE%E7%89%87%E7%88%AC%E8%99%AB%E7%BB%93%E6%9E%9C.png)

用代码创造乐趣，我是ayu