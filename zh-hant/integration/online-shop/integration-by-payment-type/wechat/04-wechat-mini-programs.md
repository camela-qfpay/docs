import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# 微信小程序支付
<Link href="https://sdk.qfapi.com/images/wechat_mp_process.jpg" target="_blank">![WeChat MiniProgram process-flow](@site/static/img/wechat_mp_process.jpg)</Link>

### HTTP请求

`POST ../trade/v1/payment` `PayType: 800213`

**Step 1:** 微信实名认证
业务人员必须在微信官方平台进行身份验证后才能使用微信支付功能。

**Step 2:** 获取 openid
完成实名认证后，通过商户实名小程序获取openid参数。 具体获取方法详见 [微信文档](https://developers.weixin.qq.com/miniprogram/dev/api-backend/open-api/login/auth.code2Session.html).

**Step 3:** 发送付款请求
使用以下参数发起付款请求。

商户可选择开通微信实名认证。 目前实名认证仅适用于中国大陆公民，包括真实姓名和身份证号码。 如果提供身份证明，付款人的钱包信息（例如连接的银行卡）必须与商家提供的数据相同。 如果客户尚未将微信账户绑定银行卡，仍可进行付款。

```plaintext
有关代码说明，请选择带有以下选项卡的 Node.js。
```

```javascript
qfPayOpenAPI: function () {
    let app_code = 'A2BE4E015A8A4B0A8E9D88**********';
    let client_key = '498717301B0846D1992B6F**********';
    let environment = 'https://test-openapi-hk.qfapi.com/trade/v1/payment';
    let openid = this.data.openid;
    let amount = this.data.amount * 100;
    let random_number = String(Math.round(Math.random() * 1000000000));
    let datetime = new Date().toISOString().replace(/T/, ' ').replace(/\..+/, '');

    let payload = {
      txamt: 100,
      txcurrcd: 'SGD',
      pay_type: '800213',
      out_trade_no: '0123456789',
      txdtm: '2020-07-03 03:14:29',
      sub_openid: 'oS80_5dxekECAOlVBeQFk34q123s'
    };

    var ordered = {};
    Object.keys(payload).sort().forEach(function(key) {
      ordered[key] = payload[key] });
    console.log(ordered)

    var str = [];
    for (var p in ordered)
    if (ordered.hasOwnProperty(p)) {
    str.push((p) + "=" + (ordered[p]));
    }
    var string = str.join("&")+client_key;
    console.log(string)

    var signature = utilMd5.hexMD5(string).toUpperCase()
    console.log(signature)

    wx.request({
      url: environment,
      data: payload,
      method: 'POST',
      header: {
        'X-QF-APPCODE': app_code,
        'X-QF-SIGN': signature,
        'content-Type': 'application/x-www-form-urlencoded'
      },
      success: (res) => {
        if (res.statusCode == 200) {
          console.log(res)
          console.log(res.data)
          this.weChatPayment(res);
        }
      },
      fail: (err) => {
        console.log(err);
      },
      complete: (res) => {
        wx.hideLoading();
        console.log("API request completed")
      }
    })
   },
```

### 请求参数

|参数名称| 参数编码| 是否必填| 参数类型|描述|
|:----    |:---|:----- |-----   |----   |
|公共支付参数 | — | — |— |请参阅有关交易的[公共支付参数](/docs/preparation/paycode#支付API端点)|
|微信授权码 |`sub_openid`|是 |String(128)   |   |
订单到期时间 | `expired_time` | 否 | String(3)  | QRC 过期时间（以分钟为单位）。 微信小程序默认QRC过期时间为30分钟。 该参数可手动调整，最小为 5 分钟，最大为 120 分钟。
|Designated payment method   |`limit_pay`| 否 |String    |参数值指定为“no_credit”，禁止信用卡支付。 此设置仅对中国大陆有效。  |
|Extended Customer Info | `extend_info` | 否 | Object | 实名客户身份识别。 该参数目前仅适用于中国大陆公民，并且需要针对所选的[PayType](/docs/preparation/paycode#支付类型)使用微信显式激活。 参数“user_creid”中包含消费者的**身份证号码**，“user_truename”中必须提供编码形式或汉字书写的付款人**真实姓名**。 一个例子如下所示； extend_info = '\{"user_creid":"430067798868676871","user_truename":"\\\u5c0f\\\u6797"\}'|

### 响应参数

|参数编码| 二级参数编码| 参数类型| 参数名称|描述|
|:----    |:---|:----- |-----   |----   |
|`pay_params`    |`appId` |String(16) |公共WMP ID   |开发者在微信注册小程序后，即可获取appId。  |
|—   |`timeStamp` |String(32) | 时间戳    |当前时间  |
|—   |`nonceStr`  |String(32) |随机字符串 |随机字符串，长度不超过32位  |
|—   |`package`   |String(128)|订单详细信息扩展字符串   |统一接口返回的prepay_id参数值的格式为prepay_id=**  |
|—    |`signType` |String(32) |签名方式  |签名类型， 默认：MD5  |
|—    |`paySign`  |String(64) |签名  |  |
|公共响应参数  |—  |— |—  | 请参阅有关交易的[公共支付参数](/docs/preparation/paycode#支付API端点) |
|`txcurrcd`    |  | |货币   | 交易货币。 查看[货币](/docs/preparation/paycode#支付币种) 表以获取可用货币的完整列表 |

**Step 4:** 唤起支付模块

```plaintext
有关代码说明，请选择带有以下选项卡的 Node.js。
```

```javascript
weChatPayment: function(res) {
    wx.requestPayment(
    {
    'timeStamp': '1593746074',
    'nonceStr': '69d8a67fe34e44ca9bb2a20dd299cc58',
    'package': 'prepay_id=wx03111434674853b80d25e0911996417600',
    'signType': 'MD5',
    'paySign': 'B0AECE676746F2A310CB06F27641E809',
    'success': function(res){},
    'fail': function(res){},
    'complete': function(res){}
    })
},
```

获取“pay_params”参数，然后提供相应的付款详细信息。 欲了解更多详情，请参阅
[微信文档](https://pay.weixin.qq.com/wiki/doc/api/wxa/wxa_api.php?chapter=7_7&index=5).

## 微信小程序样板

要快速开始使用，请下载 [QFPay 微信小程序样板](@site/static/files/qfpay_mini_program_payments_boilerplate.zip) 并获得 MD5 哈希算法的访问权限。

<Link href="/img/miniprogram_boilerplate.png" target="_blank">![WeChat Mini Program Boilerplate](@site/static/img/miniprogram_boilerplate.png)</Link>

<br/>
**设置说明**

1) 注册 QFPay，我们会将您的微信 appid 绑定到您的 API 凭证。 <br/>
2) 访问微信 MP 门户 [https://mp.weixin.qq.com](https://mp.weixin.qq.com) 并将我们的环境列入传入服务器流量的白名单：<br/>
开发 -> 开发设置 -> 服务器域名 -> request合法域名: e.g. https://test-openapi-hk.qfapi.com <br/>
3) 将 zip 文件中的文件复制并粘贴到本地硬盘并设置云函数环境。 <br/>
4) 使用云函数“getUserOpenID”获取用户openid，并根据代码运行API调用。 <br/>
