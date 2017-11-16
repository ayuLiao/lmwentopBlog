# Django实现用户短信注册

## 简介
网站常见的注册方式有两种，一种是邮箱注册，一种是短信注册，其实两者的实现思路没有太大的差异，就是具体的实现方式有点不同，为了防止简单的爬虫，在注册时要加上图片验证码，当然这也只是增加爬取的难度，并不是杜绝，爬虫的事情其实也可以写一篇文章来聊聊。

本篇文章讲解Django后端实现短信注册的逻辑和代码，不关心前端实现，以前后端分离的形式进行开发，前端可以用Django本身的模板语言，或者其他前端框架

## 图片验证码的逻辑
在实现完整的短信注册的功能前，先要理清图片验证码的流程，前面也简单提了，使用图片验证码的目的是为了防止简单爬虫和恶意用户使用脚本暴力注册，降低服务器的压力。

所谓图片验证码其实就是让用户输入图片中的内容，将用户在前端输入的内容返回给后端做验证，那么具体是什么逻辑呢？

首先我们得在后台生成图片、图片的答案和图片的唯一ID，将图片答案和图片的唯一ID存到数据库中，将生成的图片和图片的唯一ID发送给前端，让前端进行显示，那么当用户填写图片验证码时，就可以将用户填写的答案和唯一的ID传回给后端，后端通过唯一的ID在数据库中找答案，如果找出的答案和用户填写的内容一致，图片验证码就通过，如果不一致，就返回错误信息，为了让系统更可靠，还可以加上过期时间，也就是每个图片验证码都有个有效时间，如果过了这个有效时间了，该图片验证码就失效了，不能再用于验证了

## 短信验证码的逻辑
短信验证码的逻辑其实比图片验证码更加简单，找一个发短信的商户，这里我使用云片网提供的短信服务，使用起来比较方便，首先在云片网上注册一个账号，然后填写一下个人信息和公司信息，接着去申请短信模板，模板申请成功后就可以接入短信服务了，使用云片网的短信服务需要注意的就是**申请的短信模板和编写代码时要发送的短信内容必须一致**，不然短信发送就无法成功，差一个标点符号都不会成功

我申请的模板
![yunpian](http://onxxjmg4z.bkt.clouddn.com/yunpian.jpg)

申请好短信模板后，就可以来理一理代码逻辑，首先要验证前端传递过来的值，这些值必须有用户的手机号、图片验证码答案和图片验证码的ID，获得这些值后，还需要进行验证，首先通过正则表达式验证手机号，如果你是去网上找的，要注意这个正则表达式的日期，太旧的正则表达式会让很多新的合法手机号验证失败，自己去正则表达式验证网站多验证几个手机号，看看这个正则表达式是否合理，如果手机号是合法的，就验证图片验证码，判断图片验证码的答案是否正确，图片验证码有无过期

上面的验证都通过后，就可以调用第三方商户给我提供的短信服务了

## 用户短信注册逻辑
前面两个都明白了，这个就非常简单了，简单将就是验证用户的短信验证码是否正确，同时再验证一下手机号，图片验证码，还有就是要判断短信验证码和图片验证码是否过期，如果所有条件都通过，就将用户的数据存到数据表中

## 项目环境
短信注册的项目算是每个中大型web系统中都会有的东东，前后端交互性还是挺强的，也是很重要的一部分，项目的开发环境：Python3.5+Django(1.11.6)+Django REST Framework(3.7.1)

Django REST Framework是Django中用于开发Restful api的利器，也是Django前后端分离开发常用的库，它对Django的方法进行了二次分装，提供更多方法给我开发合适的Restful api，并且有强大的文档功能，可以直接根据代码来生产接口文档给前端看，这样就不必我们手动维护一个文档了，对于Django REST Framework基础的用法，本篇文章就不讲了，各位可以先去[官网](http://www.django-rest-framework.org/)上看看

Django和Django rest framework都可以直接通过pip安装，至于项目中要使用的其他库，自行pip安装

## 搭建项目和创建数据库
如何搭建Django项目就不多说了，这里我们创建users模块，后面短信注册的代码都写到这个模块中，先来看一下整个项目的结构

![短信注册项目结构]()

我们创建了media文件夹，在该文件夹下创建了codeimage文件夹，用于存放验证码图片。创建了static文件夹，存放静态文件，这里存放了simsum这个字体文件，该字体文件用于生成验证码图片中的内容

接着来创建该项目要使用的数据表，这里我们要创建3张表，分别是用于存用户信息的用户表UserProfile，用于存图片验证码数据的表ImageCode，还有就是用于存短信验证码的表VerifyCode

将这三张表的代码写在 users/models.py中

```python
from datetime import datetime

from django.db import models
from django.contrib.auth.models import AbstractUser

class UserProfile(AbstractUser):
    """
    用户模块，继承Django默认的User，添加新字段
    """
    # null=True 数据库可空 blank=True HTML可空
    name = models.CharField(max_length=30, null=True, blank=True, verbose_name="姓名")
    gender = models.CharField(max_length=6, choices=(("male", u"男"), ("female", "女")), default="female",verbose_name="性别")
    birthday = models.DateField(null=True, blank=True, verbose_name='出生年月')
    # 手机号可以为空，本项目将username作为手机号来使用
    mobile = models.CharField(null=True, blank=True, max_length=11, verbose_name="电话")

    class Meta:
        verbose_name = "用户"
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.username


class ImageCode(models.Model):
    '''
    图片验证码，防止短信狂注册
    将随机生成的短信验证码图片和验证码ID返回前端，前端返回用户填写的结果和codeid
    '''
    codeid = models.CharField(max_length=40, verbose_name='验证码ID')
    code = models.CharField(max_length=10, verbose_name='验证码')
    add_time = models.DateTimeField(default=datetime.now, verbose_name="添加时间")

    class Meta:
        verbose_name = "图片验证码"
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.code


class VerifyCode(models.Model):
    """
    短信验证码
    """
    code = models.CharField(max_length=10, verbose_name="验证码")
    mobile = models.CharField(max_length=11, verbose_name="电话")
    add_time = models.DateTimeField(default=datetime.now, verbose_name="添加时间")

    class Meta:
        verbose_name = "短信验证码"
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.code
```
Django建表就不用多说了，代码中都有简单的注释，这里简单提几点，UserProfile用户表继承自Django自带的用户表，也就是AbstractUser，那么我们的UserProfile就会获得AbstarctUser中的字段，如Username，password，email这些字段，还有就是图片验证码表和短信验证码表中的add_time字段用于存时间，主要用于判断验证码是否过期，这里要注意的就是我们使用的datetime.now，而不是datetime.now()

这样本项目中要使用的数据表就设计好了，接着就去settings.py配置数据库连接，这里使用最常见的mysql

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': '数据库名',
        'USER':'数据库用户名',
        'PASSWORD':'数据库密码',
        'HOST':'127.0.0.1',
    }
}
```

这里顺便将项目中的其他配置一起配了

先配置一下Django的语言环境，将英文配置成中文

```
LANGUAGE_CODE = 'zh-hans'

TIME_ZONE = 'Asia/Shanghai'

USE_I18N = True

USE_L10N = True
# 默认是Ture，时间是utc时间，由于我们要用本地时间，所用手动修改为false
USE_TZ = False
```

因为是前后端分离的方式，前端请求我们后端就会存在跨域问题，要解决这个问题也非常简单，首先通过pip安装一下corsheader

```
pip install django-cors-headers
```

接着价格corsheader注册到INSTALLED_APPS中，当然前面创建的users也要注册到其中

```INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'users',
    # 实现跨越请求
    'corsheaders'
]
```

然后就要配置一下允许与后端进行数据交互的IP，只有这个IP发起的方法才允许跨域，这里让所有IP都可以与后端进行数据交互，非常粗暴

```
#跨域增加忽略
CORS_ALLOW_CREDENTIALS = True
CORS_ORIGIN_WHITELIST = (
    '*'
)

CORS_ALLOW_METHODS = (
    'DELETE',
    'GET',
    'OPTIONS',
    'PATCH',
    'POST',
    'PUT',
    'VIEW',
)

CORS_ALLOW_HEADERS = (
    'XMLHttpRequest',
    'X_FILENAME',
    'accept-encoding',
    'authorization',
    'content-type',
    'dnt',
    'origin',
    'user-agent',
    'x-csrftoken',
    'x-requested-with',
    'Pragma',
)
```

最后还需要设置一下拦截器，拦截器的位置要注意，因为拦截器处理数据请求时是有顺序的

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    # 跨域请求
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

这样跨域问题就解决了

然后将静态文件的路径和资源文件路径配置一下，也就是static和media的路径，只有配置了Django才能路由到这两个文件夹中的内容

```
STATIC_URL = '/static/'

STATIC_URL = '/static/'

MEDIA_URL = '/media/'

STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'static'),
)
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

因为我们是使用短信来注册，验证用户的手机号，那么后期肯定是通过手机号来登录，而Django默认只能通过Username字段来登录，所以我们要自定义一下登录规则，这里先配置一下，后面再写具体的登录逻辑

```
# 使用自定义的登录规则
AUTHENTICATION_BACKENDS = (
    'users.views.CustomBackend',
)
```

因为我们的UserProfile继承了Django默认的用户表，所以要在这里配置一下，告诉Django不要使用默认的用户表了，而是使用我们自己创建的用户表

```
# 使用自定义的用户
AUTH_USER_MODEL = 'users.UserProfile'
```

最后将一些常量也放到这里，一个是手机号的正则表达式，一个是云片网的API Key，发送短信时要带上这个API Key

```
#手机号码正则表达式
REGEX_MOBILE = "^1[358]\d{9}$|^147\d{8}$|^176\d{8}$"

#云片网设置
APIKEY = "云片网API Key"
```

这样配置就配完了，整个项目的骨架也就有了

配置完后，就**使用makemigrations和migrate命令将数据库同步一下，在开发项目时，数据表的每个变动都要通过这两个命令，让变动真正在MySQL数据库中生效**

## 图片验证码
搭建完项目骨架后，就先从最简单的功能写起，先写图片验证码，创建一个名为makeimage的py文件，我们使用Python著名的图片处理库PIL来创建图片验证码所需要的验证码图片，先定义一个图片类，该类用于生成验证码图片，代码比较简短，而且有详细的注释，就不在细讲

```python
from PIL import Image, ImageDraw, ImageFont, ImageFilter
import random,time,string

from SMSReg.settings import STATICFILES_DIRS,MEDIA_ROOT

class Picture(object):
    '''
    图片类，用于图片验证码
    '''
    def __init__(self, text_str, size, background):
        '''
        text_str: 验证码显示的字符组成的字符串
        size:  图片大小
        background: 背景颜色
        '''
        self.text_list = list(text_str)
        self.size = size
        self.background = background

    def create_pic(self):
        '''
        创建一张图片
        '''
        self.width, self.height = self.size
        self.img = Image.new("RGB", self.size, self.background)
        # 实例化画笔
        self.draw = ImageDraw.Draw(self.img)

    def create_point(self, num, color):
        '''
        num: 画点的数量
        color: 点的颜色
        功能：画点
        '''
        for i in range(num):
            self.draw.point(
                (random.randint(0, self.width), random.randint(0, self.height)),
                fill=color
            )

    def create_line(self, num, color):
        '''
        num: 线条的数量
        color: 线条的颜色
        功能：画线条
        '''
        for i in range(num):
            self.draw.line(
                [
                    (random.randint(0, self.width), random.randint(0, self.height)),
                    (random.randint(0, self.width), random.randint(0, self.height))
                ],
                fill=color
            )

    def create_text(self, font_type, font_size, font_color, font_num, start_xy):
        '''
        font_type: 字体
        font_size: 文字大小
        font_color: 文字颜色
        font_num:  文字数量
        start_xy:  第一个字左上角坐标,元组类型，如 (5,5)
        功能： 画文字
        '''
        font = ImageFont.truetype(font_type, font_size)
        ran_text = " ".join(random.sample(self.text_list, font_num))
        self.draw.text(start_xy, ran_text, font=font, fill=font_color)
        return ran_text

    def opera(self):
        '''
        功能：给画出来的线条，文字，扭曲一下，缩放一下，位移一下，滤镜一下。
        就是让它看起来有点歪，有点扭。
        '''
        params = [
            1 - float(random.randint(1, 2)) / 100,
            0,
            0,
            0,
            1 - float(random.randint(1, 10)) / 100,
            float(random.randint(1, 2)) / 500,
            0.001,
            float(random.randint(1, 2)) / 500
        ]
        self.img = self.img.transform(self.size, Image.PERSPECTIVE, params)
        self.img = self.img.filter(ImageFilter.EDGE_ENHANCE_MORE)

```

创建好这个类后，就来使用它，先生成带有噪音的背景，然后通过随机抽取的4个字符生成图片验证码的内容，接着扭曲、缩放一下字符内容，这样就可以得到一张简单的验证码，最后将答案和验证唯一ID返回，这个ID其实也是图片的名称，代码如下

```
def GetImageCode():
    '''
    获得图片验证码
    :return: right_text:正确答案  code_id:验证码唯一ID
    '''
    strings = "abcdefghjkmnpqrstwxyz23456789ABCDEFGHJKLMNPQRSTWXYZ"
    size = (150, 50)
    background = 'white'
    pic = Picture(strings, size, background)
    pic.create_pic()
    pic.create_point(500, (220, 220, 220))
    pic.create_line(30, (220, 220, 220))
    # 加载simsun.ttc字体文件，用于生产图片中的内容
    right_text = pic.create_text(STATICFILES_DIRS[0] + "/simsun.ttc", 24, (0, 0, 205), 4, (7, 7))
    pic.opera()
    # pic.img.show()
    # 随机字符串：随机字符+时间戳
    ran_str = ''.join(random.sample(string.ascii_letters + string.digits, 16)) + str(int(time.time()))
    pic.img.save(MEDIA_ROOT + '/codeimage/' + ran_str + '.png')
    return {'right_text':right_text,'code_id':ran_str}


if __name__ == '__main__':
 # 自己可以简单测试一下
    GetImageCode()
```

获得验证码图片的方法写完了，接着就来写相应的接口，将代码写到 users/views.py中，因为要获得图片验证码不需要什么参数字段，所以直接通过GET方法获得就好了，使用刚刚在makgeimage.py中定义的GetImageCode()方法，每当一个GET请求过来时就调用这个方法，将正确的答案和ID保存到数据库，将图片的路径和ID返回给前端，具体代码如下

```python
class ImageCodeView(APIView):
    '''
    获得图片验证码
    '''
    def get(self, request, *args, **kwargs):
        codedict = GetImageCode()
        codeid = codedict.get('code_id',0)
        reight_text = codedict.get('right_text',0)
        reight_text = reight_text.replace(' ','')
        imagemodel = ImageCode(codeid=codeid, code= reight_text)
        imagemodel.save()
        re_dict = {'imagepath':'/media/codeimage/'+codeid+'.png','codeid':codeid}
        return Response(re_dict, status=status.HTTP_201_CREATED)
```

接着配置一下url，写到urls.py中

```
from django.conf.urls import url,include
from django.views.static import serve

# 图片验证码
    url(r'^imagecode', ImageCodeView.as_view(), name='imagename'),
    # 访问图片URL
    url(r'^media/(?P<path>.*)$', serve, {'document_root': MEDIA_ROOT}),
```
这里处理配置imagecode这个接口的url外，还需要**配置访问生成的验证码图片的接口，如果不配置这个接口，图片验证码的图片就无法通过URL来访问**

这样就可以获得图片验证码了，我们可以通过postman来访问一下这个接口

![](http://onxxjmg4z.bkt.clouddn.com/%E8%8E%B7%E5%BE%97%E5%9B%BE%E7%89%87%E9%AA%8C%E8%AF%81%E7%A0%81.jpg)

将图片的路径复制下来，放到浏览器上，看看图片的效果

![](http://onxxjmg4z.bkt.clouddn.com/%E5%9B%BE%E7%89%87%E5%8F%AF%E4%BB%A5%E8%AE%BF%E9%97%AE.jpg)

通过Navicat查看一下图片数据表

![](http://onxxjmg4z.bkt.clouddn.com/%E5%9B%BE%E7%89%87%E6%95%B0%E6%8D%AE%E5%BA%93.jpg)

## 短信验证码
有了图片验证码，就可以来实现短信验证码了，因为每次发短信都要验证图片验证码是否正确，同样将逻辑写到 users/views.py 中

```
lass SmsCodeViewset(mixins.CreateModelMixin, viewsets.GenericViewSet):
    """
    发送短信验证码
    """
    serializer_class = SmsSerializer

    def generate_code(self):
        """
        生成四位数字的验证码
        :return:
        """
        seeds = "1234567890"
        random_str = []
        for i in range(4):
            random_str.append(choice(seeds))
        return "".join(random_str)

    def codecheck(self,code, codeid):
        '''
        验证图片验证码，防止恶意用户狂注册
        :param code: 用户填写的验证码
        :param codeid: 验证码ID
        :return:
        '''
        five_mintes_ago = datetime.now() - timedelta(hours=0, minutes=5, seconds=0)
        right_code = ImageCode.objects.filter(codeid=codeid)
        # lte 小于等于 gte 大于等于
        tright_code = right_code.exclude(add_time__gte=five_mintes_ago)
        if not tright_code:
            return 2
        if right_code:
            if str(right_code[0]).lower() != str(code).lower():
                return 0
        else:
            return 0

        return 1


    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        # 序列化和验证
        serializer.is_valid(raise_exception=True)
        mobile = serializer.validated_data['mobile']
        data = request.data
        imagecode = request.data.get('code',0)
        imagecodeid = request.data.get('codeid',0)

        image_c = self.codecheck(code=imagecode, codeid=imagecodeid)

        if image_c ==2:
            return Response(data={'error':'图片验证码超时'}, status=status.HTTP_400_BAD_REQUEST)
        elif image_c == 0:
            return Response(data={'error': '图片验证码错误'}, status=status.HTTP_400_BAD_REQUEST)
        yun_pian = YunPian(APIKEY)

        code = self.generate_code()

        sms_status = yun_pian.send_sms(code=code, mobile=mobile)

        if sms_status["code"] != 0:
            return Response({
                "data":'error'
            }, status=status.HTTP_400_BAD_REQUEST)
        else:
            code_record = VerifyCode(code=code, mobile=mobile)
            code_record.save()
            return Response({
                "data":'success'
            }, status=status.HTTP_201_CREATED)
```
这里使用了SmsSerializer类对前端传递的值进行序列化和验证，使用generate_code()方法随机生成用于短信发送的4位数验证码，使用chodecheck()方法检查前端传递过来的图片验证码，除了验证用户输入的是否正确，还加了验证码5分钟过期的时间限制，这里有几个细节要注意，**1.验证码验证的时间要比当前时间减去5分钟大。2.用户输入的验证码在对比时通过lower转换成小写，实现不区分大小写
    
接着看到我们重写的create()方法，该方法会被Django自动调用，在该方法中，我们使用get_serializer()方法将要序列化和验证的值传入，代码中将前端传递的所有值都传入，然后通过is_valid()方法判断序列化是否通过，通过了才会继续下面的逻辑，也就是将通过序列化的手机号提取出来，使用自己封装好的方法将手机号和要发送的验证码发送出去
    
这里先来看一下SmsSerializer中的代码，看看序列化到底干了啥？
    
在users下创建名为serializer的py文件，该文件专门用于写序列化代码，看到SmsSerializer类的代码，其实就是对手机号进行验证判断是否合法，是否已经注册过了，发送短信的频率是否过快

```
import re
from datetime import datetime, timedelta

from rest_framework import serializers
from django.contrib.auth import get_user_model
from rest_framework.validators import UniqueValidator

from SMSReg.settings import REGEX_MOBILE
from .models import VerifyCode,ImageCode

# 可直接获得用户
User = get_user_model()

# 继承最基本的Serializer
class SmsSerializer(serializers.Serializer):

    mobile = serializers.CharField(max_length=11)

    def validate_mobile(self, mobile):
        """
        验证手机号码
        :param data:
        :return:
        """
        # 手机是否注册
        if User.objects.filter(mobile=mobile).count():
            raise serializers.ValidationError("用户已经存在")

        # 验证手机号码是否合法
        if not re.match(REGEX_MOBILE, mobile):
            raise serializers.ValidationError("手机号码非法")

        # 验证码发送频率
        one_mintes_ago = datetime.now() - timedelta(hours=0, minutes=1, seconds=0)
        if VerifyCode.objects.filter(add_time__gt=one_mintes_ago, mobile=mobile).count():
            raise serializers.ValidationError("距离上一次发送未超过60s")

        return mobile
```

接着来看通过云片网发送短信的代码，非常简单

```python
import json
import requests


class YunPian(object):

    def __init__(self, api_key):
        self.api_key = api_key
        self.single_send_url = "https://sms.yunpian.com/v2/sms/single_send.json"

    def send_sms(self, code, mobile):
        parmas = {
            "apikey": self.api_key,
            "mobile": mobile,
            "text": "【微宽量化】您的验证码是{code}（5分钟内有效，如非本人操作，请忽略本短信）".format(code=code)
        }

        response = requests.post(self.single_send_url, data=parmas)
        re_dict = json.loads(response.text)
        return re_dict
```
这里需要获得你云片网的API Key，大家注册一下就有了，然后就是使用固定的格式来发送信息，**前面也强调了，这里的text，必须跟云片网上申请的短信模板一模一样，不然短信回发送失败**，通过requests对云片网提供的URL进行POST请求，就将短信发送了，可以将云片网返回的数据保存到字典中，用于判断短信是否发送成功

这样短信发送和图片验证码验证的逻辑就写完了，老规矩，配置一下URL就好了

```
from rest_framework.routers import DefaultRouter

from SMSReg.settings import MEDIA_ROOT
from users.views import UserViewset,SmsCodeViewset,ImageCodeView

router = DefaultRouter()

# 短信验证码接口
router.register(r'codes', SmsCodeViewset, base_name="codes")
```

同样通过postman来测试一下这个接口

先通过上一个接口获得的图片验证码，填写正确的答案和验证码ID，再将手机号传入，通过POST方法访问短信验证码接口

![](http://onxxjmg4z.bkt.clouddn.com/%E5%8F%91%E9%80%81%E7%9F%AD%E4%BF%A1.jpg)

可以看到发送短信成功，打开手机查看一下，果真有条短信

![](http://onxxjmg4z.bkt.clouddn.com/%E7%9F%AD%E4%BF%A1%E9%AA%8C%E8%AF%81%E7%A0%812.jpeg)

查看一下数据库中的短信表，发现有一条与之对应的数据

![](http://onxxjmg4z.bkt.clouddn.com/%E7%9F%AD%E4%BF%A1%E9%AA%8C%E8%AF%81%E7%A0%81.jpg)

## 用户注册
做完了图片验证码和短信验证码，相当于做完了前期工作，接着才是用户注册的逻辑，用户注册要查表判断用户填写的短信是否正确，同样在 users/views.py 中写逻辑

```python

class UserViewset(mixins.CreateModelMixin, mixins.UpdateModelMixin, mixins.RetrieveModelMixin,
                  viewsets.GenericViewSet):
    '''
    用户
    '''
    serializer_class =  UserRegSerializer
    queryset = User.objects.all()

    def create(self, request, *args, **kwargs):
        # 将post过来的数据传给UserRegSerializer进行序列化和验证
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        user = self.perform_create(serializer)
        re_dict = serializer.data
        re_dict["name"] = user.name if user.name else user.username
        re_dict["success"] = '注册成功'

        headers = self.get_success_headers(serializer.data)
        return Response(re_dict, status=status.HTTP_201_CREATED, headers=headers)

    def get_object(self):
        return self.request.user

    def perform_create(self, serializer):
        return serializer.save()
    ```
    
看上去代码量似乎比前面的少，这也就Django Rest Framework的优势，它已经为我们提供了很多常用的操作，如minins中已经实现好了相关的增删改查操作，让UserViewset用户类的代码变得非常少，但是却依旧有很多功能，这些代码没什么特别的，就是通过UserRegSerializer对前端数据传递过来的数据进行序列化和验证，验证通过，则使用perform_create()方法调用serializer.save()将数据保存一下

重点是UserRegSerializer中发生了什么？

接着看到serializer.py中的UserRegSerializer类

```python
class UserRegSerializer(serializers.ModelSerializer):
    # 这里使用了write_only是因为后面我们将code字段通过del方法删除了，所以在序列化时，不使用write_only字段会报错，
    code = serializers.CharField(required=True, write_only=True, max_length=4, min_length=4, label="验证码",
                                 error_messages={
                                     "blank": "请输入验证码",
                                     "required": "请输入验证码",
                                     "max_length": "验证码格式错误",
                                     "min_length": "验证码格式错误"
                                 }, help_text="验证码")
    # 图片验证码
    imagecode = serializers.CharField(required=True, write_only=True, max_length=4, min_length=4, label="图片验证码",
                                      error_messages={
                                          "blank": "请输入图片验证码",
                                          "required": "请输入图片验证码",
                                          "max_length": "图片验证码格式错误",
                                          "min_length": "图片验证码格式错误"
                                      }, help_text="图片验证码")
    imagecodeid = serializers.CharField(required=True, write_only=True, label='图片验证码ID')

    # UniqueValidator:验证数据集中数据的唯一性，用户不能重复注册
    username = serializers.CharField(label='手机号', required=True, allow_blank=False,
                                     validators=[UniqueValidator(queryset=User.objects.all(), message='用户已存在')])

    password = serializers.CharField(style={'input_type':'password'}, label='密码', write_only=True)

    def create(self, validated_data):
        # 继承
        user = super(UserRegSerializer, self).create(validated_data=validated_data)
        # 加密密码
        user.set_password(validated_data['password'])
        user.save()
        return user


    def validate_imagecode(self,imagecode):
        # initial_data是没有经过serializers处理过的数据,post中需要传入imagecodeid
        image_code = ImageCode.objects.filter(codeid=self.initial_data['imagecodeid'])
        five_mintes_ago = datetime.now() - timedelta(hours=0, minutes=5, seconds=0)
        timage_code = image_code.exclude(add_time__gte=five_mintes_ago)
        if not timage_code:
            raise serializers.ValidationError("图片验证码超时")
        if image_code:
            k = str(image_code[0])
            k2 = str(image_code[0]).lower()
            if str(image_code[0]).lower() != imagecode.lower():
                raise serializers.ValidationError("图片验证码错误")
        else:
            raise serializers.ValidationError("图片验证码错误")


    def validate_code(self, code):
        # post中需要传入username
        verify_records = VerifyCode.objects.filter(mobile=self.initial_data["username"]).order_by("-add_time")
        if verify_records:
            last_record = verify_records[0]
            # 5分钟有效期
            five_mintes_ago = datetime.now() - timedelta(hours=0, minutes=5, seconds=0)
            if five_mintes_ago > last_record.add_time:
                raise serializers.ValidationError("验证码过期")
            if last_record.code != code:
                raise serializers.ValidationError("验证码错误")
        else:
            raise serializers.ValidationError("验证码错误")

    def validate(self, attrs):
        '''
        对序列化的字段统一操作，删除不需要的验证存库的字段
        '''
        attrs['mobile'] = attrs['username']
        del attrs['code']
        del attrs['imagecode']
        del attrs['imagecodeid']
        return attrs

    class Meta:
        model = User
        fields = ("username", "code", "mobile", "password", 'imagecode', 'imagecodeid','name')
```

这里有几点值得注意
1.code、imagecode、imagecodeid、password都使用write_only字段，该字段标识只写，也就是不能读，将其设置为True以确保在更新或创建实例时可以使用该字段，但在序列化表示时不包括该字段,默认为False

2.username字段中对validators使用了UniqueValidator，DjangoRestFramework中还有多个可以这样使用的函数，这里UniqueValidator函数的作用就是让前端传递过来的username在User.objects.all()查询的结果集中唯一，目的就是保证这个用户不是重复注册

3.password字段设置了style，将style设置成{'input_type':'password'}可以让用户使用前端界面输入时是密文的形式，这里的前端界面是DjangoRestFramework自带用于测试接口的前端

4.重写了create()方法，在该方法中，我们继承了默认create()方法，然后使用了set_password()方法将密码密文保存

5.validate_imagecode()和validate_code()方法都是validate_字段名的格式，这样形式的方法会针对某个字段，如validate_imagecode就针对imagecode字段生效，validate_code针对code字段生效，这两个方法中的逻辑跟前面图片验证码和短信验证码的逻辑类似

6.validate()方法对所有字段生效，在这个方法中，我们将username中的值赋值给mobile，前端可以不传递mobile，但是必须传递username，然后将code、imagecode、imagecodeid删除，因为UserProfile表中没有这几个字段，如果不删除在使用serializer.save()保存时就会报错

最后配置一下url

```
# 用户接口
router.register(r'users', UserViewset, base_name="users")
```

同样通过postman来测试一个注册接口，将短信和图片验证码都正确的填上

可以看到注册成功，返回了用户填写的一些信息
![](http://onxxjmg4z.bkt.clouddn.com/%E6%B3%A8%E5%86%8C%E6%88%90%E5%8A%9F2.png)

查看一下UserProfile用户信息表，发现了相应内容
![](http://onxxjmg4z.bkt.clouddn.com/%E5%AF%86%E7%A0%81%E5%AF%86%E6%96%87.png)


## 自定义登录
这里再提一下自定义登录，跟本文有一定的联系，因为使用了短信注册，那么后期登录肯定是要使用手机号进行登录，Django默认只能通过username，所以自定义一下登录规则，非常简单，将代码写到 users/views.py中

主要就是重写了authenticate，通过Q方法实现登录的并集逻辑

```
class CustomBackend(ModelBackend):
    def authenticate(self, request, username=None, password=None, **kwargs):
        try:
            user = User.objects.get(Q(username=username)|Q(mobile=username))
            if user.check_password(password):
                return user

        except Exception as e:
            return None
```

## 结尾
短信注册时很多web系统都有的部分，本文的代码写到不算精美，还有很多可以优化的地方，这就交个大家了

项目完整代码放到了Github上，各位点个Start
[Github Django实现用户短信注册 DjangoSMSReg](https://github.com/ayuLiao/DjangoSMSReg)





