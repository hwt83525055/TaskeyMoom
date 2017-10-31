---
title: 微博脚本
date: 2017-10-15 21:53:06
tags: Python
---

<font color=grey>恩，慢慢更新，先挖个坑
写这个的主要原因是自己的微博被许多卖片的小姐姐们关注了，很恶心，准备一次清理掉

>## 账号密码
首先还是老样子，分析一下网站对账号密码的加密操作

老规矩，能从移动端入手就不从PC端入手，但是这次直接从PC入手，[手机版微博](weibo.com)

当输入完账号按下tab切到密码框时，我们会捕捉到一个get请求，打开看看

https://login.sina.com.cn/sso/prelogin.php?checkpin=1&entry=mweibo&su=MTU2MDYxMzAwMDk=&callback=jsonpcallback1508381565848

![Aaron Swartz](https://raw.githubusercontent.com/hwt83525055/PhotoSource/master/weibocrawl1.jpeg)

	
	0
servertime
	
	1508381509
pcid
	
	"tc-f85808f34da5621cedf8bd9170eae0006ce2"
nonce
	
	"BR74OK"
pubkey
	
	"-----BEGIN PUBLIC KEY-----MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDrKjhWhmGIf6GAvdtcq9XyHHv9WcCQyy0kWoesJTBiiCcpKT5VBjUFCOf5qju3f0MzIxSQ+RX21jxV/i8IpJs1P0RK05k8rMAtt4Sru45CqbG7//s4vhjXjoeg5Bubj3OpKO4MzuH2c5iEuXd+T+noihu+SVknrEp5mzGB1kQkQwIDAQAB-----END PUBLIC KEY-----"
rsakv
	
	"1330428213"
is_openlock
	
	0
lm
	
	1
smsurl
	
	"https://login.sina.com.cn/sso/msglogin?entry=mweibo&mobile=15606130009&s=2ae11f76e8865a2a07492ed2dbab69f6"
showpin
	
	0
exectime
	
	9
>这么看好像也挺清晰的，就是占空间了点-，-
挨个分析一下，retcode毫无疑问是返回的状态值，servertime顾名思义是时间，对这个敏感的同学应该发现这是一个去除小数点的时间戳，即time.time()*1000

中间几个后面会用到，这边先不说，showpin是对验证码是否弹出的一个设定，前两天测试的时候总是会弹出一个验证码图片，即showpin=0，今天测试的时候不知道为什么这个值变成了0，即没有验证码，不清楚触发的条件是什么

再对域名传过去的值进行分析

checkpin=1&entry=mweibo&su=MTU2MDYxMzAwMDk=&callback=jsonpcallback1508381565848

checkpin应该是检验是否请求验证码的值（猜测），entry入口是手机版微博，callback即我们上面截图里的json串，下面就是这个su了

su是什么值呢，根据我们申请的时间分析，毫无疑问这就是我们的账号进行了加密之后的显示

在js中寻找一下给账号加密的函数（用mac的同学可以尝试一下Charles，很不错的软件，LInux下不清楚怎么用，所以直接firebug慢慢找了）

我们在js中找到了这么一句
```js
opt.data.su = utf8_to_b64(trim(that.loginName.value));
```
OK,这就是对我们用户名的加密的过程了，trim是js包中的一个函数，寻找一下他的定义
```js
function trim(str) {
        return (!str) ? '' : str.toString().replace(/^\s+|\s+$/g, '');
    }
```
utf8_to_b64即是base64的加密方法

总结一下我们可以获得
```python
 su = base64.b64encode(urllib.parse.quote_plus(user_name).encode('utf-8')).decode('utf-8')
```

那么第一个函数可以着手写了

```python
def get_username(self):
        """
        get legal username
        """
        username_quote = urllib.parse.quote_plus(self.user_name)
        username_base64 = base64.b64encode(username_quote.encode("utf-8"))
        return username_base64.decode("utf-8")
```
那么用户名获取写完了后面毫无疑问要写密码的加密了

我们先尝试进行一次登录，看一下到底一次登录post了哪些数据
其他就不看了，直接看到了sp，即加密之后的密码，我们拎出来看一下

b223228fbe8bd7510d338871081ca1e01b346e9ea0c47dff16f7a72432ebac05f3064c65e07d97d1a48d747c4db3b2b0d94e7b109db80f1e9e430a4c2780f8b3e3a721723a30db41c951c03af6ea04d35c6f48e597fbf32e770fe574af1941bf6d3484443d3c83d2114f83d3b649c01d25d57b779efa031169ac880b5ac3e694

因为看到了rsakv这个列，所以我们几乎可以确定，加密过程中必然有rsa的参与，让我们看看js里能否找到加密的过程

```js
if(me.service){request.service=me.service}if((me.loginType&rsa)&&me.servertime
&&sinaSSOEncoder&&sinaSSOEncoder.RSAKey){request.servertime=me.servertime;request.nonce=me.nonce;request
.pwencode="rsa2";request.rsakv=me.rsakv;var RSAKey=new sinaSSOEncoder.RSAKey();RSAKey.setPublic(me.rsaPubkey
,"10001");password=RSAKey.encrypt([me.servertime,me.nonce].join("\t")+"\n"+password)}else{if((me.loginType
&wsse)&&me.servertime&&sinaSSOEncoder&&sinaSSOEncoder.hex_sha1){request.servertime=me.servertime;request
.nonce=me.nonce;request.pwencode="wsse";password=sinaSSOEncoder.hex_sha1(""+sinaSSOEncoder.hex_sha1(sinaSSOEncoder
.hex_sha1(password))+me.servertime+me.nonce)}}request.sp=password;
```
具体在里面就可以找到实现的过程了，那么代码实现我全部丢到github上了，下面拍个链接 

[Github-微博脚本](https://github.com/hwt83525055/weibofans)