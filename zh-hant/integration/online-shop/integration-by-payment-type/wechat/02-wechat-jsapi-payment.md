---
id: wechat-jsapi-payment
title: 微信 JSAPI 支付（公眾號）
description: 商戶可依此指引整合微信公眾號支付（JSAPI），包括使用實名認證與非認證模式，支援 OAuth 流程與支付參數構造。
sidebar_label: 微信 JSAPI 支付
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# 微信 JSAPI 支付（公眾號）

<Link href="/img/wechat_jsapi_process.jpg" target="_blank">![WeChat JSAPI process-flow](@site/static/img/wechat_jsapi_process.jpg)</Link>

:::warning
JSAPI 支付僅能從微信內建瀏覽器發起，無法從 Chrome、Safari 等外部瀏覽器開啟。
:::

## JSAPI 支付類型

:::info
加拿大地區的商戶請參閱 [此文檔](/docs/online-shop/alipay/alipay-web-payments)（pay_type 為 800207）。
:::

JSAPI 提供兩種整合方式：

### 1. 擁有實名認證的公眾號

商戶需註冊並完成實名認證的微信公眾號，且該帳號需與商戶之 QFPay 帳號綁定。流程如下：

- 獲取 `oauth_code` 和 `openid`
- 發起支付請求 `/trade/v1/payment`
- 跳轉至微信支付頁面

詳細步驟：

- Step 1：完成公眾號實名認證，獲得用戶 `openid`，詳見 [官方微信說明](https://developers.weixin.qq.com/doc/offiaccount/en/Getting_Started/Overview.html)
- Step 2：呼叫 QFPay 的 `/trade/v1/payment` 並傳入 `openid`，參見 [HTTP 請求](/docs/api-reference/request-format#http-request)
- Step 3：於支付授權目錄中發起支付，詳見 [微信支付文檔](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_7&index=6)

### 2. 使用 QFPay 公眾號（非認證公眾號）

此方式由 QFPay 提供微信公眾號並代為結算。適用於未擁有實名公眾號的商戶。

流程包括：
- 透過 QFPay API 取得 `oauth_code`
- 根據 code 換取 `openid`
- 呼叫支付介面

## 取得微信 oauth_code

以獲取微信 `oauth_code` 為例，GET 請求的完整 URL 結構如下：
```http
https://test-openapi-hk.qfapi.com/tool/v1/get_weixin_oauth_code?app_code=<你的 App Code>&sign=<簽名值>&redirect_uri=<跳轉網址>
```

### 說明

| 參數名稱       | 是否必填 | 描述 |
|----------------|----------|------|
| `app_code`     | 是       | 開發者憑證，由 QFPay 分配。 |
| `sign`         | 是       | 使用參數與密鑰生成的 MD5 簽名，參考 [簽名生成](/docs/api-reference/signature-generation)。 |
| `redirect_uri` | 是       | 認證完成後跳轉的網址，需為 URL 編碼格式。 |

:::note
此 URL 必須在 **微信內部瀏覽器** 中發起，否則將無法正常獲取 `oauth_code`。
:::

```python
import hashlib
import requests
from flask import Flask, redirect
from flask import request
import json
import random
import datetime
import string
import urllib
import urllib.parse

# Enter Client Credentials
environment = 'https://test-openapi-hk.qfapi.com'
app_code = "******"
client_key = "******"

# Create MD5 signature 
def make_req_sign(data, key):
    keys = list(data.keys())
    keys.sort()
    p = []
    for k in keys: 
        v = data[k]
        p.append('%s=%s'%(k,v))
    unsign_str = ('&'.join(p) + key).encode("utf-8")
    s = hashlib.md5(unsign_str).hexdigest()
    return s.upper()

def get_out_code():
    # Body payload
    redirect_uri = 'http://49ae4dbd47a6.ngrok.io/getcode'  
    data = {'app_code': app_code, 'redirect_uri': redirect_uri}   
    sign = make_req_sign(data, client_key)
    
    return environment+"/tool/v1/get_weixin_oauth_code?app_code="+app_code+"&sign="+sign+"&redirect_uri="+redirect_uri #+"&mchid="+mchid
```

:::info
成功取得 oauth_code 後將自動跳轉至 redirect_uri 並帶上 `code`。
:::

## 取得 openid

獲得 `oauth_code` 後，需呼叫以下 API 以取得用戶的 `openid`。

:::note
每次呼叫支付前，皆需重新獲取新的 `oauth_code` 與 `openid`。
:::

### HTTP 請求

`GET ../tool/v1/get_weixin_openid`


### 請求參數

| 參數名稱 | 是否必填 | 類型 | 描述 |
|----------|----------|------|------|
| `code`   | 是       | String | 從上一步 oauth_code 取得的授權碼，僅可使用一次 |
| `mchid`  | 否       | String(16) | QFPay 分配的商戶代碼 |

:::info
該請求需於 HTTP Header 中加入以下欄位：

- `X-QF-APPCODE`：QFPay 發放的 App Code  
- `X-QF-SIGN`：根據簽名規則計算所得簽名

參考：[簽名規則說明](/docs/api-reference/signature-generation)
:::


```python
def get_open_id(data):
    
    try:
        r = requests.get(environment+"/tool/v1/get_weixin_openid",params=data,headers={'X-QF-APPCODE':app_code,'X-QF-SIGN':make_req_sign(data, client_key)})
        print (r.request.url)
        print (r.content)
        if r.json()["respcd"]=="0000":
            return r.json()["openid"]
        else:
            pass
    except:
        print("An exception occurred")
```

### 回應參數

| 參數名稱 | 類型       | 描述 |
|----------|------------|------|
| `openid` | String(64) | 使用者的微信 OpenID |

```json
{
  "resperr": "",
  "respcd": 0000,
  "respmsg": "",
  "openid": "oo3Lss8d0hLOuyTuSJMVwLTk68JE"
}
```

## 發起支付

### HTTP 請求

`POST ../trade/v1/payment`（PayType: `800207`）

### 請求參數

| 參數名稱 | 是否必填 | 類型 | 描述 |
|----------|----------|------|------|
| `sub_openid` | 是 | String | 使用者的 openid，需從上述步驟取得 |
| `limit_pay` | 否 | String | 限制信用卡類型（如不允許信用卡） |
| `extend_info` | 否 | Object | 實名資訊，如身份證號與真實姓名（僅限中國大陸） |

## 回應參數（pay_params）

| 參數名稱 | 類型 | 描述 |
|----------|------|------|
| `appId` | String(16) | 小程序或公眾號 ID |
| `timeStamp` | String(32) | 時間戳記 |
| `nonceStr` | String(32) | 隨機字串 |
| `package` | String(128) | 預支付憑證資訊（prepay_id） |
| `signType` | String(32) | 簽名類型（預設 MD5） |
| `paySign` | String(64) | 簽名值 |

```python
def payment(openid):
    # Create parameter values for data payload
    current_time = datetime.datetime.now().replace(microsecond=0)                                
    # Body payload
    txamt = '1' #In USD,EUR,etc. Cent. Suggest value > 200 to avoid risk control
    txcurrcd = 'THB'
    pay_type = '800207' 
    letters = string.digits   
    out_trade_no = ''.join(random.choice(letters) for i in range(20)) 
    txdtm = current_time
    key = client_key
    
    
    data = {'txamt': txamt, 'txcurrcd': txcurrcd, 'pay_type': pay_type, 'out_trade_no': out_trade_no, 'txdtm': txdtm, 'sub_openid':openid}    
    
    try:
        r = requests.post(environment+"/trade/v1/payment",params=data,headers={'X-QF-APPCODE':app_code,'X-QF-SIGN':make_req_sign(data, key)})
        if r.json()["respcd"]=="0000":
            
            return r.json()['pay_params']
        else:
            pass
    except:
        print("An exception occurred")
    
    
app = Flask(__name__)

@app.route("/payment",methods=['GET', 'POST'])
def api_payment():
    
    if "MichroMessenger" in request.headers.get('User-Agent'):  #get an oauth_code
        print (get_out_code())
        return redirect(get_out_code(), code=302)    
    
@app.route("/getcode",methods=['GET', 'POST'])
def api_get_code():      
    print ("------------------------------------")
    print (request.args)                      
    print (request.args.get("code"))
    code = request.args.get('code')
    print (code)
    if code != "":    # user returned with oauth_code      
        
        sub_openid=get_open_id({"code": code})  # get open id using oauth_code
        param=payment(sub_openid)   # payment request to QFPay
        
        # add necessary parameters and redirect
        param["mchntnm"]="Pet Shop"
        param["txamt"]="1"
        param["currency"]="THB"
        param["redirect_url"]="www.example.com"
        return redirect("https://o2-hk.qfapi.com/q/direct?"+urllib.parse.urlencode(param), code=302)             # direct user"""
    else:
        print("unable to obtain code")
        return     

if __name__ == '__main__':
    app.run(host="127.0.0.1",port = 80)
```

## 調用微信支付模組

`GET https://o2-hk.qfapi.com/q/direct`

### 請求參數

| 參數名稱 | 是否必填 | 類型 | 描述 |
|----------|----------|------|------|
| `mchntnm` | 是 | String(128) | 商戶名稱，若為中文須以 UTF-8 編碼 |
| `txamt` | 是 | Int(11) | 金額（如 100 = $1） |
| `currency` | 是 | String(3) | 幣別 |
| `redirect_url` | 是 | String(512) | 支付完成後導向的 URL（需 urlencode） |
| `package`、`timeStamp`、`signType`、`paySign`、`appId`、`nonceStr` | 是 | 各類型 | 請使用前段 API 回傳之 pay_params |

---

如需進一步使用範例或測試請求，請聯絡技術支援團隊取得測試憑證與公眾號授權資訊。


