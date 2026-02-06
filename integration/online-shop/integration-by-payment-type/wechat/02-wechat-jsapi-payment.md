---
id: wechat-jsapi-payment
title: WeChat JSAPI Payment (Official Account)
description: This guide walks merchants through integrating WeChat JSAPI (in-app official account) payments, with two implementation methods and full authorization/payment steps.
sidebar_label: WeChat JSAPI Payment
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# WeChat JSAPI Payment (Official Account)

<Link href="/img/wechat_jsapi_process.jpg" target="_blank">![WeChat JSAPI process-flow](@site/static/img/wechat_jsapi_process.jpg)</Link>

:::warning
This payment method must be initiated from within the WeChat in-app browser. It cannot be launched from external browsers such as Chrome or Safari.
:::

## Overview of JSAPI Payment Methods

> For merchants in Canada, please refer to [this documentation](/docs/online-shop/alipay/alipay-web-payments), where `pay_type` is `800207`.

There are currently two integration methods for JSAPI payments:

---

## Method 1: Own Verified Official Account

Merchants must have their own verified WeChat official account and link it to their QFPay payment account.

### Steps

1. After verifying the official account, merchants can obtain the user’s `openid`. See [WeChat official docs](https://developers.weixin.qq.com/doc/offiaccount/en/Getting_Started/Overview.html).
2. Call `/trade/v1/payment` with the `openid` to get `pay_params`. See [API Request Format](/docs/api-reference/request-format#http-request).
3. Ensure your domain is added to the JSAPI whitelist, then invoke WeChat payment using JSAPI. See [WeChat JSAPI Payment Guide](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_7&index=6).

---

## Method 2: Use QFPay’s Official Account (Settlement by QFPay)

If you do not have a verified account, QFPay provides an official account for indirect settlement merchants.

---

## Step 1: Get WeChat `oauth_code`

**HTTP Request:**  
`GET /tool/v1/get_weixin_oauth_code`

:::note
Must be opened in WeChat browser. `app_code` and `sign` should be query parameters, not in HTTP headers.
:::

### Request Parameters

| Name        | Param        | Required | Type         | Description                                  |
|-------------|--------------|----------|--------------|----------------------------------------------|
| Developer ID | `app_code`   | Yes      | String(32)   | Assigned by QFPay                            |
| Redirect URL | `redirect_uri` | Yes    | String(512)  | Where users will be redirected post-auth     |
| Merchant ID  | `mchid`      | No       | String(16)   | Assigned by QFPay                            |
| Signature    | `sign`       | Yes      | String       | MD5 signature using `client_key`             |

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

**Example Response:**

```json
{
  "redirect": "http://yourdomain.com/callback?code=011xxxxx"
}
```

## GET openid

**HTTP Request:**  
```http
{
  https://test-openapi-hk.qfapi.com/tool/v1/get_weixin_openid?code=011QipnO1yMIla1VJdoO1FUrnO1Qipnv
}
```

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

> The above command returns JSON structured like this:

```json
{
  "resperr": "",
  "respcd": 0000,
  "respmsg": "",
  "openid": "oo3Lss8d0hLOuyTuSJMVwLTk68JE"
}
```

:::note Everytime the payment interface is called a new `oauth_code` and `openid` must be obtained. In order to request the `openid` the `X-QF-APPCODE` and `X-QF-SIGN` have to be submitted in the http header.
:::

## Step 2: Obtain `openid`

After the user authorises via WeChat OAuth, you must exchange the returned `oauth_code` for the user’s `openid`.

### HTTP Request

`GET /tool/v1/get_weixin_openid`

### Request Parameters

| Parameter | Required | Type | Description |
|---------|----------|------|-------------|
| `code` | Yes | String | The WeChat `oauth_code` returned from **Step 1**. This code is **single-use only**. |
| `mchid` | No | String(16) | QFPay-assigned Merchant ID (required only for certain merchants). |

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

### Response Parameters

|Attribute| Type | Description |
|:-----  |:-----|----- |
|`openid`|String(64)  | WeChat openid, every WeChat user is assigned a unique openid |

### Sample Response

```json
{
  "resperr": "",
  "respcd": 0000,
  "respmsg": "",
  "openid": "oo3Lss8d0hLOuyTuSJMVwLTk68JE"
}
```

:::note
A new `oauth_code` and `openid` must be obtained for every payment attempt.
Do not cache or reuse these values.
:::

## Step 3: Submit Payment Request

Once `openid` is available, submit a JSAPI payment request to QFPay.


**HTTP Request**

`POST /trade/v1/payment`  
`pay_type: 800207`（WeChat JSAPI）

### Request Parameters

| Parameter | Field | Required | Type | Description |
|----------|----------|----------|------|------|
| WeChat User ID | `sub_openid` | Yes | String | The `openid` obtained in Step 2. |
| Payment Method Limit | `limit_pay` | No | String | Restrict specific payment methods (e.g. disable credit cards). |
| Customer Extended Info | `extend_info` | No | Object | Real-name verification data (Mainland China only). Includes user_creid (ID number) and user_truename (real name). Activation depends on merchant setup and PayType. See 
[PayType Reference](/docs/api-reference/paytypes#paytypes)。 |
| Other Common Fields | — | Yes | — | Amount, currency, timestamp, order number, etc. Refer to
 [Payment API Request Format](/docs/api-reference/request-format#http-request)。 |

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

### Response Parameters (pay_params)

On success, the API returns `pay_params`, which are required to invoke the WeChat JSAPI payment module.

| Field | Type | Description |
|----------|------|------|
| `appId` | String(16) | WeChat Official Account App ID |
| `timeStamp` | String(32) | Current timestamp (string format) |
| `nonceStr` | String(32) | Random nonce value |
| `package` | String(128) | Payment package in format `prepay_id=***` |
| `signType` | String(32) | Signature method (default: `MD5`) |
| `paySign` | String(64) | Payment signature |

---

## Step 4: Redirect to WeChat Payment Module

After receiving `pay_params` redirect the user to the WeChat payment module to complete the payment.

**HTTP Request:**

`GET https://o2-hk.qfapi.com/q/direct`

### Request Parameters

| Parameter | Required | Type | Description |
|----------|----------|------|------|
| `mchntnm` | 是 | String(128) | Merchant display name shown on the payment page. UTF-8 encoding required for non-ASCII characters. |
| `txamt` | 是 | Int(11) | URL to redirect the user after payment. Must be URL-encoded. |
| `currency` | 是 | String(3) | URL to redirect the user after payment. Must be URL-encoded. |
| `goods_name` | 否 | String(64) | URL to redirect the user after payment. Must be URL-encoded. |
| `redirect_url` | 是 | String(512) | URL to redirect the user after payment. Must be URL-encoded. |
| `package` | 是 | String(128) | Value returned in `pay_params`. |
| `timeStamp` | 是 | String(32) | Value returned in `pay_params`. |
| `signType` | 是 | String(32) | Value returned in `pay_params`. |
| `paySign` | 是 | String(64) | Value returned in `pay_params`. |
| `appId` | 是 | String(16) | Value returned in `pay_params`. |
| `nonceStr` | 是 | String(32) | Value returned in `pay_params`. |

:::note
This request is sent directly from the user’s browser.
No `X-QF-APPCODE` or `X-QF-SIGN` headers are required at this stage.
:::

---

## Additional Notes
- The JSAPI flow must be executed sequentially; `oauth_code` and `openid` cannot be reused
- If payment status is not immediately confirmed, use the [Transaction Enquiry API](/docs/common-api/transaction-enquiry) to verify the final result
- Real-name verification is optional and depends on merchant configuration and business requirements