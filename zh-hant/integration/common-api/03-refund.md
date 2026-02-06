---
id: refund
title: 退款 API 指南
description: 使用退款 API 處理成功交易的全額或部分退款。
sidebar_label: 退款 API
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 退款 API 指南

:::tip
僅當交易返回碼為 `0000`（交易成功）時，才能進行退款操作。
:::

:::note
若為信用卡交易，由於系統會自動進行捕獲（Capture），同日退款申請將視為 void 並自動處理，無需商戶另外判斷。
:::

:::note
不同錢包的退款規則（如可退款時間上限、是否支持部分退款）可能有所不同，請聯繫 QFPay 支援確認。
:::

## API Endpoint

```plaintext
Endpoint: /trade/v1/refund
Method: POST
```

### HTTP 請求說明

- Content-Type: `application/x-www-form-urlencoded`
- Headers:
  - `X-QF-APPCODE`: 你的 App Code
  - `X-QF-SIGN`: 使用簽名函式生成的簽名值

## 請求參數

| 參數            | 必填 | 類型         | 說明                                                                 |
|-----------------|------|--------------|----------------------------------------------------------------------|
| `syssn`         | 是   | String(128)  | 欲退款的原始交易 ID                                                 |
| `out_trade_no`  | 是   | String(128)  | 外部退款訂單號（不可與原訂單號重複）                                |
| `txamt`         | 是   | Int(11)      | 退款金額（單位為分）。部分錢包不支持部分退款，建議金額大於 200    |
| `txdtm`         | 是   | String(20)   | 請求時間，格式為：YYYY-MM-DD hh:mm:ss                              |
| `mchid`         | 否   | String(16)   | 商戶號，如系統已分配給商戶，則為必填                               |
| `udid`          | 否   | String(40)   | 裝置 ID，用於識別交易設備                                           |

## 回應參數

| 參數                 | 類型         | 說明                                                                           |
|----------------------|--------------|----------------------------------------------------------------------------------|
| `syssn`              | String(40)   | 此次退款交易所對應的交易號                                                     |
| `orig_syssn`         | String(128)  | 原始交易號                                                                      |
| `txamt`              | Int(11)      | 退款金額（單位為分）                                                           |
| `sysdtm`             | String(20)   | 系統退款時間，格式：YYYY-MM-DD hh:mm:ss，用作結算分界時間                     |
| `respcd`             | String(4)    | 返回碼：`0000` 成功，`1143/1145` 處理中，其它為失敗。請參考狀態碼說明文件     |
| `resperr`            | String(128)  | 返回訊息                                                                       |
| `cash_fee`           | String       | 實際支付金額（交易金額 - 折扣）                                               |
| `cash_fee_type`      | String       | 實際支付幣種，如 CNY                                                           |
| `cash_refund_fee`    | String       | 實際退款金額                                                                   |
| `cash_refund_fee_type` | String     | 實際退款幣種，如 CNY                                                           |

## 範例程式碼

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
app_code = 'D5589D2A1F2E42A9A60C37**********'
client_key = '0E32A59A8B454940A2FF39**********'


# Create parameter values for data payload
current_time = datetime.datetime.now().replace(microsecond=0)         
random_string = ''.join(random.choices(string.ascii_uppercase + string.digits, k=32))                       
  

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
txamt = '10' #Partial or full refund amount
syssn = '20191227000200020061752831' #Original transaction number
out_trade_no = random_string
txdtm = current_time 
key = client_key
mchid = 'ZaMVg*****'


#data ={'txamt': txamt, 'syssn': syssn, 'out_trade_no': out_trade_no, 'txdtm': txdtm, 'udid': udid, 'mchid': mchid}
data ={'mchid': mchid, 'txamt': txamt, 'syssn': syssn,  'out_trade_no': out_trade_no, 'txdtm': txdtm}

r = requests.post(environment+"/trade/v1/refund",data=data,headers={'X-QF-APPCODE':app_code,'X-QF-SIGN':make_req_sign(data, key)})

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
        String appcode="D5589D2A1F2E42A9A60C37**********";
        String key="0E32A59A8B454940A2FF39**********";
        String mchid="ZaMVg*****"; // Only Agents must provide the mchid

        String out_trade_no= "22333444455555";
        SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String date=df.format(new Date());
        String txdtm=date;
        String txamt="15";
        String syssn="20191227000300020061662295";
         //如果是国内钱台，产品名称对应的字段是goods_name，不是product_name.
         //String product_name="Test Name";


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
        String resp= Requests.sendPostRequest(url+"/trade/v1/refund", data, appcode,key);
        System.out.println(resp);
    }
}
```

</TabItem>
<TabItem value="javascript" label="Javascript">

```javascript
// Enter Client Credentials
const environment = 'https://test-openapi-hk.qfapi.com'
const app_code = 'D5589D2A1F2E42A9A60C37**********'
const client_key = '0E32A59A8B454940A2FF3***********'

// Generate Timestamp
var dateTime = new Date().toISOString().replace(/T/, ' ').replace(/\..+/, '')
console.log(dateTime)

// Body Payload
const key = client_key
var tradenumber = String(Math.round(Math.random() * 1000000000))
console.log(tradenumber)

var payload = {
'syssn': '20191231000300020063521806',
'txamt': '10',
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
  uri: environment+"/trade/v1/refund",
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
     $api_type = '/trade/v1/refund';
     $syssn = '20200311066100020000977840';
     //$mchid = "MNxMp11FV35qQN"; //Only agents must provide this parameter
     $app_code = 'FF2FF74F2F2E42769A4A73*********'; //API credentials are provided by QFPay
     $app_key = '7BE791E0FD2E48E6926043B*********'; //API credentials are provided by QFPay
     $now_time = date("Y-m-d H:i:s"); //Get the currend date-time  
     
     $fields_string = '';
     $fields = array(
	  //'mchid' => urlencode($mchid),
    'syssn' => urlencode($syssn),
	  'out_trade_no' => urlencode(GetRandStr(20)),
	  'txamt' => urlencode(2200),
	  'txdtm' => $now_time
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

## 回應示例

```json
{
  "orig_syssn": "20191227000200020061752831", 
  "sysdtm": "2019-12-27 11:11:23", 
  "paydtm": "2019-12-27 11:11:26", 
  "txdtm": "2019-12-27 11:10:38", 
  "udid": "qiantai2", 
  "txcurrcd": "EUR", 
  "txamt": "10", 
  "resperr": "success", 
  "respmsg": "", 
  "out_trade_no": "RGNOEIVU9JZLNP9GGYXWXCW7OEMI720F", 
  "syssn": "20191227000300020061652643", 
  "respcd": "0000", 
  "chnlsn": "2019122722001411461404119764", 
  "cardcd": ""
}
```

## 注意事項

:::note
- 請確保退款金額不超過原始交易金額。
- 部分錢包不支援部分退款，請先確認。
- 各支付通道的退款有效期限不同，請聯絡 QFPay 支援團隊以取得更多資訊。
- 若退款結果為失敗（`respcd` 非 `0000`），建議實作重試邏輯，或透過 [交易查詢 API](/docs/common-api/transaction-enquiry) 驗證退款狀態。
:::