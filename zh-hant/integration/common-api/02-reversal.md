---
id: reversal
title: 沖正 API 指南
description: 使用沖正 API 撤回尚未完成的交易。若交易已成功，請改用退款 API。
sidebar_label: 沖正 API
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 沖正 API 指南

本頁面說明如何使用「沖正 API」來撤銷一筆**尚未成功完成**的交易。請注意，沖正並非退款。

:::warning
沖正（reversal）不是退款。只有當原始交易尚未成功完成時，才能發起沖正。
:::

---

## 支援場景

:::note
目前僅以下場景支援 `/trade/v1/reversal` 沖正 API，適用於交易處於非完成狀態（例如掃碼後未付款）。
:::

### 支援的支付場景及 PayType

支付寶正掃
- `800101`：支付寶跨境線下掃碼支付
- `801501`：支付寶線上掃碼支付 (香港商戶)

微信支付正掃
- `800201`：微信掃碼支付

支付寶反掃
- `800108`：支付寶跨境反掃支付

> 若您欲對其他錢包進行交易取消，請參考 [Close API](#reversal-vs-close) 或聯絡 QFPay 支持團隊取得整合建議。

---

## API 端點

* **Endpoint**：`/trade/v1/reversal`
* **方法**：`POST`

若沖正成功，回應會包含 `respcd=0000`。

若原始交易已成功完成（即付款回應中為 `respcd=0000`），則**無法沖正**，請改用 [退款 API](/docs/common-api/refund)。

---

## 請求參數

| 參數名稱           | 類型          | 必填 | 說明                             |
| -------------- | ----------- | -- | ------------------------------ |
| `mchid`        | String(16)  | 否  | QFPay 商戶號，僅代理商需填寫。             |
| `syssn`        | String(40)  | 是* | QFPay 交易號                      |
| `out_trade_no` | String(128) | 是* | 商戶端交易單號                        |
| `txamt`        | Int(11)     | 是  | 交易金額（單位：分），建議金額大於 200 以避免風控    |
| `txdtm`        | String(20)  | 是  | 原交易時間，格式：`YYYY-MM-DD hh:mm:ss` |
| `udid`         | String(40)  | 否  | 裝置唯一 ID（用於追蹤交易設備）              |

> * `syssn` 與 `out_trade_no` 至少擇一提供。

---

## 回應參數

| 參數名稱         | 類型        | 說明                            |
| ------------ | --------- | ----------------------------- |
| `syssn`      | String    | 此次沖正動作的 QFPay 交易號             |
| `orig_syssn` | String    | 原始（被沖正）交易的 QFPay 交易號          |
| `txamt`      | Int       | 沖正金額（單位：分）                    |
| `txcurrcd`   | String(3) | 幣別代碼（例如 HKD）                  |
| `txdtm`      | String    | 原始交易時間                        |
| `sysdtm`     | String    | QFPay 系統處理時間                  |
| `chnlsn`     | String    | 錢包端交易號（若未執行則可能為空）             |
| `respcd`     | String(4) | 回應代碼（`0000` = 成功，其它 = 失敗或處理中） |
| `resperr`    | String    | 結果訊息                          |
| `respmsg`    | String    | 額外補充說明（如有）                    |

---

## 程式碼範例

<Tabs>
<TabItem value="python" label="Python">

```python

import urllib.request, urllib.parse, urllib.error, urllib.request, urllib.error, urllib.parse, hashlib
import requests
from hashids import Hashids
import datetime
import string
import random

# Enter Client Credentials
environment = 'https://test-openapi-hk.qfapi.com'
app_code = '3F504C39125E4886AB4741**********'
client_key = '5744993FBC034DBBB995FA**********'


# Create parameter values for data payload
current_time = datetime.datetime.now().replace(microsecond=0)                                
random_string = ''.join(random.choices(string.ascii_uppercase + string.digits, k=32))

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
    print(unsign_str)
    s = hashlib.md5(unsign_str).hexdigest()
    return s.upper()



# Body payload
txamt = '2500' #In USD,EUR,etc. Cent. Suggest value > 200 to avoid risk control
out_trade_no = '4MDGEJ7L496LAAU1V1HBY9HMOGWZWLXQ'
syssn = '20200305066100020000977812' 
txdtm = '2020-03-05 16:50:30' 
mchid = 'MNxMp11FV35qQN'
key = client_key

#data ={'txamt': txamt, 'out_trade_no': out_trade_no, 'syssn': syssn, 'txdtm': txdtm, 'udid': udid, 'mchid': mchid}
data ={'txamt': txamt, 'out_trade_no': out_trade_no, 'txdtm': txdtm, 'mchid': mchid}

r = requests.post(environment+"/trade/v1/reversal",data=data,headers={'X-QF-APPCODE':app_code,'X-QF-SIGN':make_req_sign(data, key)})

print(r.json())
```

</TabItem>
<TabItem value="java" label="Java">

```java

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;


public class Refund {
    public static void main(String args[]){
        String appcode="3F504C39125E4886AB4741**********";
        String key="5744993FBC034DBBB995FA**********";
        String mchid="MNxMp11FV35qQN"; // Only Agents must provide the mchid

        SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String date=df.format(new Date());

        String txdtm="2020-03-05 16:50:30"; 
        String txamt="2500";
        String syssn="20200305066100020000977812"; //only syssn or out_trade_no must be provided
        String out_trade_no="4MDGEJ7L496LAAU1V1HBY9HMOGWZWLXQ"; //only syssn or out_trade_no must be provided


        Map<String, String> unsortMap = new HashMap<>();
        unsortMap.put("mchid", mchid);
        unsortMap.put("txamt", txamt);
        unsortMap.put("syssn", syssn);
        unsortMap.put("out_trade_no", out_trade_no);
        unsortMap.put("txdtm", txdtm);

        String data=QFPayUtils.getDataString(unsortMap);
        System.out.println("Data:\n"+data+key);
        String md5Sum=QFPayUtils.getMd5Value(data+key);
        System.out.println("Md5 Value:\n"+md5Sum);

         //如果是国内钱台，网址是：https://test-openapi-hk.qfapi.com.
        String url="https://test-openapi-hk.qfapi.com";
        String resp= Requests.sendPostRequest(url+"/trade/v1/reversal", data, appcode,key);
        System.out.println(resp);
    }
}
```

</TabItem>
<TabItem value="javascript" label="Javascript">

```javascript

// Enter Client Credentials
environment = 'https://test-openapi-hk.qfapi.com'
app_code = '3F504C39125E4886AB4741**********'
client_key = '5744993FBC034DBBB995FA**********'

// Generate Timestamp
var dateTime = new Date().toISOString().replace(/T/, ' ').replace(/\..+/, '')
console.log(dateTime)

// Body Payload
const key = client_key
var tradenumber = String(Math.round(Math.random() * 1000000000))
console.log(tradenumber)

var payload = {
'txamt': '2500', 
'out_trade_no': '4MDGEJ7L496LAAU1V1HBY9HMOGWZWLXQ', //only syssn or out_trade_no must be provided
'syssn': '20200305066100020000977812', //only syssn or out_trade_no must be provided
'txdtm': '2020-03-05 16:50:30',
'mchid': 'MNxMp11FV35qQN'
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
  uri: environment+"/trade/v1/reversal",
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
$api_type = '/trade/v1/reversal';
$syssn = '800101';
//$out_trade_no = 'zCvo0IqTg0SaQkGnHd6w';
//$mchid = "MNxMp11FV35qQN"; //Only agents must provide this parameter
$app_code = 'FF2FF74F2F2E42769A4A73E********'; //API credentials are provided by QFPay
$app_key = '7BE791E0FD2E48E6926043B5********'; //API credentials are provided by QFPay
$now_time = date("Y-m-d H:i:s"); //Get the current date-time   

$fields_string = '';
$fields = array(
    //'mchid' => urlencode($mchid),
    'syssn' => urlencode($syssn),
    //'out_trade_no' => urlencode($out_trade_no),
    'txcurrcd' => urlencode('HKD'),
    'txamt' => urlencode(2200),
    'txdtm' => date('2020-03-05 16:50:30'),
);
ksort($fields); //Sort parameters in ascending order from A to Z
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

## 回應範例

```json
{
    "surcharge_fee": "0", 
    "resperr": "success", 
    "txdtm": "2020-03-05 16:50:30", 
    "syssn": "20200305066100020000977814", 
    "sysdtm": "2020-03-05 16:54:38", 
    "txcurrcd": "EUR", 
    "respmsg": "", 
    "chnlsn2": "", 
    "cardcd": "", 
    "udid": "qiantai2", 
    "txamt": "2500", 
    "orig_syssn": "20200305066100020000977813", 
    "surcharge_rate": "0", 
    "respcd": "0000", 
    "chnlsn": ""
}
```

---

## 補充說明

* **沖正無法保證使用者未被扣款**，建議搭配使用 [交易查詢 API](/docs/common-api/transaction-enquiry) 驗證結果。
* 沖正僅適用於**即時失敗的交易**，請勿用於已成功的訂單退款處理。
* 若回應為 `respcd=1143` 或 `1145`，表示沖正處理中，應持續查詢其狀態直至確認結果。

---

## Reversal(沖正) vs Close（關閉訂單）
部分錢包採用關閉訂單接口 `/trade/v1/close` 取代沖正。

### 支援的支付場景及 PayType

微信支付反掃
- `800008`：微信反掃
- `800208`：微信反掃支付
- `801008`：微信香港反掃支付（適用於向微信香港申請的商戶）


**方法**：`GET`

如需於非支付寶／微信錢包上使用此接口，請與我們聯繫確認。

### 請求參數

| 參數名稱           | 必填 | 類型          | 說明                            |
| -------------- | -- | ----------- | ----------------------------- |
| `mchid`        | 否  | String(16)  | QFPay 配發的商戶號碼                 |
| `syssn`        | 是* | String(40)  | 系統回傳的 QFPay 交易號               |
| `out_trade_no` | 是* | String(128) | 商戶自訂訂單編號                      |
| `txamt`        | 是  | Int(11)     | 交易金額（分為單位）。建議金額 > 200。        |
| `txdtm`        | 是  | String(20)  | 交易時間，格式：`YYYY-MM-DD hh:mm:ss` |
| `udid`         | 否  | String(40)  | 裝置識別碼，將於商戶後台顯示                |

> * `syssn` 與 `out_trade_no` 至少擇一提供。

### 回應參數

| 參數名稱           | 類型          | 說明                                                    |
| -------------- | ----------- | ----------------------------------------------------- |
| `orig_syssn`   | String(40)  | 原始 QFPay 交易號                                          |
| `syssn`        | String(40)  | 關單交易的 QFPay 交易號                                       |
| `out_trade_no` | String(128) | 商戶訂單編號                                                |
| `txamt`        | Int(11)     | 交易金額（單位：分）                                            |
| `txcurrcd`     | String(3)   | 幣別代碼，詳見 [交易貨幣](/docs/api-reference/currencies) |
| `txdtm`        | String(20)  | 原始交易時間                                                |
| `sysdtm`       | String(20)  | 系統處理時間。此值將用作結算截止時間。                                   |
| `chnlsn`       | String      | 錢包端交易號                                                |
| `respcd`       | String(4)   | 回應代碼：`0000` = 成功，`1143/1145` = 處理中，其餘 = 失敗            |
| `resperr`      | String(128) | 結果描述                                                  |
| `respmsg`      | String(128) | 補充資訊                                                  |