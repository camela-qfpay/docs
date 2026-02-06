---
id: wechat-web-qr-payment
title: 微信掃碼支付（WeChat QR Code Payment）
description: 商戶透過本指引可整合微信線上掃碼支付功能，生成 QR Code 供顧客掃碼付款，支援身份驗證、超時設定等功能。
sidebar_label: 微信掃碼支付
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# 微信掃碼支付（WeChat QR Code Payment）

<Link href="/img/online_qr_process.png" target="_blank">
![WeChat QR Code Payment Flow](@site/static/img/online_qr_process.png)
</Link>

## 產品說明

微信掃碼支付允許顧客使用 **微信 App 的「掃一掃」功能**，掃描商戶網站或系統產生的 **專屬支付二維碼（QRCode）** 以完成付款。

在此支付模式下：
- 商戶系統會將訂單資訊嵌入至唯一的 QR Code
- 顧客掃碼後，微信將進行必要的安全檢查
- 驗證完成後即會完成付款流程

此方式常用於：
- 桌面網站付款
- POS 顯示 QR Code
- Kiosk / 自助機付款場景

---

## 實名認證（選用）

商戶可選擇啟用 **微信實名認證**（Real-name Verification）。

:::note
目前實名認證 **僅適用於中國大陸公民**，需提供：
- 付款人真實姓名
- 中國居民身份證號碼
:::

實名認證規則說明：
- 若商戶已提供身份資訊，付款人微信錢包（如綁定銀行卡）需與提供資料一致
- 若付款人尚未綁定銀行卡，**仍可完成付款**
- 是否強制實名，依商戶開通設定為準

---

## 範例程式碼（多語言）

本文件提供以下語言的請求範例，說明如何呼叫微信掃碼支付 API：

- Python  
- Java  
- JavaScript (Node.js)  
- PHP  

:::info
請依實際開發語言選擇對應範例。所有範例邏輯一致，僅語法不同。
:::

<Tabs>
<TabItem value="python" label="Python">

```python
#coding=utf8
import urllib.request, urllib.parse, urllib.error, urllib.request, urllib.error, urllib.parse, hashlib
import requests
import datetime
import string

# Enter Client Credentials
environment = 'https://test-openapi-hk.qfapi.com'
app_code = 'D5589D2A1F2E42A9A60C37*********'
client_key = '0E32A59A8B454940A2FF39**********'


# Create parameter values for data payload
current_time = datetime.datetime.now().replace(microsecond=0)                                

print(current_time)

# Create signature
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


# Body payload
txamt = '10' #In USD,EUR,etc. Cent. Suggest value > 200 to avoid risk control
txcurrcd = 'HKD'
pay_type = '800201'
auth_code='283854702356157409' #CPM only
out_trade_no = '01234567890123'
txdtm = current_time
goods_name = 'test1'   
mchid = 'ZaMVg*****'
key = client_key


#data ={'txamt': txamt, 'txcurrcd': txcurrcd, 'pay_type': pay_type, 'out_trade_no': out_trade_no, 'txdtm': txdtm, 'goods_name': goods_name, 'udid': udid, 'mchid': mchid}
data ={'txamt': txamt, 'txcurrcd': txcurrcd, 'pay_type': pay_type, 'out_trade_no': out_trade_no, 'txdtm': txdtm, 'mchid': mchid}

r = requests.post(environment+"/trade/v1/payment",data=data,headers={'X-QF-APPCODE':app_code,'X-QF-SIGN':make_req_sign(data, key)})

print(r.json())
```

</TabItem>
<TabItem value="java" label="Java">

```java
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;


public class TestMain {
    public static void main(String args[]){
        String appcode="D5589D2A1F2E42A9A60C37*********";
        String key="0E32A59A8B454940A2FF39*********";
        String mchid="ZaMVg*****";

        String pay_type="800201";
        String out_trade_no= "01234567890123";
        SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String date=df.format(new Date());
        String txdtm=date;
        String txamt="10";
        String txcurrcd="EUR";

        Map<String, String> unsortMap = new HashMap<>();
        unsortMap.put("mchid", mchid);
        unsortMap.put("pay_type", pay_type);
        unsortMap.put("out_trade_no", out_trade_no);
        unsortMap.put("txdtm", txdtm);
        unsortMap.put("txamt", txamt);
        unsortMap.put("txcurrcd", txcurrcd);
        //unsortMap.put("product_name", product_name);
        //unsortMap.put("valid_time", "300");

        String data=QFPayUtils.getDataString(unsortMap);
        System.out.println("Data:\n"+data+key);
        String md5Sum=QFPayUtils.getMd5Value(data+key);
        System.out.println("Md5 Value:\n"+md5Sum);

        String url="https://test-openapi-hk.qfapi.com";
        String resp= Requests.sendPostRequest(url+"/trade/v1/payment", data, appcode,key);
        System.out.println(resp);
    }
}
```

</TabItem>
<TabItem value="javascript" label="JavaScript">

```javascript
// Enter Client Credentials
const environment = 'https://test-openapi-hk.qfapi.com'
const app_code = 'D5589D2A1F2E42A9A60C37*********'
const client_key = '0E32A59A8B454940A2FF39*********'

// Generate Timestamp
var dateTime = new Date().toISOString().replace(/T/, ' ').replace(/\..+/, '')
console.log(dateTime)

// Body Payload
const key = client_key
var tradenumber = String(Math.round(Math.random() * 1000000000))
console.log(tradenumber)

var payload = {
'txamt': '10', // In USD,EUR,etc. Cent. Suggest value > 200 to avoid risk control
'txcurrcd': 'HKD',
'pay_type': '800201',
'out_trade_no': tradenumber,
'txdtm': dateTime,
'mchid': 'ZaMVg*****'
};

// Signature Generation
const ordered = {};
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

const crypto = require('crypto')
var hashed = crypto.createHash('md5').update(string).digest('hex')
console.log(hashed)


// API Request
var request = require("request");
request({
  uri: environment+"/trade/v1/payment",
  headers: {
    'X-QF-APPCODE': app_code,
    'X-QF-SIGN': hashed
  },
  method: "POST",
  form: payload,
  }, 
  function(error, response, body) {
  console.log(body);
});
```

</TabItem>
<TabItem value="php" label="Php">

```php
<?php
ob_start();
  function GetRandStr($length){
  $str='abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
  $len=strlen($str)-1;
  $randstr='';
  for($i=0;$i<$length;$i++){
  $num=mt_rand(0,$len);
  $randstr .= $str[$num];
  }
  return $randstr;
  }
  
     $url = 'https://test-openapi-hk.qfapi.com';
     $api_type = '/trade/v1/payment';
     $pay_type = '800201';
     //$mchid = "MNxMp11FV35qQN"; //Only agents must provide this parameter
     $app_code = 'FF2FF74F2F2E42769A4A73*********'; //API credentials are provided by QFPay
     $app_key = '7BE791E0FD2E48E6926043B*********'; //API credentials are provided by QFPay
     $now_time = date("Y-m-d H:i:s"); //Get current date-time
     
     $fields_string = '';
     $fields = array(
      //'mchid' => urlencode($mchid),
      'pay_type' => urlencode($pay_type),
      'out_trade_no' => urlencode(GetRandStr(20)),
      'txcurrcd' => urlencode('HKD'),
      'txamt' => urlencode(2200),
      'txdtm' => $now_time
    );
    ksort($fields); //字典排序A-Z升序方式
    print_r($fields);
    
    foreach($fields as $key=>$value) { 
  $fields_string .= $key.'='.$value.'&' ;
  }
  $fields_string = substr($fields_string , 0 , strlen($fields_string) - 1);
  
  $sign = strtoupper(md5($fields_string . $app_key));
  
  //// Header ////
  $header = array();
  $header[] = 'X-QF-APPCODE: ' . $app_code;
  $header[] = 'X-QF-SIGN: ' . $sign;
  
  //Post Data
  $ch = curl_init();
  curl_setopt($ch, CURLOPT_URL, $url . $api_type);
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
  curl_setopt($ch, CURLOPT_HTTPHEADER, $header);
  curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
  curl_setopt($ch, CURLOPT_POST, 1);
  curl_setopt($ch, CURLOPT_POSTFIELDS, $fields_string);
  $output = curl_exec($ch);
  curl_close($ch);    

  $final_data = json_decode($output, true);
  print_r($final_data);

ob_end_flush();
?>
```

</TabItem>
</Tabs>

---

## API 回傳結果說明

API 成功請求後，將回傳包含 **QRCode URL** 的回應資料。

:::info
商戶需將回傳的 `qrcode` 欄位內容轉換為 QR Code 圖像，並展示給顧客掃描。
:::

```json
{
  "sysdtm": "2020-04-10 11:45:44", 
  "paydtm": "2020-04-10 11:45:44", 
  "txcurrcd": "HKD", 
  "respmsg": "OK", 
  "qrcode": "weixin://wxpay/bizpayurl?pr=4PsXP5N", 
  "pay_type": "800201", 
  "cardcd": "", 
  "udid": "qiantai2", 
  "txdtm": "2020-04-10 11:45:44", 
  "txamt": "300", 
  "resperr": "success", 
  "out_trade_no": "3Z6HPCS6RN54J2Y8LUQM8RBDVBA9URYE", 
  "syssn": "20200410000300020086358791", 
  "respcd": "0000", 
  "chnlsn": ""
  }
```

---

## HTTP 請求說明

- **Method**：`POST`  
- **Endpoint**：`/trade/v1/payment`  
- **PayType**：`800201`（微信掃碼支付）

---

## 請求參數

| 參數名稱         | 參數編碼       | 二級參數                 | 是否必填               | 參數類型     | 描述 |
|------------------|----------------|--------------------------|------------------------|--------------|------|
| 訂單支付金額     | `txamt`        | –                        | 是                  | Int(11)      | 單位為最小幣值（如 100 = $1），僅支援整數。**建議大於 200** 以避免被風控拒付。 |
| 幣種             | `txcurrcd`     | –                        | 是                  | String(3)    | 請參考 [支援的交易貨幣](/docs/api-reference/currencies)。 |
| 支付類型         | `pay_type`     | –                        | 是                  | String(6)    | 微信掃碼支付固定為 `800201`。參見 [支付類型一覽表](/docs/api-reference/paytypes#paytypes)。 |
| 外部訂單號       | `out_trade_no` | –                        | 是                  | String(128)  | 商戶自定義交易單號，在同一商戶下需具備唯一性。 |
| 請求交易時間     | `txdtm`        | –                        | 是                  | String(20)   | 時間格式為 `YYYY-MM-DD hh:mm:ss`。 |
| 交易到期時間     | `expired_time` | –                        | 否（限正掃）         | String(3)    | 單位為分鐘，預設 30 分鐘，可設為 5–120 分鐘。 |
| 商品名稱         | `goods_name`   | –                        | 否                  | String(64)   | 建議僅使用英數與常規標點，若為中文請使用 UTF-8 編碼。 |
| 子商戶號         | `mchid`        | –                        | 否                  | String(16)   | 僅代理商或特定場景需提供，請聯絡技術人員確認是否必填。 |
| 設備唯一 ID      | `udid`         | –                        | 否                  | String(40)   | 商戶裝置識別碼，將出現在後台交易列表中。 |
| 人民幣標記       | `rmb_tag`      | –                        | 否                  | String(1)    | 如支付幣種為 CNY 且使用香港錢包，需設定為 `Y`。 |
| 客戶擴展資訊     | `extend_info`  | `user_creid`, `user_truename` | 否              | Object       | 實名制身份驗證，僅限中國大陸公民。參數 `user_creid` 中包含消費者的身分證號碼，`user_truename` 中必須提供編碼形式或漢字書寫的付款人真實姓名。 例子如下： `extend_info = '{"user_creid":"430067798868676871","user_truename":"\\u5c0f\\u6797"}'` |

---

## 回應參數

| 參數名稱        | 參數編碼     | 類型        | 描述 |
|-----------------|--------------|-------------|------|
| 交易類型        | `pay_type`   | String(6)   | 固定為 `800201`，代表微信掃碼支付。 |
| 系統交易時間    | `sysdtm`     | String(20)  | QFPay 系統產生的交易完成時間。 |
| 請求交易時間    | `txdtm`      | String(20)  | 商戶提交的原始交易時間。 |
| 響應訊息        | `resperr`    | String(128) | 系統處理結果，例如 OK、failed。 |
| 支付金額        | `txamt`      | Int(11)     | 實際扣款金額（單位為最小幣值）。 |
| 額外留言        | `respmsg`    | String(128) | 附加錯誤提示或返回資訊。 |
| 外部交易號碼    | `out_trade_no` | String(128) | 商戶提供的訂單編號。 |
| QFPay 訂單編號  | `syssn`      | String(40)  | QFPay 系統產生的訂單 ID。 |
| 返回碼          | `respcd`     | String(4)   | `0000` 表示成功；其餘代碼請參考 [交易狀態碼](/docs/api-reference/status-codes)。 |
| 通道流水號      | `chnlsn`     | String      | 第三方平台回傳的交易流水號（如微信訂單編號）。 |

:::warning
當回傳碼為 `1143` 或 `1145` 時，代表交易狀態未確定，**商戶必須主動查詢交易結果**。
:::

---

## 小結

- 微信掃碼支付適合「顯示 QR Code → 顧客主動掃碼」的場景  
- 商戶需自行顯示 QR Code 圖像  
- 可選擇是否啟用實名認證  
- 建議實作交易結果查詢機制以確保狀態正確  