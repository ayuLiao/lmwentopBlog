# Django微信支付开发

## 简介
因为公司项目需要使用微信支付，所以就着手开发，一开始想着，文档微信给了，也很多人做了，应该很简单，是我太年轻了，深刻的体会到了微信支付文档对中华文化的巧妙运用，就是不跟你说明白，自己慢慢试，欲哭无泪，本篇文章简单整理了一下我的开发过程

微信支付有多种，我这里，使用的微信公众号支付，比如，公众号里开发了商城之类的运用需要支付都算是微信公众号支付


## 微信支付流程
虽然说很多人做过，网上也有很多代码可以让我们直接使用，但是不弄明白微信的支付流程，还是两眼黑的

直接上微信文档中的支付流程图，仔细看多几遍

![](http://onxxjmg4z.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%94%AF%E4%BB%98%E4%B8%9A%E5%8A%A1%E6%B5%81%E7%A8%8B.png)

从图中可以知道几个主要的步骤：

1.商户系统通过**统一下单接口API**请求微信支付系统，微信支付系统会**返回预付订单信息，其中就包括prepay_id**,prepay_id是非常重要的
2.通过prepay_id和其他关键信息做签名运算，将这些部分关键信息和签名运算后的结果返回（这里的说法比较暧昧，继续看下文就明白这些信息是什么？如何签名？）
3.前端获得这些关键信息后，就可以使用JSSDK将这些关键信息发送给微信支付系统，微信支付系统认证通过后，就会出现支付界面
4.支付成功后，微信支付系统会**异步**的返回用户支付结果通知，这里的通知前端后端都会获得，前端获得支付成功的通知可以做页面跳转操作，后端收到支付成功通知同样可以做一下操作

这里以商城为例，从一个实际的项目角度来看支付流程

1.前端会将用户购买商品的信息传递给后端，比如用户选购的商品ID，商品属性，商品单价等等，后端通过这些信息生成一个订单，订单的状态是待支付，然后返回生成订单成功的数据给前端

2.前端收到后端订单生成成功的信息后就跳转到支付界面，这个界面是前端实现的，在该界面中使用微信jssdk，通过jssdk来调用微信支付，前端使用jssdk前需要通过微信授权认证从而获得调用微信支付接口的权限

这一步要拆分成两个重要的步骤
a.前端要获得jssdk的权限，一般的做法是通过后端去调用微信的接口获得jssdk授权需要的配置信息，然后将这些配置信息返回给前端
b.前端通过后端返回的jssdk配置信息请求微信jssdk的授权接口，拿到jssdk的使用权限，只有拥有了使用权限才有资格去请求微信支付的接口
c.前端请求微信支付接口同样需要相应的信息，这些信息同样通过后端返回，后端此时就需要去请求微信的统一下单接口，获得统一下单接口返回的信息，通过这些信息进行签名，然后将签名信息和通过统一下单接口返回的信息都返回给前端
d.前端获得了后端返回的关于微信支付的信息后，就可以使用jssdk唤醒微信支付了，微信支付的付款界面和输入密码的界面完全不用我们管，jssdk已经帮我们实现好了

3.通过了步骤2，调用了微信支付后，用户付款成功、失败或者中途取消微信支付都会有个回调信息，这个信息是异步的，前端后端都会收到，前端收到就可以做一些支付后页面跳转的操作，而后端就做支付后的逻辑操作，比如将订单设置为支付成功，给用户安排发货的，因为这里是比较重要的逻辑，所以一定要验证回调接口的信息是不是微信支付系统返回的信息

上面就是大致的步骤，但是还有很多细节没有讲清楚，接着就通过代码来讲这些细节

## 微信支付必要配置
要调用微信支付，除了代码要写对之外，最重要的事情就是必要的配置必须配了，而且必须配置正确，不然调用微信支付就会错误，这里不得不吐槽一下，第一次调用微信支付失败应该是很正常的，那么失败的报错提醒应该是一个比较准确的信息，方便我们查错，但是微信就只告诉你，这里错了，为什么？有很多种原因，除了自己一个个试，没有啥办法，下面讲讲具体的配置

1.服务器可以与微信通信
这个在微信公众平台-->开发-->基本配置-->服务器配置中设置，你需要在服务器端写好对应的响应接口，当你点击启用时，微信服务器会请求配置的url，你需要按微信的要求返回相应的数据，如果你返回了正确的数据，就会提示认证成功，此时服务器可以与微信服务器进行通信
![](http://obfs4iize.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%94%AF%E4%BB%981.png)

2.配置网页授权域名和JS接口安全域名
网页授权域名：只有授权域名下的网页才能通过微信正常访问，不会出现，非微信官方网页，要用户点击继续访问的界面，一般做微信公众号开发都要将开发的网站的域名进行授权，**这里填写网站根域名则可，如www.lmwen.top**

![](http://obfs4iize.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%94%AF%E4%BB%982)

JS接口安全域名：只有JS接口安全域名才能调用微信提供的js，简单说就是只有这个域名下才能调用jssdk，当然要正常使用jssdk处理配置JS接口安全域名外还需要jssdk，**这里填写调用jssdk网页的全域名，如www.lmwen.top/usejssdk.com/**，如果前端是单页面开发，那么同样填写根域名则可

3.设置IP白名单
不是说通过了步骤1后，我们的服务器已经可以跟微信服务器通信了吗? 为什么要配置ip白名单呢？步骤1只是让你可以与微信服务器通信，但是要获得微信的access_token就需要配置ip白名单了，access_token是微信公众号开发的一个重要概念，需要搞清楚，而且微信不止一个access_token

4.配置支付授权目录
这个目录是JSAPI支付授权目录，也就是说，要使用JSSDK调用微信支付必须在授权目录下，这里需要注意的是，**必须配置成最后一层，怎么说？比如你的支付目录是：www.lmwen.top/ayuliao/weipay/weipay.html，那么最后一层就是www.lmwen.top/ayuliao/weipay/**

## 微信支付代码开发
确保配置没问题了，就开始开发
先上一张我调用微信支付成功的图，让大家嗨一嗨

![](http://obfs4iize.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%94%AF%E4%BB%983.png)

下面就一步步来把坑踩了

### JSSDK授权
首先要看一遍微信的官方文档 [微信JS-SDK说明文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115)

这里关注到步骤三，通过config接口注入权限验证配置，需要的信息如下：

```
wx.config({
    debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
    appId: '', // 必填，公众号的唯一标识
    timestamp: , // 必填，生成签名的时间戳
    nonceStr: '', // 必填，生成签名的随机串
    signature: '',// 必填，签名，见附录1
    jsApiList: [] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
});
```

除了debug和jsApiList外，其他信息都需要后端返回，那么前端要做的就是通过Ajax请求后端提供的接口，向后端索要appId,timestamp,nonceStr,signature等信息

具体怎么做，其实微信JS-SDK说明文档中是有提的，但是坑爹的将它放到附录1，这么重要的东西你放附录，就是你爱用不用的态度，只能服气，具体的步骤如下：

1.获得access_token，如何获取access_token看这里[获取access_token](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140183)

简单讲，就是通过你公众号的appid和secret向 https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET 发送GET请求，然后微信就会返回JSON类型的数据

```
{"access_token":"ACCESS_TOKEN","expires_in":7200}
```

access_token只会保存2小时且请求次数有限制，所以一般你需要将它保存到数据库中，每2小时请求一次该接口更新一下access_token，我一般1个半小时就更新一次，保证access_token是可以用的

具体代码如：

```
def getAccessToken():
    '''
    获得微信公众号的Access Token
    每1个半小时调用一次，获得最新的Access Token
    '''
    url = "https://api.weixin.qq.com/cgi-bin/token"
    payload = {'grant_type': 'client_credential', 'appid': APPID,'secret':APPSecret}
    r = requests.get(url, params=payload)
    if r.status_code == requests.codes.ok:
        return json.loads(r.text).get('access_token','')
    return ''
```

我在系统中开启了定时任务，每一个半小时就会调用一次getAccessToken()方法，并且将获得的最新的access_token存入mysql数据库总

2.通过access_token来获得jsapi_ticket
jsapi_ticket是公众号用于调用微信JS接口的临时票据，有效期同样是2个小时，因为jsapi_ticket调用次数也是有显示的，所以同样要将其存放到数据中，每2小时更新一次

其实就是通过access_token以GET方法请求 https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=ACCESS_TOKEN&type=jsapi

成功会返回JSON格式数据，其中的ticket就是我们需要的jsapi_ticket：

```
{
"errcode":0,
"errmsg":"ok",
"ticket":"bxLdikRXVbTPdHSM05e5u5sUoXNKd8-41ZO3MhKoyN5OfkWITDGgnr2fwJ0m9E8NYzWKVZvdVtaUgWvsdshFKA",
"expires_in":7200
}
```

看回到JSSDK授权的信息，似乎既没有要access_token也没有要jsapi_ticket，其实获得这两个值是为了通过微信规定的签名算法计算出jssdk授权时需要的signature

3.获得appId、timestamp、nonceStr和signature
先看下官方规定的签名规则：

>签名生成规则如下：参与签名的字段包括noncestr（随机字符串）, 有效的jsapi_ticket,timestamp（时间戳）, url（当前网页的URL，不包含#及其后面部分） 。对所有待签名参数按照字段名的ASCII 码从小到大排序（字典序）后，使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串string1。这里需要注意的是所有参数名均为小写字符。对string1作**sha1加密**，字段名和字段值都采用原始值，不进行URL 转义。

为了快速开发，可以去网上找一下第三方的开源库，这里使用的是weixin-python，可以直接通过pip安装

```
pip install weixin-python
```

weixin-python中已经帮我们完成了微信登录、支付等常用功能的封装，并提供出相应的接口供我们使用，可以从Github上直接将这个项目拉下来

[weixin-python Github地址](https://github.com/zwczou/weixin-python)

你需要看一下文档，主动了解器参数代表的含义

```
WEIXIN_TOKEN 必填，微信主动推送消息的TOKEN
WEIXIN_SENDER 选填，微信发送消息的发送者
WEIXIN_EXPIRES_IN 选填，微信推送消息的有效时间
WEIXIN_MCH_ID 必填，微信商户ID，纯数字
WEIXIN_MCH_KEY 必填，微信商户KEY
WEIXIN_NOTIFY_URL 必填，微信回调地址
WEIXIN_MCH_KEY_FILE 可选，如果需要用退款等需要证书的api，必选
WEIXIN_MCH_CERT_FILE 可选
WEIXIN_APP_ID 必填，微信公众号appid
WEIXIN_APP_SECRET 必填，微信公众号appkey
```

微信支付只需要

```
微信支付

WEIXIN_APP_ID
WEIXIN_MCH_ID
WEIXIN_MCH_KEY
WEIXIN_NOTIFY_URL
WEIXIN_MCH_KEY_FILE
WEIXIN_MCH_CERT_FILE
```

简单看一下其中的源码，其实非常简单，这里分析一下，方便我们后面使用时不会懵逼

weixin-python有个主类

```
class Weixin(WeixinLogin, WeixinPay, WeixinMP, WeixinMsg):
    """
    微信SDK

    :param app 如果非flask，传入字典配置，如果是flask直接传入app实例
    """
    def __init__(self, app=None):
        if app is not None:
            if isinstance(app, dict):
                app = StandaloneApplication(config=app)
            self.init_app(app)
            self.app = app
...
```

它继承自WeixinLogin、WeixinPay、WeixinMP、WeixinMsg，也就是说Weixin这个类具有所有的功能，那么使用微信支付功能时，直接实例化这个类就好了，因为我们关注的是微信支付，所有重点关注一下WeixinPay类中的代码，这个类中帮我们实现了很多方法，使我们可以很方便的生成微信所需要的认证信息，但是，**不要盲目使用**，这些代码不是所有场景都能使用的，这个坑我已经踩了。

因为我们需要将appId、timestamp、nonceStr和signature返回给前端，其实appId、timestamp、nonceStr都比较好活动，appId就是公众号的appid，timestamp就是当前的时间戳，nonceStr则是随机生成的字符串

nonceStr如何获得可以看WeixinPay类中的代码

```
@property
def nonce_str(self):
    char = string.ascii_letters + string.digits
    return "".join(random.choice(char) for _ in range(32))
```

它通过@property将nonce_str方法转变成WeixinPay类中的属性

接着看到WeixinPay类中的sign()签名方法

```
def sign(self, raw):
    raw = [(k, str(raw[k]) if isinstance(raw[k], int) else raw[k])
            for k in sorted(raw.keys())]
    s = "&".join("=".join(kv) for kv in raw if kv[1])
    s += "&key={0}".format(self.mch_key)
    return hashlib.md5(s.encode("utf-8")).hexdigest().upper()
```

其实这个签名方法不满足微信对signature签名规则的要求，该签名方法是针对微信支付签名的，而不是jssdk授权签名，**两者主要的区别在于jssdk授权签名不需要商户支付密码而且加密方式是sha1加密**，所以这里需要自己实现一个针对jssdk授权签名的方法，如下：

```
def signatureSign(self, raw):
    '''
    获得jssdk授信所需要的signature
    '''
    raw = [(k, str(raw[k]) if isinstance(raw[k], int) else raw[k])
               for k in sorted(raw.keys())]
    s = "&".join("=".join(kv) for kv in raw if kv[1])

    signature = hashlib.sha1(s.encode('utf-8')).hexdigest().lower()
    return signature
```   

这样就有了前端jssdk授权所需要的所有数据，整体代码如下

```
class GetJSSDKConfigView(APIView):
    '''
    返回jssdk授权信息，前端只有获得jssdk授权信息后，才能通过微信的接口获得jssdk的使用权限，有了权限才可以调用微信支付
    '''
    from extra_apps.weixin import Weixin
    import time

    weixin = Weixin(
        dict(WEIXIN_APP_ID=微信公众号APPID, WEIXIN_MCH_ID='微信商户ID，纯数字', WEIXIN_MCH_KEY='微信商户KEY',WEIXIN_NOTIFY_URL='回调URL', WEIXIN_MCH_KEY_FILE='', WEIXIN_MCH_CERT_FILE=''))
    timestamp = str(int(time.time()))
    permission_classes = (IsAuthenticated, IsOwnerOrReadOnly)
    authentication_classes = (JSONWebTokenAuthentication, SessionAuthentication)

    def signatureSign(self, raw):
        '''
        获得jssdk授信所需要的signature
        '''
        raw = [(k, str(raw[k]) if isinstance(raw[k], int) else raw[k])
               for k in sorted(raw.keys())]
        s = "&".join("=".join(kv) for kv in raw if kv[1])

        signature = hashlib.sha1(s.encode('utf-8')).hexdigest().lower()
        return signature

    def get(self, request, *args, **kwargs):
        if self.request.user:
            data = request._request.GET
            from urllib import parse
            URL = data.get('URL', '')
            URL = parse.unquote(URL)
            try:
                k = WeiXinAccessTokenModel.objects.filter(id=1)[0]
                # 前一小时
                if k.change_time >= datetime.now() - timedelta(hours=2):
                    accesstoken = k.accesstoken
                else:
                    accesstoken = getAccessToken()
            except:
                accesstoken = getAccessToken()

            # 有效jsapi_ticket
            ticket = getJsapi_ticket(accesstoken)

            # 随机字符串
            nonce_str = self.weixin.nonce_str
            signature = self.signatureSign(dict(noncestr=nonce_str,jsapi_ticket=ticket,timestamp=self.timestamp,url=URL))
            raw = dict(appId=APPID,timestamp=self.timestamp,nonceStr=nonce_str,signature=signature)
            return Response(raw, status=status.HTTP_201_CREATED)
        else:
            return Response({'error':'用户未登录'}, status=status.HTTP_400_BAD_REQUEST)
```

这里是典型的Django REST Framework的写法，重写了它的get方法，access_token从数据库中取，jsapi_ticket有点偷懒，直接获取了，其他的都是前面讲了的逻辑


前端获得了这些内容后，将它和需要申请的权限传给微信服务器，如果第一大步的配置都做了，就可以获得jssdk授权了

## JSSDK调用微信支付
JSSDK有了权限后就可以调用微信支付了，调用的方法也在微信JS-SDK说明文档中

```
wx.chooseWXPay({
timestamp: 0, // 支付签名时间戳，注意微信jssdk中的所有使用timestamp字段均为小写。但最新版的支付后台生成签名使用的timeStamp字段名需大写其中的S字符
nonceStr: '', // 支付签名随机串，不长于 32 位
package: '', // 统一支付接口返回的prepay_id参数值，提交格式如：prepay_id=\*\*\*）
signType: '', // 签名方式，默认为'SHA1'，使用新版支付需传入'MD5'
paySign: '', // 支付签名
success: function (res) {
// 支付成功后的回调函数
}
});
```

从这里可以看出前端需要timestamp、nonceStr、package、singType和paySign

其中package和paySign的获得需要写点代码，但是使用lweixin-python库后，一行代码就可以获得这些信息了，如下

```
jsdict = self.weixin.jsapi(out_trade_no=order, body='微信支付测试', total_fee=1, trade_type='JSAPI',spbill_create_ip=ip,openid=openid)
```

你需要做的不过是传入你系统的订单号、用户端的IP和用户的openid

这里我遇到的一个坑就是所谓的用户端IP，如果你是微信公众号支付，这里的IP就要获得用户手机的IP，这个IP是用户的IP不是你服务器的IP，而openid则是该用户对应你微信公众号唯一的ID，这个有多种方式可以获得，这里我们使用了静默授权的方式，后面有机会会讲讲所谓的静默授权，这里就不展示相关的代码

那么将这些内容传递进去就可以了，其实并不玄妙，看都jsapi()方法的代码

```
    def jsapi(self, **kwargs):
        """
        生成给JavaScript调用的数据
        详细规则参考 https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_7&index=6
        """
        kwargs.setdefault("trade_type", "JSAPI")
        raw = self.unified_order(**kwargs)
        package = "prepay_id={0}".format(raw["prepay_id"])
        timestamp = str(int(time.time()))
        nonce_str = self.nonce_str
        raw = dict(appId=self.app_id, timeStamp=timestamp,
                   nonceStr=nonce_str, package=package, signType="MD5")
        sign = self.sign(raw)
        return dict(package=package, appId=self.app_id,
                    timeStamp=timestamp, nonceStr=nonce_str, sign=sign)
```

其中通过unified_order()方法调用了微信的统一下单接口从而获得prepay_id，通过这个就可以组成我们需要的package，然后有调用了sign()方法，对appid、timeStamp、nonceStr、package和signType进行签名，获得我们需要的paySign

sign()方法前面已经看过了，这里来看看unified_order()方法

```
    def unified_order(self, **data):
        """
        统一下单
        out_trade_no、body、total_fee、trade_type必填
        app_id, mchid, nonce_str自动填写
        spbill_create_ip 在flask框架下可以自动填写, 非flask框架需要主动传入此参数
        """
        url = "https://api.mch.weixin.qq.com/pay/unifiedorder"

        # 必填参数
        if "out_trade_no" not in data:
            raise WeixinPayError("缺少统一支付接口必填参数out_trade_no")
        if "body" not in data:
            raise WeixinPayError("缺少统一支付接口必填参数body")
        if "total_fee" not in data:
            raise WeixinPayError("缺少统一支付接口必填参数total_fee")
        if "trade_type" not in data:
            raise WeixinPayError("缺少统一支付接口必填参数trade_type")

        # 关联参数
        if data["trade_type"] == "JSAPI" and "openid" not in data:
            raise WeixinPayError("trade_type为JSAPI时，openid为必填参数")
        if data["trade_type"] == "NATIVE" and "product_id" not in data:
            raise WeixinPayError("trade_type为NATIVE时，product_id为必填参数")
        data.setdefault("notify_url", self.notify_url)
        data.setdefault("spbill_create_ip", self.remote_addr)

        raw = self._fetch(url, data)
        return raw       
```

看到该方法，其实就是简单将所需要的数据放到data方法中，如果没有需要的数据就给你返回个错误

最终调用_fetch()方法，该方法才是真正去请求微信统一支付接口的方法，看到这个方法我的疑惑就解开了，因为微信支付文档中对于调用统一支付接口需要很多参数，但是我调用jsapi()方法时只穿了少数几个，其他的参数呢？原来直接通过Weixin这个类的实例来获得

```
    def _fetch(self, url, data, use_cert=False):
        data.setdefault("appid", self.app_id)
        data.setdefault("mch_id", self.mch_id)
        data.setdefault("nonce_str", self.nonce_str)
        data.setdefault("sign", self.sign(data))

        if use_cert:
            resp = self.sess.post(url, data=self.to_xml(data), cert=(self.cert, self.key))
        else:
            resp = self.sess.post(url, data=self.to_xml(data))
        content = resp.content.decode("utf-8")
        if "return_code" in content:
            data = Map(self.to_dict(content))
            if data.return_code == FAIL:
                raise WeixinPayError(data.return_msg)
            if "result_code" in content and data.result_code == FAIL:
                raise WeixinPayError(data.err_code_des)
            return data
        return content
```

那么到这里相信你也通透了，看一下完整代码

```python
class WeiPayView(APIView):
    permission_classes = (IsAuthenticated, IsOwnerOrReadOnly)
    authentication_classes = (JSONWebTokenAuthentication, SessionAuthentication)


    from extra_apps.weixin import Weixin, WeixinError
    from wk.settings import APPID

    weixin = Weixin(
        dict(WEIXIN_APP_ID=微信公众号APPID, WEIXIN_MCH_ID='微信商户ID，纯数字', WEIXIN_MCH_KEY='微信商户KEY',WEIXIN_NOTIFY_URL='回调URL', WEIXIN_MCH_KEY_FILE='', WEIXIN_MCH_CERT_FILE=''))

    def get(self, request, *args, **kwargs):
        '''
        调用微信支付，返回前端数据，让前端使用jssdk来使用微信支付
        前端传递订单号，通过订单号调用微信接口
        '''
        data = request._request.GET
        # 订单号
        order = str(data.get('order', ''))

        # order_sn = OrderInfo.objects.all()[0].order_sn
        openid = self.request.user.openid
        ip = request._request.META['REMOTE_ADDR']
        # 需要给前端返回的内容，out_trade_no订单号，total_fee是金额*100，trade_type调用微信支付的方式，spbill_create_ip用户端的IP，需要配置
        jsdict = self.weixin.jsapi(out_trade_no=order, body='微信支付测试', total_fee=1, trade_type='JSAPI',spbill_create_ip=ip,openid=openid)
        return Response(jsdict, status=status.HTTP_201_CREATED)
```

需要提一下，实例化Weixin类时传入的WEIXIN_NOTIFY_URL，除了可以是URL外，还可以是IP，这个IP后可以加上相应的端口，比如8.8.8.8:9000/callback，这个路径用于接受微信服务器的回调，无用支付成功或失败

前端获取这些信息后，按微信提供的格式传递给微信服务器就可以调用微信支付了，这里需要注意，**前端传值的字段名要严格按照微信提供的来，比如微信要appId，你不能穿appid，微信要nonceStr，你不能noncestr**，被前端坑了一下午，一直以为配置错了，当时咋没将自己99级的杀猪大刀带来上班呢？


## 微信回调
虽然微信支付调用成功了，但是还有个很多的逻辑就是微信回调，前端的回调比较简单，就是跳转一下页面，提醒一下用户，这里说一个坑，因为我们前端使用的是AngelarJS，一般AngelarJS通过路由的方式来跳转界面，但是微信回调会带上一些信息，此时通过AngelarJS的路由进行页面跳转的话，很可能就会造成页面混乱的问题，解决方法也很多简单，就会通过原生的js跳转就好了

微信回调的主要逻辑其实在后端，比如用户购买成功了，微信系统就会通过你配置的回调接口将用户支付的行为返回，那么这里其实涉及到了一个安全问题，微信文档中是没有提回调的，网上一查，如何验证微信的回调信息？有人直接通过返回来的nonceStr与此前传过去的nonceStr做对比，如果相同，那么就认为没问题

虽然nonceStr是随机生成的，但是根本不能保证安全性，如果你无法保证是微信返回的消息，那么很容易就被hack攻击，攻击过程也很简单，使用你的产品，设置代理进行抓包，抓到你发送给微信的信息和微信返回给你的信息，此时hack就知道微信返回的数据格式了，下次购买时，同样抓包，获得你发送给微信的信息，从中获取此次发送给微信的nonceStr，构造虚假的微信返回数据格式，将nonceStr放到虚假的格式中，通过发包软件发送给你的系统，那么你的系统就会判断用户支付成功，实际上hack一分钱没付，如果你是商城，那么分分钟被人撸光所有的货品

那要怎么做才能保证安全呢？很简单，就是对数据进行再次签名，签名中使用了微信商户的KEY，这个除了自己和微信系统其它人都是不知道的，通过微信商户KEY对微信返回回调接口的数据进行签名后，在于回调数据中的前面数据进行对比，如果相同，那么可以认为数据没有被修改，可以正常使用，当然这不能保证数据是微信服务器发送过来的，因为第三者可以保留这份数据，再对你进行发送，当然这份数据第三者不能进行修改，所以对我们系统来说，其实没有什么差，这个过程可以使用WeixinPay类中的check()方法，帮我们完成了验证微信回传数据是否可靠的过程

```
from django.http.response import HttpResponse, HttpResponseBadRequest
from django.views.decorators.csrf import csrf_exempt


@csrf_exempt
def WeiPayCallBackView(request):
    '''
    因为Django Rest Framework不支持text/xml形式的xml，要么修改drf的源代码，要么通过原始的Django方法来响应
    '''
    if request.method == 'POST':
        from extra_apps.weixin import Weixin, WeixinError
        from wk.settings import APPID
        weixin = Weixin(
        dict(WEIXIN_APP_ID=微信公众号APPID, WEIXIN_MCH_ID='微信商户ID，纯数字', WEIXIN_MCH_KEY='微信商户KEY',WEIXIN_NOTIFY_URL='回调URL', WEIXIN_MCH_KEY_FILE='', WEIXIN_MCH_CERT_FILE=''))

        data = weixin.to_dict(request.body)
        order_sn = data.get('out_trade_no','')
        # 支付结果，用户正常支付，会返回SUCCESS
        result_code = data.get('result_code','')
        # check 检查微信回传数据是否可靠
        if not weixin.check(data):
            return HttpResponse(weixin.reply("签名验证失败", False))

        try:
            pay_status = OrderInfo.objects.filter(order_sn=order_sn)[0].pay_status
            
            if pay_status == 'paying' and result_code == 'SUCCESS':
                
              '''
              用户支付成功后，你系统要做的具体逻辑
              '''
            return HttpResponse(weixin.reply("OK", True), content_type='text/xml')
        except:
            return HttpResponse(weixin.reply("签名验证失败", False), content_type='text/xml')
```

代码逻辑比较简单，这样我也遇到了几个坑，因为整个项目采用的是前后端分离开发，后端使用的是Django REST Framework，因为微信回传的数据是xml格式的，而且Content-Type是text/xml，Django REST Framework中不支持text/xml这种格式，解决方法有多种，一种是修改Django REST Framework的源代码，不推荐，另一种就直接用原生的Django来写，因为原生的Django默认需要进行csrf_exempt验证，微信时不可能给你返回csrf_exempt信息的，所以要去除Django默认的验证，非常简单，在这个接口上加上@csrf_exempt就可以了

接着要注意的是我们回传给微信系统的数据也要是Content-Type为text/xml的，然后又是一个坑，无论你是否返回数据给微信系统响应这次回调，微信为了保证你能收到回传的数据，都会多次回调你这个接口，如果你在逻辑上不做处理，你的逻辑就会执行多遍，我的做法是先获得系统中对于订单的状态，如果订单是未支付的状态，那么就执行相应的逻辑，如果订单已经完成支付，就直接跳过这些逻辑，返回一个OK信息给微信支付系统

当然，微信无论用户支付成功、支付失败还是取消支付都会通过回调接口返回信息，所以要对微信返回信息的状态做一个判断，如果是SUCCESS，说明用户支付成功了，只有这个使用才执行自己支付成功的逻辑

## 结尾
到这里微信支付才算完整的弄完了，回头一看，感觉还是挺简单的，这里就牵扯出一个有趣的现象，当一个人掌握了一个东西后就会觉得简单，从而忘记了掌握这个东西的过程中的困难，当其他人问你的时候，你就会给它造成认知偏差，说，这东西非常简单，几天就可以精通了，这个在编程圈屡见不鲜。

微信支付坑多吗？不会啊，就配置一下，写点代码就好，这有什么坑的，非常简单，1个小时应该就可以搞定了，哈哈哈。

