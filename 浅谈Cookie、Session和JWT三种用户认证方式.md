# 浅谈Cookie、Session和JWT三种用户认证方式

## 简介
因为HTTP是无状态的协议，也就是说，我们无法通过HTTP来标识出不同的用户，可是有些资源我们只想返回给特定的用户，比如说用户账号的个人信息，此时就需要验证该请求的身份了，判断是不是特定用户，此时就需要用户认证，判断出不同的用户。
常见的用户认证有cookie、session和JWT（Token）这三种形式，本篇文章来谈谈这三种形式

## Cookie用户认证
cookie算是最早的用户认证机制了，现在几乎见不到了，它消失的原因也简单，就是不安全，非常不安全，cookie认证的流程很简单，服务器会将用户的账号和密码返回给用户，用户存到cookie中就好了，那么下次用户再发HTTP请求给服务器时，带上账号和密码就可以了，服务器接收到了用户发送过来的请求，通过其中的账号和密码就可以判断出这个请求是不是特定用户发送出来的，当服务器确定了用户的身份后就可以将该用户私人的数据返回了，比如返回该用户的订单数据等等

可以看出这种方式非常粗暴，理解起来很简单，但是将账号和密码直接放到Cookie中明显是不安全的，因为其他人可以直接查看Cookie，如图

![cookie保存密码.png](http://onxxjmg4z.bkt.clouddn.com/cookie%E4%BF%9D%E5%AD%98%E5%AF%86%E7%A0%81.png)

那么我们将密码加密然后再保存到Cookie中不就得了？Too Young Too simple，虽然密码加了密，但是Cookie容易被人获得，当别人获得了你的Cookie，就可以拿到账号密码，他就可以直接使用获得的账号密码来登录你的账号了

## Session用户认证
因为Cookie那么不安全，所以后面就出现了Session，Session算是比较常见的用户认证机制了，先来理一理Session用户认证的过程。

1.首先服务器会建立一张表，这里简单的叫session表
2.当用户登录时，服务器查用户表判断用户是否合法
3.如果用户认证通过，就会将用户的一些信息存放到session表中，如账号、密码之类的，同时将一个Session ID返回给客户端，这个ID是唯一的，通过这个ID就可以查找session表中对应的信息
4.客户端将获得的ID存到Cookie中，在下次请求时，将Cookie中的ID带到请求中
5.服务器收到客户端发送的请求，获得其中的Session ID，通过Session ID来查找Session表，获得用户的信息，并验证用户的身份。

上面就是Session的一个机制，当然不同语言实现的方式可能有些出入，但是原理就是这样的，Session的认证过程其实也不复杂，那么相对于Cookie，它的优势在哪？一个致命的优势就是比Cookie安全，我们可以发现，虽然Session中也将数据存放都客户端的Cookie中，但是这些数据不是敏感数据，就算hacker拿到了这些数据，也不能一直使用，它有个过期时间，过了过期时间，就可以将Session表中相应的字段删除，这样通过过期的Session ID就无法通过用户身份验证了，而如果将用户的账号密码存到Cookie中，hacker获得后，就可以一直使用。

当然Session还是有一些问题：

1.Session机制是服务端的机制，每次用户认证都要查Session表或者在Session表增添记录，当用户量比较大时，会给服务器带来比较大的压力，而且通常来讲，Session表都是保存到内存中，这样可以加快搜索速度，但是用户基数太大，服务端的内存开销就会很大

2.扩展性比较差，如果Session表保存到内存中，那么用户认证成功后，下次请求必须还是这台服务器做响应，降低了分布式服务集群的优势，扩展性差

3.前面也提了，hacker拿到了Cookie中唯一的Session ID依旧可以通过用户认证，虽然用过期时间，但是在这段时间内，hacker该做的也做完了

## JWT用户认证
前面介绍了那么多，就是为了引出今天的主角，Json Web Token，简称JWT。因为Session的各种缺点，人们就想着有没有更好的方法，比如服务器生产遵守一定规律的ID，简单计算这个ID就可以认证用户身份并且保证安全之类的，有，JWT就是这样的一个方法，我们一般将JWT生成的具有规律的ID叫做Token

JWT是一个开放标准，它规定通信双方使用JSON对象的形式进行安全通信，它可以使用非对称加密算法对数据进行加密，保证数据的安全

### JWT结构
JWT由三个部分组成，分别是Header头部、Payload载荷、Signature签名，三个部分的具体结构如图

![JWT结构.jpg](http://onxxjmg4z.bkt.clouddn.com/JWT%E7%BB%93%E6%9E%84.jpg)

简单来介绍一下这三个结构

首先是Header 头部

头部一般包含两个部分，typ定义JWT的类型，alg定义JWT使用的加密算法

```
{
    "typ":"JWT",
    "alg":"HS256"
}
```
可以看出是Header是JSON的格式，然后使用Base64算法对Header进行编码，编码后的内容就是JWT结构的第一部分

Base64算法是可以轻松解密的，所以严格说Base64不能算是加密过程，只能算编码过程

Header经过Base64编码后可以获得如下数据
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

接着看Payload 载荷
其实载荷就是用来放信息的地方，我们可以将用户的各种非敏感信息放到payload中，JWT规范中对这部分的字段做了比较详细的介绍

```
{
    "iss": "ayuliao JWT",
    "iat": 1652323456,
    "exp": 1652328456,
    "aud": "www.lmwen.com",
    "sub": "ayuliao@xxx.com"
}
```
其中
iss：这个Token的签发者
iat：签发时间
exp：过期时间，过期时间必须在当前时间之后
aud：接受方
sub：Token的主题

payload部分的数据也是JSON格式，同样要通过Base64对这部分数据进行编码，编号后的数据如下(可以自己解码一下)
```
eyJpc3MiOiAiYXl1bGlhbyBKV1QiLCJpYXQiOiAxNjUyMzIzNDU2LCAiZXhwIjogMTY1MjMyODQ1NiwgImF1ZCI6ICJ3d3cubG13ZW4uY29tIiwic3ViIjogImF5dWxpYW9AeHh4LmNvbSJ9
```

最后是Singnature 签名
Singnature比Header和Payload复杂点，需要将上面我们通过Base64编码后得到的Header和Payload通过 . 号链接起来构成一个字符串，在使用Header定义的加密算法对该字符串进行加密，加密时还必须加盐，增加反解密的难度，这个盐由服务端来定义，这里就使用HS256算法进行加密，盐是ayuliao

```
var encodedString = base64UrlEncode(header) + "." + base64UrlEncode(payload);
HMACSHA256(encodedString, 'ayuliao');
```

加密完后，得到的字符串如下

```
a4e049dfe99def8217a2fa9e48d0574b90e4902335ad66dd53a69bb71be19e51
```

进行签名的目的是为了保证JWT没有被人篡改过，保证信息的安全性和可靠性，因为Header和Payload只是简单的使用了Base64进行编码，很容易被人解码，那么他们就可以修改其中的内容，再进行Base64编码，然后发送篡改后的数据给我们，那么此时服务端可以判断出Header和Payload形成的签名和正常的Singnature签名是不一样的。因为篡改Header和Payload人不知道服务器生成的盐，所以得出签名就是不一样的


获得了JWT的三部分后，将这三部分通过 . 号连接在一起就获得了JWT，完整的JWT如下

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiAiYXl1bGlhbyBKV1QiLCJpYXQiOiAxNjUyMzIzNDU2LCAiZXhwIjogMTY1MjMyODQ1NiwgImF1ZCI6ICJ3d3cubG13ZW4uY29tIiwic3ViIjogImF5dWxpYW9AeHh4LmNvbSJ9.a4e049dfe99def8217a2fa9e48d0574b90e4902335ad66dd53a69bb71be19e51
```

## JWT的思考
通过前面的内容，我们都知道了Header和Payload中的数据是很容易就可以被解码获得的，那么放到这里的信息不就暴露了吗？

没错，Payload的目的其实就是暴露数据，不然在设计时也不会考虑使用Base64这种可以轻易被反解的算法，在Payload中，我们不应该放置用户的敏感信息，比如用户的密码之类的，可以将用户名或者用户ID之类的数据放到Payload中，这样在编写代码逻辑时，如果需要，自己就可以直接从Payload中获得用户数据，让代码逻辑变得简单些

从对比JWT和Session这两种验证方式也可以看出JWT的优点，Session因为是将用户信息存到服务端，所以在用户基数大时，占用了服务器资源，对比较大型的应用，可能需要在Session表中存很多状态，而JWT方法将用户状态分散到了客户端中，存到payload中，这样可以减轻服务器的压力，虽然JWT在服务端进行验证时需要进行计算，还有就是Session的扩展性比较差，只适合单点登录，而JWT没有这个问题

在网上找到一个JWT用户认证的流程图，在这里贴一下

![JWT用户认证](http://onxxjmg4z.bkt.clouddn.com/JWT%E7%94%A8%E6%88%B7%E8%AE%A4%E8%AF%81.jpg)

>1.首先，前端通过Web表单将自己的用户名和密码发送到后端的接口。这一过程一般是一个HTTP POST请求。建议的方式是通过SSL加密的传输（https协议），从而避免敏感信息被嗅探。

>2.后端核对用户名和密码成功后，将用户的id等其他信息作为JWT Payload（负载），将其与头部分别进行Base64编码拼接后签名，形成一个JWT。形成的JWT就是一个形同lll.zzz.xxx的字符串。

>3.后端将JWT字符串作为登录成功的返回结果返回给前端。前端可以将返回的结果保存在localStorage或sessionStorage上，退出登录时前端删除保存的JWT即可。

>4.前端在每次请求时将JWT放入HTTP Header中的Authorization位。(解决XSS和XSRF问题)

>5.后端检查是否存在，如存在验证JWT的有效性。例如，检查签名是否正确；检查Token是否过期；检查Token的接收方是否是自己（可选）。

>6.验证通过后后端使用JWT中包含的用户信息进行其他逻辑操作，返回相应结果。

整个流程就是这样，可以看看原文[前后端分离之JWT用户认证](http://lion1ou.win/2017/01/18/)

## 总结
能用JWT就用JWT吧，如果你使用Django做前后端分离开发，推荐使用[django-rest-framework-jwt](https://github.com/ayuLiao/django-rest-framework-jwt)，通过Github方法下载源码，以源码的方式集成到项目中，方便修改源码，满足自己对JWT的需要



