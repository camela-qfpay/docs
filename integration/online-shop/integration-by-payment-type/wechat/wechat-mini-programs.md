---
title: "WeChat Mini Programs"
---

# WeChat Mini Programs

<Link href="https://sdk.qfapi.com/images/wechat_mp_process.jpg" target="_blank">
  ![WeChat MiniProgram process-flow](@site/static/img/wechat_mp_process.jpg)
</Link>

## HTTP Request

**Endpoint** : `/trade/v1/payment`

**Method** : `POST`

**PayType** : `800213`

**Step 1:** WeChat real name authentification Before the payment function within WeChat can be used the business personnel must authenticate themselves on the official WeChat platform.

**Step 2:** Get openid Once real name authentication has been completed, the openid parameter is obtained through the small program of the real name of the merchant. The specific acquisition method is described in the [Wechat documentation](https://developers.weixin.qq.com/miniprogram/dev/api-backend/open-api/login/auth.code2Session.html).

**Step 3:** Send payment request Initiate a payment request with the parameters below.

Optionally merchants can activate real-name authentication with WeChat. Currently real-name identification is only available for Mainland Chinese citizens and include a person's real name and national ID card number. In case identification is provided the payer's wallet information like a connected bank card must be identical with the data provided by merchants. If customers did not yet bind their WeChat account to a bank card the payment will go through regardless.

```plaintext
For code instructions select Node.js with the tabs above.
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

## Request Parameters

<table style="min-width: 100px">
<colgroup><col style="min-width: 25px"><col style="min-width: 25px"><col style="min-width: 25px"><col style="min-width: 25px"></colgroup><tbody><tr><th colspan="1" rowspan="1"><p>Attribute</p></th><th colspan="1" rowspan="1"><p>Mandatory</p></th><th colspan="1" rowspan="1"><p>Type</p></th><th colspan="1" rowspan="1"><p>Description</p></th></tr><tr><td colspan="1" rowspan="1"><p>Public request parameters</p></td><td colspan="1" rowspan="1"><p>—</p></td><td colspan="1" rowspan="1"><p>—</p></td><td colspan="1" rowspan="1"><p>Please refer to the <a target="_blank" rel="noopener noreferrer nofollow" href="/docs/preparation/paycode#public-payment-parameters">Public Payment Section</a> for more details</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">sub_openid</code></p></td><td colspan="1" rowspan="1"><p>Yes</p></td><td colspan="1" rowspan="1"><p>String(128)</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">expired_time</code></p></td><td colspan="1" rowspan="1"><p>No</p></td><td colspan="1" rowspan="1"><p>String(3)</p></td><td colspan="1" rowspan="1"><p>QRC expiration time in unit minutes. The default QRC expiration time for WeChat Mini Programs is 30 minutes. The parameter can manually be adjusted to a minimum of 5 minutes, and up to a maximum of 120 minutes.</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">limit_pay</code></p></td><td colspan="1" rowspan="1"><p>No</p></td><td colspan="1" rowspan="1"><p>String</p></td><td colspan="1" rowspan="1"><p>The parameter value is specified as <code spellcheck="false">no_credit</code>, and credit card payment is prohibited. This setting is only valid for mainland China.</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">extend_info</code></p></td><td colspan="1" rowspan="1"><p>No</p></td><td colspan="1" rowspan="1"><p>Object</p></td><td colspan="1" rowspan="1"><p>Real name customer identification. This parameter is currently only available for Mainland Chinese citizens and needs to be explicitly activated with WeChat for the selected <a target="_blank" rel="noopener noreferrer nofollow" href="/docs/preparation/paycode#payment-codes">PayType</a>. The consumer's <strong>national ID card number</strong> is contained in the parameter <code spellcheck="false">user_creid</code> and the payer's <strong>real name</strong> in encoded form or written in Chinese characters must be provided in <code spellcheck="false">user_truename</code>. An example looks like this; extend_info = '{"user_creid":"430067798868676871","user_truename":"\\u5c0f\\u6797"}'</p></td></tr></tbody>
</table>

## Response Parameters

<table style="min-width: 100px">
<colgroup><col style="min-width: 25px"><col style="min-width: 25px"><col style="min-width: 25px"><col style="min-width: 25px"></colgroup><tbody><tr><th colspan="1" rowspan="1"><p>Attribute</p></th><th colspan="1" rowspan="1"><p>Secondary Attribute</p></th><th colspan="1" rowspan="1"><p>Type</p></th><th colspan="1" rowspan="1"><p>Description</p></th></tr><tr><td colspan="1" rowspan="1"><p>Public response parameters</p></td><td colspan="1" rowspan="1"><p>—</p></td><td colspan="1" rowspan="1"><p>—</p></td><td colspan="1" rowspan="1"><p>Please refer to the <a target="_blank" rel="noopener noreferrer nofollow" href="/docs/preparation/paycode#public-payment-parameters">Public Payment Section</a> for more details</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">pay_params</code></p></td><td colspan="1" rowspan="1"><p><code spellcheck="false">appId</code></p></td><td colspan="1" rowspan="1"><p>String(16)</p></td><td colspan="1" rowspan="1"><p>Public WMP ID, after the developer registers the Mini Program with the WeChat, the appId can be obtained.</p></td></tr><tr><td colspan="1" rowspan="1"><p>—</p></td><td colspan="1" rowspan="1"><p><code spellcheck="false">timeStamp</code></p></td><td colspan="1" rowspan="1"><p>String(32)</p></td><td colspan="1" rowspan="1"><p>Current time</p></td></tr><tr><td colspan="1" rowspan="1"><p>—</p></td><td colspan="1" rowspan="1"><p><code spellcheck="false">nonceStr</code></p></td><td colspan="1" rowspan="1"><p>String(32)</p></td><td colspan="1" rowspan="1"><p>Random string, no longer than 32 bits</p></td></tr><tr><td colspan="1" rowspan="1"><p>—</p></td><td colspan="1" rowspan="1"><p><code spellcheck="false">package</code></p></td><td colspan="1" rowspan="1"><p>String(128)</p></td><td colspan="1" rowspan="1"><p>Order details extension string. The value of the prepay_id parameter returned by the unified interface is in the format of prepay_id=**</p></td></tr><tr><td colspan="1" rowspan="1"><p>—</p></td><td colspan="1" rowspan="1"><p><code spellcheck="false">signType</code></p></td><td colspan="1" rowspan="1"><p>String(32)</p></td><td colspan="1" rowspan="1"><p>Signature type, default is MD5</p></td></tr><tr><td colspan="1" rowspan="1"><p>—</p></td><td colspan="1" rowspan="1"><p><code spellcheck="false">paySign</code></p></td><td colspan="1" rowspan="1"><p>String(64)</p></td><td colspan="1" rowspan="1"><p>Signature</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">txcurrcd</code></p></td><td colspan="1" rowspan="1"><p>Transaction currency. View the <a target="_blank" rel="noopener noreferrer nofollow" href="/docs/preparation/paycode#currencies">Currencies</a> table for a complete list of available currencies</p></td></tr></tbody>
</table>

**Step 4:** Evoke the payment module

```plaintext
For code instructions select Node.js with the tabs above.
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

Obtain the `pay_params` parameter, and then provide payment details accordingly. For more details, please refer to the [Wechat documentation](https://pay.weixin.qq.com/wiki/doc/api/wxa/wxa_api.php?chapter=7_7&index=5).

## WeChat Mini Program Boilerplate

To get started quickly, download the [QFPay WeChat Mini Program Boilerplate](@site/static/files/qfpay_mini_program_payments_boilerplate.zip) and get access to the MD5 hash algorithm.

<Link href="/img/miniprogram_boilerplate.png" target="_blank">
  ![WeChat Mini Program Boilerplate](@site/static/img/miniprogram_boilerplate.png)
</Link>

### Setup Instructions

1. Sign up with QFPay and we bind your WeChat appid to your API credentials.
2. Visit the WeChat MP portal at https://mp.weixin.qq.com and whitelist our environment for incoming server traffic: 开发 -\> 开发设置 -\> 服务器域名 -\> request合法域名: e.g. `https://test-openapi-hk.qfapi.com`
3. Copy and paste the files from the zip file to your local harddrive and setup a cloudfunction environment.
4. Obtain the user openid with the cloudfunction "getUserOpenID" and run the API calls accroding to the code.