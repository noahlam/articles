> 本文不涉及到 AI 的知识,如果你是冲着 AI 来的,那么可能会让你失望了.

前一阵子一个朋友找我,问我能不能搞一个微信自动加好友的软件,(在普通人眼里,程序员就是专门写木马病毒外挂软件的三流黑客.不会写那就连三流都不是.

所以为了证明我是三流黑客,我随便百度了两个现成的给他.本来事情到这里应该结束了的,不过本着探索的精神,想顺便了解一下这种外挂的原理,于是百歌谷度了一下,
最终原理没找到,倒是找到几个有意思的 github 仓库,利用网页版的微信 API 做第三方微信.

先看个效果？

<img src="https://raw.githubusercontent.com/noahlam/wxbot/master/snapshot/WechatIMG1.png" width="300" />
<img src="https://raw.githubusercontent.com/noahlam/wxbot/master/snapshot/WechatIMG2.png" width="300" />


### 步骤

我们看看大致步骤

1. 获取 UUID
1. 根据 UUID 获取二维码
1. 扫码登陆, 获取登陆信息
1. 拿登陆信息换初始化数据
1. 拿数据初始化
1. 获取好友列表和消息列表
1. 发送消息


以下为具体过程,不感兴趣的可以直接拉到末尾查看源码仓库


### 一、获取 UUID

接口地址 https://wx.qq.com/jslogin

参数

```
{
    appid: 'wx782c26e4c19acffb',
    fun: 'new',
    lang: 'zh_CN',
     _: new Date().valueOf()
}
```

除了最后一个当前时间戳不是固定的,其他的3个参数都是写死的,照抄即可,调用成功的话,会到一个字符串 `window.QRLogin.code = 200; window.QRLogin.uuid = "obizONtqZA==";`, 需要自己想办法截取到 `window.QRLogin.uuid = ` 后面的那串字符,即 UUID.


### 二、获取二维码

这一步很简单,有了 UUID 后,我们可以直接请求 `'https://wx.qq.com/qrcode/' + UUID` 获取到二维码. 获取到二维码以后,先别急着去扫描二维码,因为我们要先去监听二维码的扫描状态,这样我们才能知道什么时候被登陆.


### 三、监听二维码的扫描结果

接口地址 https://wx.qq.com/cgi-bin/mmwebwx-bin/login

参数

```
{
    tip: 0,
    uuid: 'obizONtqZA==',
    _: new Date().valueOf(),
    loginicon: true
}
```

tip 取值 0 或 1, 监听分2个阶段,第一阶段,监听用户是否扫码,tip 为 0,第二阶段,监听用户是否在微信上点确认登陆,tip 为 1.

uuid 就是第一步获取到的那个 UUID

_ 当前时间戳

loginicon 我猜应该是否扫码完返回用户头像,都填 true 即可.

返回结果,当你扫描二维码的时候,接口会返回你一个这样的对象

```
{
    'window.code': 201,
    'window.userAvatar': 头像的 base64 地址
}
```

得到的 code 是 201, 说明已扫码,但并不代表已登陆,还需要继续监听是否在手机微信上点击 确认登陆 按钮(重复上面步骤,把 参数里的 tip 改为 1 即可)

这步如果成功的话,会返回一个如下对象
```
{
    'window.code': '200',
    'window.redirect_uri': 'https://wx.qq.com/cgi-bin/mmwebwx-bin/webwxnewloginpage?ticket=ARD37_ikx-Kakd2i0W-f-E7q@qrticket_0&uuid=4f6yOkV4AA==&lang=zh_CN&scan=1548300672' }
}
```

### 四、获取初始化数据(敏感数据)

上一步获取到的数据里面的 `window.redirect_uri` 里包含了一个 url 和一些 查询参数,直接请求这个地址好像没办法成功,需要将 url 和 参数拆分,然后加入其他参数

接口地址 就是上面的 url

参数

```
{
    ticket: 上面得到的 ticket,
    uuid: 上面得到的 uuid,
    lang: 'zh_CN',  // 固定
    scan: 上面得到的 scan,
    fun: 'new' // 固定
}
```

这一步的返回的头部里面,会有个 cookie ,需要存起来,接来来得到请求头里面要带上这个 cookie,另外就是一个 xml 格式的 敏感的信息,也是要存起来.

> tip： xml 格式可以用 [xml2js](https://github.com/Leonidas-from-XIV/node-xml2js ) 转换成 json.


### 五、初始化

呼,到这一步,终于接近登陆成功了,只需再调用以下接口,初始化以下

接口地址 `https://wx.qq.com/cgi-bin/mmwebwx-bin/webwxinit?r=${~(new Date().valueOf())}`

参数

```
{
    BaseRequest: {
        DeviceID: 'e747337466044216', // 这个好像随便填都可以
        Sid: 上一步获取到的 wxsid,
        Uin: 上一步获取到的 wxuin,
        Skey: 上一步获取到的 skey
    }
}
```

这里有 2 个地方跟之前不同的,第一是地址后面要跟一个时间戳,而且这个时间戳还要按位取反,第二个是请求参数是放在 BaseRequest 下面,而不是对象的一级属性下面.

返回的数据里面有 2 个数据需要保存起来,一个是 data.SyncKey, 一个是 res.data.User.UserName,后面都会用到

到此才真正完成登陆,下面如果你不需要好友列表的话,可以直接收取消息了


### 六、检测新消息

接口地址 https://webpush.wx.qq.com/cgi-bin/mmwebwx-bin/synccheck

参数

```
let time = new Date().getTime()

let synckey = ''

let sk = data.SyncKey.List || []   // data.SyncKey 就是上一步获取到的那个

for (let i = 0; i < sk.length; i++) {
    synckey += `${sk[i].Key}_${sk[i].Val}`
    if (i !== sk.length - 1) synckey += '|'
}

// 传递的参数
{
    r: time,
    sid: 第四步拿到的 wxsid,
    uin: 第四步拿到的 wxuin,
    skey: 第四步拿到的 skey,
    deviceid: 'e747337466044216', // 同上一步
    synckey: synckey,
    _: time
}
```

返回内容的 data 里面 包含如下内容

```
window.synccheck={retcode:"0",selector:"2"}
```

如果 selector 是 2, 说明有新消息,走下一步,获取消息内容


### 七、获取消息内容

接口地址 https://wx.qq.com/cgi-bin/mmwebwx-bin/webwxsync

参数

```
{
    BaseRequest: {
        Uin: 第四步拿到的 wxuin,
        Sid: 第四步拿到的 wxsid,
        Skey: 第四步拿到的 skey,
        DeviceID: 'e747337466044216', // 同上一步
    },
    SyncKey: data.SyncKey, // 还记得上一步我们费尽千辛万苦转换这个数据吗？ 你没看错,这里不需要转换,就是这么神奇
    rr: ~(new Date().valueOf())
}
```

返回结果里面有个 `data.AddMsgList` 就是消息列表了,还有个 `data.SyncCheckKey` 就是下次请求的时候用的 `SyncKey`, 每次都会变的.

AddMsgList 是一个数组,里面可能包含多条消息,消息的自动比较多,就不一一说明了,这里说说 2 个比较重要的字段,其他的字段有兴趣的可以自己打印出来看一下.

FromUserName 对方的微信名,说是微信名,其实是一个 @ 或 @@ 开头的内部的id, 完全不可读,据我猜测 @ 开头的应该是普通好友, @@ 开头的是群或者公众号之类的

Content 消息内容

有了消息内容,和发消息的人,我们就可以回复对方,不过回复什么？ 当然不可能写一大堆 if else 或者 switch case 去适应各种情况,不妨网上搜索一下 `价值一个亿的ai代码` 哈哈哈


### 八、获取自动回复内容

这边我用的是图灵机器人的 API [地址](http://www.tuling123.com/),当然你也可以用其他的.

接口地址 http://openapi.tuling123.com/openapi/api/v2

参数

{
    perception: {
        inputText: {
            text: '待回复的消息'
        }
    },
    userInfo: {
        apiKey: tulingApiKey,  // 在图灵官网申请
        userId: tulingUserId   // 同上
}

你要是懒得去申请的话,可以在我的项目里面复制, 在 `src/global.js` 里面,在返回的内容里面 `data.results[0].values.text` 下面可以看到图灵给你生成的自动回复内容(results是一个数组,支持一次回复多条)


### 九、回复消息

拿到自动回复以后,我们只需要把它发给你的好友,即完成一次自动对话.

接口地址 https://wx.qq.com/cgi-bin/mmwebwx-bin/webwxsendmsg

参数

```
let timeStamp = new Date().getTime() + '' + (9000 * Math.random() + 1000)
{
    BaseRequest: {
      Uin: 同上,
      Sid: 同上,
      Skey: 同上,
      DeviceID: 同上
    },
    Msg: {
      Type: 1,  // 消息类型 1 是文字消息,其他的暂时没用过
      Content: '回复的内容',
      FromUserName: '你的用户名,在第五步有拿到',
      ToUserName: '对方的微信名 第七步的 FromUserName',
      LocalID: timeStamp,
      ClientMsgId: timeStamp
}
```

发送成功的话,会返回如下内容

```
{
    BaseResponse: { Ret: 0, ErrMsg: '' },
    MsgID: '2033517278669301361',
    LocalID: ''
}
```

好了,这样我们的一个自动回复机器人就完成了.完整的代码在[这里](https://github.com/noahlam/wxbot)


### 广告时间

我们40人的前端团队常年招兵买马中,在厦门的和想来厦门的童鞋们,不要吝惜你的简历,使劲砸过来 邮箱：`atob('bnVveWFAZ2FvZGluZy5jb20=')`, 期待你一起来`稿`事


对本文有意见或者建议,请尽量在 [github](https://github.com/noahlam/wxbot) 上提 [issue](https://github.com/noahlam/wxbot/issues), 最近比较忙,比较不怎么逛社区
