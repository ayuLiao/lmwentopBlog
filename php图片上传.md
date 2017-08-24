---
title: php图片上传
date: 2016-08-10 21:56:07
tags: php
---

## 简介
朋友圈有很多人发一些很“有趣的”合成图片，比如各种证件的头像和人名是你自己的，觉得这是一个比较有趣的东西，便打算用刚刚学的php实现一下。

## 开发流程
其实思路很简单，就是前端通过from表单的来上传图片和文字，后端接收到图片和文字，将文字转成图片，然后将前端传来的头像图片和文字图片和后端本身就存在的背景图片合并成一张图片，传回给前端。

看一下最终结果图：
![前端](http://obfs4iize.bkt.clouddn.com/php%E5%9B%BE%E7%89%87%E5%90%88%E6%88%90%E5%89%8D%E7%AB%AF.jpg)
![后端](http://obfs4iize.bkt.clouddn.com/php%E5%9B%BE%E7%89%87%E5%90%88%E6%88%90%E7%BB%93%E6%9E%9C.jpg)

这一篇博文聊聊php文件上传，对于这个小项目的图片处理留在下一篇博客聊！！

## php文件上传
图片也是文件的一种，在前端我们用from这种简单的方法来选择本地的图片，然后进行提交，跳转到相应的php中。

通过**input type='file'**标记选择本地文件，如果支持文件上传操作，我们必须要将<from>标签中的enctype和method两个属性指明相应的值。

enctype = 'multipart/form-data'	用来指明表单的编码数据方式，让服务器知道我们要上传一个文件了，并带有常规的表单信息。

mothod = 'POST' 用post方法来传输要上传的数据，上传文件是不能用get方法的

最好在form表单中设置一个hidden类型的input框，其中name为MAX_FILE_SIZE，通过它的value来限制上传文件大小，将input框隐藏，但是其功能和没有隐藏的input是一样的，可以理解为，我不想显示这个东西，但是传值时我又需要，input的name和value都是元素的属性，name可以理解为元素的名字，value可以理解为元素的值，表单提交后input会以name的值=value的值的形式传给后台（也就是键值对的形式），这样可以通过name找到value。

这里我用了Amaze UI前端框架来构造一下页面，具体代码如下：

```html
<!doctype html>
<html class="no-js">
<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="description" content="">
  <meta name="keywords" content="">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Play Picture</title>
  <meta name="renderer" content="webkit">
  <meta http-equiv="Cache-Control" content="no-siteapp"/>
  <meta name="mobile-web-app-capable" content="yes"> 
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black">
  <meta name="apple-mobile-web-app-title" content="Amaze UI"/>
  <link rel="stylesheet" href="http://cdn.amazeui.org/amazeui/2.7.1/css/amazeui.min.css">
</head>
<body>   
    <form action="playimg.php" method="post" enctype="multipart/form-data">
        <input type="hidden" name="MAX_FILE_SIZE" value="1000000">
        <input type="text" class="am-form-field am-radius" placeholder="输入姓名" name="id" />
        <div class="am-form-group am-form-file">
          <button type="button" class="am-btn am-btn-success am-btn-sm">
            <i class="am-icon-cloud-upload"></i> 选择图片</button>
          <input id="img" type="file" name="img" multiple> 
           <p id="choose_img">请选择要上传的文件...</p>
        </div>    
    <button type="sumbit" class="am-btn am-btn-success btn-loading-example" data-am-loading="{spinner: 'circle-o-notch', loadingText: '提交中...', resetText: '提交成功'}">提交</button>
    </form> 
<script src="http://libs.baidu.com/jquery/1.11.3/jquery.min.js"></script>
<script src="http://cdn.amazeui.org/amazeui/2.7.1/js/amazeui.min.js"></script>
 <script>
          $(function() {
            $('#img').on('change', function() {
              var fileNames = '';
              $.each(this.files, function() {
                  fileNames += '<span>选中图片为：' + this.name + '</span> ';
              });
              $('#choose_img').html(fileNames);
            
            });
          });
        </script>
</body>
</html>
```

上面就是前端代码，我们选择一个文件，然后通过type为sumbit的button来提交这个表单，随后就会通过post方式提交表单的内容到playimg.php中，这种方式上传图片和文字虽然十分方便，但是不会有上传进度条，用户不知道上传进度，用户体验可能不好！这里就用这种简单的方法来实现。

通过观察前端代码可以发现，用户输入的名字的name为id，那么我们在php中就可以通过$_POST['id']来得到用户名，具体的后端代码如下，有详细的注释

这里解释一下里面的一些函数，让没有什么php代码经验的人也可以看得懂。

$_POST数组：是通过HPPT POST方法传递的变量组成的数组，$_POST和$_GET都可以保存表单提交的变量，使用哪一种方法在from表单的method属性来设置，$_POST数组只能访问以POST方法提交的表单，通过name来得到其中的值

$_FILES数组：使用表单的file输入域上传文件事必须用POST方法，但是在服务器中不能通过$_POST这个超全局变量数组来获取表单中的file域的内容，通过$_FILES超全局变量数组是表单通过POST方法传递已经上传的文件项目组成的数组，它是一个二维数组，但是如果from表单同时通过file输入域上传多个文件，那么$_FILES就是一个三维数组了。

$_FILES数组内容如下: 
$_FILES['myFile']['name'] 客户端文件的原名称。 
$_FILES['myFile']['type'] 文件的 MIME 类型，需要浏览器提供该信息的支持，例如"image/gif"。 
$_FILES['myFile']['size'] 已上传文件的大小，单位为字节。 
$_FILES['myFile']['tmp_name'] 文件被上传后在服务端储存的临时文件名，一般是系统默认。可以在php.ini的upload_tmp_dir 指定上传到服务器的文件的临时存储文件夹，这些临时文件会放到该路径下，文件名称是$_FILES['myFile']['tmp_name']。 
$_FILES['myFile']['error'] 和该文件上传相关的错误代码。['error'] 是在 PHP 4.2.0 版本中增加的。下面是它的说明：(它们在PHP3.0以后成了常量) 
UPLOAD_ERR_OK 
值：0; 没有错误发生，文件上传成功。 
UPLOAD_ERR_INI_SIZE 
值：1; 上传的文件超过了 php.ini 中 upload_max_filesize 选项限制的值。 
UPLOAD_ERR_FORM_SIZE 
值：2; 上传文件的大小超过了 HTML 表单中 MAX_FILE_SIZE 选项指定的值。 
UPLOAD_ERR_PARTIAL 
值：3; 文件只有部分被上传。 
UPLOAD_ERR_NO_FILE 
值：4; 没有文件被上传。 
值：5; 上传文件大小为0。

**这里要比较注意的一点是在php.ini的upload_tmp_dir 指定上传到服务器的文件的临时存储文件夹，这个如果不设置的话，上传也不会报错，因为upload_tmp_dir会有默认的值**


explode(separator,string,limit)函数：参数一是指定的分割符号，参数二是要分割的字符串，它会根据参数一来分割参数二，返回一个数组，数组元素是分割好的字符串

array_pop(array)函数：删除数组最后一个元素，并返回这个被删除的元素，其实就是将数组最后一个元素出栈了

in_array(array)函数：用来判断数组中是否有指定元素

is_uploaded_file(file)函数：判断指定的文件是否是通过POST方法上传的，如果是，则返回TRUE，用于防止潜在的攻击者对原本不能通过脚本交互文件进行非法管理，这个函数可以用来确保恶意的用户无法欺骗脚本去访问本不能访问的文件。这里的file必须用$_FILES['img']['tmp_name']，不能用客户端的文件名，不然无法正常执行。

move_uploaded_file(file,newloc)函数：虽然copy()和move()都可以达到移动文件的效果，但是该函数还提供了一个检查并确保第一个指定的文件是否是合法上传文件（既是否通过PHP的POST上传机制所上传的），如果文件合法，才会将文件移动到第二个参数指定的文件路径中去，如果不是合法的上传文件，就不会有任何操作，成功返回TRUE。

```php
<?php

	$name = $_POST['id'];//得到传来的文字

	$allowtype = array('png','jpg','jpeg');//允许上传的文件
	$size = 1000000;//允许上传文件大小
	$path = "D:/phpImg";

	if($_FILES['img']['error']>0){
		echo '上传错误:';
	
		switch($_FILES['img']['error']){
			case 1: die('上传文件超出PHP配置文件中的大小：upload_max_filesize');
			case 2: die('上传文件超出表单中MAX_FILE_SIZE大小');
			case 3: die('文件只被部分上传');
			case 4: die('没有上传任何文件');
			case 5: die('上传文件大小为0');
			default: die('未知错误');
		}
	} 

	//判断上传文件是否为允许的类型
	$hz =array_pop(explode(".", $_FILES['img']['name']));
	if(!in_array($hz,$allowtype)){
		die('不允许上传'.$hz.'类型文件');
	}

	//判断上传文件是否为允许文件大小
	if($_FILES['img']['size']>$size){
		die('超过了{$size}字节大小，不允许上传');
	}

	date_default_timezone_set("UTC");//设置一个时间域，不然使用date方法会有warning
	$filename = date(Yhis).rand(100,999).".".$hz;

	//判断是否为上传的文件,必须使用$_FILES['img']['tmp_name']才可以判断，如果使用从客户端上的名字，则不能正常运行
	if(is_uploaded_file($_FILES['img']['tmp_name'])){
		if(!move_uploaded_file($_FILES['img']['tmp_name'],$path.'/'.$filename)){
			die('不能将文件临时时目录移动到指定目录');
		}
	}else{
		die('上传文件{$_FILES["img"]["name"]}不是合法路径');
	}
	// echo "文件{$upfile}上传成功，保存在目录{$path}下，大小为{$filename}字节";
```
上面就是后端代码，得到文件并且进行一系列PHP内置函数的检查。

**用代码创造乐趣---ayu**