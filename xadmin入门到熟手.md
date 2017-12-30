# xadmin入门到熟手

## 简介
如果你开发过Django项目肯定会知道Django自带了一个后台管理系统admin，其实admin已经十分强大了，只需要简单配置一下，就可以直接使用了，xadmin基于Django默认的admin，但是它的功能相对于默认的admin更加强大，同样通过配置就可以直接使用，而且可以使用的功能更多，这篇文章就来讲讲xadmin中的使用技巧

目前xadmin这个项目在Github上已经有一段时间没有更新了，但是它的代码依旧有很大的参考价值，看完这篇文章体会了它的强大功能后你会爱不释手，当我们熟悉了xadmin后就可以简单的修改它的代码，让它适配最新的Django和Python

为什么标题不是xadmin入门到精通？因为我觉得只是会有还不能称为精通

## xadmin替代admin
首先创建一个Django项目，在Django项目中创建extra_apps文件夹，该文件夹专门用于放第三方插件代码，将xadmin的代码从[Github](https://github.com/sshwsfc/xadmin)上下载下来，放到extra_apps文件夹中，并将该文件夹设置为Sources Root

将文件夹设置为Sources Root的方式：选中extra_apps文件夹-->左击-->选中Mark Directory as-->Sources Root

将文件夹设置为Sources Root，以后在项目中就可以直接引用extra_apps中的python文件了，而不必加文件名前缀，简单说就是可以通过xadmin直接使用，而不必extra_apps.xadmin的形式来使用

![xadminproject.png](http://onxxjmg4z.bkt.clouddn.com/xadminproject.png)

但是单纯的将extra_apps设置为Sources Root还不够，这只是让PyCharm这个IDE知道可以直接使用extra_apps文件夹中的内容，如果在命令行直接通过python来运行已经报找不到xadmin的错误，要解决这个问题，我们需要在settings.py文件中声明一下extra_apps文件夹的路径

```
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
# 添加下面两句，声明extra_apps文件夹的路径
sys.path.insert(0, BASE_DIR)
sys.path.insert(0, os.path.join(BASE_DIR, 'extra_apps'))
```

###### 其实除了去Github上直接下载xadmin，还可以直接通过pip来安装xadmin

```
pip install xadmin
```

但是不推荐使用pip安装，因为pip安装的并不是最新的代码，而且通过pip安装不方便修改xadmin中的源代码

接着通过startapp创建名为users的应用，在users应用下使用xadmin

要在users下使用xadmin首先需要在settings.py中声明两者xadmin和crispy_forms

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'users',
    'xadmin',
    'crispy_forms'
]
```

为了方便使用，将Django链接数据库的方式改为最为熟悉的MySQL，并且将系统默认的语言设置为中文

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'bookstore',
        'USER':MySQL用户名,
        'PASSWORD':MySQL密码,
        'HOST':'127.0.0.1',
    }
}

LANGUAGE_CODE = 'zh-hans'

TIME_ZONE = 'Asia/Shanghai'

USE_I18N = True

USE_L10N = True
#默认是Ture，时间是utc时间，由于我们要用本地时间，所用手动修改为false
USE_TZ = False
```

作为上面的配置，就可以在users模块下创建adminx.py的python文件，这个文件名是xadmin规定的，它会自动在相应的模块下加载adminx.py文件

![](http://obfs4iize.bkt.clouddn.com/adminx%E6%96%87%E4%BB%B6.png)

首先可以在adminx.py中写上一些默认的配置，如xadmin后台的样式选择，后台显示的标签和后台的页脚等

```
import xadmin
from xadmin import views

# 开启后台主题样式选择
class BaseSetting(object):
    enable_themes = True
    user_bootswatch = True


# 后台全局设置
class GlobalSettings(object):
    # 后台标签
    site_title = 'AYULIAO的后台'
    # 后台页脚
    site_footer = 'AYULIAO'
    
xadmin.site.register(views.BaseAdminView, BaseSetting)
xadmin.site.register(views.CommAdminView, GlobalSettings)
```

最新的代码主题样式选择其实失效了，但是这个对于轻样式的后台来说，没什么影响

为了可以访问，需要修改一下urls.py文件，该文件主要用于放Django的全局路由

```
from django.conf.urls import url, include
import xadmin

urlpatterns = [
    url(r'^xadmin/', xadmin.site.urls),
]
```

这样运行一下，效果如图：

![](http://obfs4iize.bkt.clouddn.com/xadmin%E5%90%8E%E5%8F%B0.png)

## 创建模块
Django有强大的ORM功能，所以我们不必去写SQL，通过Django的ORM机制，可以很轻松的使用数据库，当然使用Django ORM来操作MySQL当然没有最原始的SQL快

这里我们创建3个模块，分别是作者、书籍和出版社，其中作者和书籍是多对多的关系，一个作者可以写多本书，一本书也可以由多个作者一同合著，而书籍和出版社是一对多的关系，一本书籍只能由一个出版社印刷出书，而一个出版社却可以出多本书籍，在后面细谈Django ORM机制的文章中也会使用这3个模型

第一个是作者模块

```
class Author(models.Model):
    '''
    作者表
    '''
    name = models.CharField(max_length=15, verbose_name='作者名', null=True, blank=True)
    age = models.IntegerField(default=0, verbose_name='作者年龄')
    # 星星越多，后台排名越前
    star = models.IntegerField(default=0, verbose_name='星星')
    # 被封杀的作者后台不看见
    is_block = models.BooleanField(default=False, verbose_name='封杀')
    # 该作者写过多少本书
    booknums = models.IntegerField(default=0, verbose_name='书籍数量')
    add_time = models.DateTimeField(default=datetime.now, verbose_name="添加时间")

    # xadmin显示
class Meta:
    verbose_name = '作者'
    verbose_name_plural = verbose_name

    #  在xadmin中新增和修改时的显示
def __str__(self):
    return self.name
```
作者用于作者名、作者年龄、星星数、是否被封杀、书籍数等属性

verbose_name是后台显示这张表的名字，同时将复数形式的verbose_name_plural也定义成verbose_name，这样复数形式和单数形式都显示作者，在代码中，还重写了__str__(self)方法，该方法用于xadmin修改数据时显示

![](http://obfs4iize.bkt.clouddn.com/xadmin%E4%BF%AE%E6%94%B9.png)

然后是书籍模块

```
class Book(models.Model):
    '''
    书籍表
    '''
    bookname = models.CharField(max_length=15, verbose_name='书名', null=True, blank=True)
    author = models.ManyToManyField(Author, verbose_name='本书作者')
    des = UEditorField(verbose_name=u'书籍描述', width=600, height=300, toolbars="full", imagePath="book/ueditor/", filePath="book/ueditor/",
                           upload_settings={"imageMaxSize": 1204000},)
    add_time = models.DateTimeField(default=datetime.now, verbose_name="添加时间")

class Meta:
    verbose_name = '书籍'
    verbose_name_plural = verbose_name

def __str__(self):
    return self.bookname
```
数据模块通过ManyToManyField关键字与Author模块进行多对多的管理，其实Django就会通过第三张表来记录这种多对多的关系
    
![](http://obfs4iize.bkt.clouddn.com/%E5%A4%9A%E5%AF%B9%E5%A4%9Axadmin.png)

出版社模块

```
class Press(models.Model):
        '''
        出版社表
        '''
    pressname = models.CharField(max_length=15, verbose_name='出版社的名称', null=True, blank=True)
    book = models.ForeignKey(Book, verbose_name='出版社出版的书')
    url = models.CharField(max_length=100, verbose_name='出版社主页')
    add_time = models.DateTimeField(default=datetime.now, verbose_name="添加时间")

class Meta:
    verbose_name = '出版社'
    verbose_name_plural = verbose_name

def go_to(self):
    url = self.url
    aurl = "<a href='"+url+"'> 出版社主页 </>"
    from django.utils.safestring import mark_safe
    return mark_safe(aurl)
go_to.short_description =  '出版社主页'

def __str__(self):
    return self.pressname
```

    
通过ForeignKey关键字实现了书籍和出版社一对多的关系，这里还定义了go_to方法，该方法定义了一个url，并使用了Django的mark_safe方法让url可以显示出来，如果不使用mark_safe，那么所有的HTML代码都会被转译而不会被执行，这是在安全上的考虑，接着为该方法定义short_description，其实就是xadmin中的显示名
    
xadmin不只可以显示数据库中的字段，而且可以显示方法

编写好模块后，PyCharm命令行中使用下面的命令，让Django中的模块在MySQL数据库中生成对应的数据表

```
makemigrations
&&
migrate
```

## 熟练使用xadmin

生成好对应的模块后就可以跟xadmin愉快的玩耍了，xadmin有很多易用的功能，打开adminx.py文件，将与xadmin相关的代码写到其中

首先是作者模块对于的xadmin

```
class AuthorAdmin(object):
    # 列表显示
    list_display = ['name','age','booknums','star','is_block','add_time']
    # 搜索范围
    search_fields = ['name','age','booknums','star','is_block']
    # 列表过滤
    list_filter = ['name','age','booknums','star','is_block','add_time']
    # 默认排序,'-':倒序,从大到小
    ordering = ['-booknums']
    # 只读
    readonly_fields = ['star']
    # 刷新秒数
    refresh_times = [3,5]
    # 直接编辑
    list_editable = ['age','name']

    def queryset(self):
        '''
        过滤，将被封杀的作者过滤掉
        :return:
        '''
        qs = super(AuthorAdmin, self).queryset()
        qs = qs.filter(is_block=False)
        return qs
```

代码中有详细的解释
list_display定义xadmin要显示的内容，将一个list赋值给这个变量，需要注意**变量名是固定的，必须是list_display，不然xadmin认不出来，其实Django默认的admin也是这样的**

![](http://obfs4iize.bkt.clouddn.com/list_display.png)

search_fields定义xadmin可以搜索的内容，比如可以对作者的姓名进行搜索，那就将name放到list中赋值给search_fields

list_filter就是过滤器了，定义xadmin的过滤条件，让xadmin显示满足条件的数据

图中，使用ayuliao这个作者名进行搜索，出现一条结果，同时还展示了过滤功能，我可以通过作者名进行过滤、通过时间进行过滤

![](http://obfs4iize.bkt.clouddn.com/xadmin%E6%90%9C%E7%B4%A2%E5%92%8C%E8%BF%87%E6%BB%A4.png)

ordering定义xadmin的默认排序，如果我想让xadmin对作者出的书籍从多到少这样的顺序进行排序

readonly_fields让xadmin不能修改某个字段，在上段代码中，将作者的星星数定义成只读，其他字段都是可以修改的，如图

![](http://obfs4iize.bkt.clouddn.com/xadmin%E5%8F%AA%E8%AF%BB.png)

refresh_times定义xadmin刷新秒数，这里定义了3和5，也就是可以选择3秒刷新一次页面或者5秒刷新一次页面，可以在想快速看到数据表变化时使用

![](http://obfs4iize.bkt.clouddn.com/xadmin%E5%88%B7%E6%96%B0%E9%A1%B5%E9%9D%A2.png)

list_editable可以让xadmin对定义的字段直接进行编辑，不用进入编辑页面，一般来说，我们要编辑一个对象需要进入该对象的编辑页面，对于一些经常需要编辑的字段就会显得麻烦，可以将经常需要编辑的字段设置给list_editable

![](http://obfs4iize.bkt.clouddn.com/xadmin%E7%9B%B4%E6%8E%A5%E7%BC%96%E8%BE%91.png)

最后重写了queryset方法，该方法会返回数据表中对应的数据给xadmin，**重写它就可以实现数据表数据显示到xadmin需要进行的操作**，这里我们不想显示被封杀的作者，那么可以在queryset方法中实现相应的逻辑

在queryset()中，首先继承了父类AuthorAdmin的queryset()，然后使用filter()方法将is_block为Flase的过滤出来，也就是将没有被封杀的过滤出来，将过滤后的数据集返回，这样xadmin就不会显示被封杀的作者了

上面就完成了AuthorAdmin，虽然在这里不显示被封杀的作者，但是我想在一个单独的模块中显示这些被封杀的作者，有点像回收站，列表中删除的内容会出现在回收站中，我们可以进入回收站看见被我们删除的内容，当然也可以恢复它，那么在xadmin中怎么实现呢？

有人可能会想，单独构建一个模块对于一张表，专门用于存放被封杀的作者，然后通过xadmin显示出来，这种方法当然可行，但是这样就需要多维护一张表，其实有更好的办法，就是Django ORM模块代理的机制，同一张表通过代理的方式对于两个模块，这里我们需要定义另一个模块来显示被封杀的作者，但是不必创建新的数据表，依旧放在Author表中

回到models.py，定义BlockAuthor模块，继承Author

```
class BlockAuthor(Author):
    '''
    被封杀的作者，使用Modle的代理功能，不必重新写一个模块
    '''
    class Meta:
        verbose_name = '封杀作者'
        verbose_name_plural = verbose_name
        proxy = True
```
**在Meta类中将proxy设置为True，这个模块就成为了一个代理，这个代理是不会产生新的表的，但是功能跟其他模块一样，**Django ORM中非常强大的一个功能

虽然BlockAuthor是代理模块，但使用方式不变，依旧在adminx.py中编写BlockAuthor的配置代码，因为BlockAuthor继承自Author模块，所以用法跟Author模块一样，只是queryset()方法将被封杀的作者返回

```
class BlockAuthorAdmin(object):
    # 列表显示
    list_display = ['name','age','booknums','star','is_block','add_time']
    # 搜索范围
    search_fields = ['name','age','booknums','star','is_block']
    # 列表过滤
    list_filter = ['name','age','booknums','star','is_block','add_time']
    # 默认排序,'-':倒序,从大到小
    ordering = ['-booknums']
    # 只读
    readonly_fields = ['star']
    # 刷新秒数
    refresh_times = [3,5]
    # 直接编辑
    list_editable = ['age','name']

    def queryset(self):
        '''
        过滤出被封杀的作者
        :return:
        '''
        qs = super(BlockAuthorAdmin, self).queryset()
        qs = qs.filter(is_block=True)
        return qs
```

![](http://obfs4iize.bkt.clouddn.com/%E5%B0%81%E6%9D%80%E4%BD%9C%E8%80%85.png)

定义好了作者相关的内容，接着就来定义出版社

```
class PressAdmin(object):
    list_display = ['pressname', 'book', 'go_to']
    search_fields = ['pressname', 'book__bookname']
    list_filter = ['pressname', 'book__bookname', 'add_time']
```

出版社的xadmin配置代码与Author是类似的，只是search_fields和list_filter中使用了book__bookname这个属性，这是因为Press模块中定义了book这个外键指向Book模块，那么搜索或者过滤时就要说清楚搜索Book模块的什么值？不然xadmin就会报错，这里搜索和过滤都是通过书名来进行的

![](http://obfs4iize.bkt.clouddn.com/%E5%87%BA%E7%89%88%E7%A4%BExadmin.png)

接着就来写书籍的xadmin配置代码

```
class PressInline(object):
    model = Press
    extra = 0

class BookAdmin(object):
    list_display = ['bookname', 'author','add_time']
    # 搜索外键时，要明确搜索该外键哪个值
    search_fields = ['bookname', 'author__name']
    list_filter = ['bookname', 'author__name','add_time']
    # 富文本显示
    style_fields = {'des':'ueditor'}
    # 书籍和出版社是一对多关系
    inlines = [PressInline,]
```

list_display、search_fields、list_filter就不多说了，看到inlines关键字，它的作用是是将Press引入到Book的编辑界面，做界面的拼接，文字可能描述不明白，看图

![](http://obfs4iize.bkt.clouddn.com/%E4%B9%A6%E7%B1%8D%E5%87%BA%E7%89%88%E7%A4%BE.png)

在书籍的编辑界面，嵌入了出版社的编辑界面，这既是inlines关键字的作用，嵌入的方式是通过定义一个新的类，该类的model定义为Press模块，然后定义extra关键字为0

同时也要主要inlines定义在哪个模块？它是不用使用到多对多关系中的，比如书籍与作者是多对多的关系，那么在书籍模块的编辑界面就不能嵌入作者模块的编辑界面，同理作者模块的编辑界面也不能嵌入到书籍模块中，还需要主要，虽然它可以使用到一对多的情况下，但也只能在一那一边，如书籍和出版社是一对多的关系，一本书籍只能在一个出版社出版，但是一个出版社却可以出版不同的数据，那么书籍在一这边，而出版社在多这边，所以inlines只能使用到BookAdmin，而不能使用PressAdmin

上面的配置代码写完后，不要忘了注册一下

```
# 注册
xadmin.site.register(Author,AuthorAdmin)
xadmin.site.register(BlockAuthor,BlockAuthorAdmin)
xadmin.site.register(Book,BookAdmin)
xadmin.site.register(Press,PressAdmin)
```

此时运行程序，在127.0.0.1:8000/xadmin下就可以看到我们功能强大的后台了，当然要进入后台，你需要创建一个超级管理员，在项目的根目录下运行下面命令创建超级管理员

```
python manage.py createsuperuser
```

xadmin基础的用法就到这里了，掌握了上面的内容，你应该可以熟练的使用xadmin了，后面会讲讲如何编辑xadmin中的插件，比如在上面书籍的相关代码中，我使用了富文本插件，原始的xadmin中是没有这个插件的，我们需要自己编写这个插件，后面的文章会聊聊这个内容

![](http://obfs4iize.bkt.clouddn.com/xadmin%E5%AF%8C%E6%96%87%E6%9C%AC.png)

编程创造乐趣---ayuliao


    


