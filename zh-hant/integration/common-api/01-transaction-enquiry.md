---
id: transaction-enquiry
title: 交易查詢
description: 使用 QFPay 提供的查詢接口以獲取交易（付款、退款、取消）的處理狀態
sidebar_label: 交易查詢
---

import Tabs from '@theme/Tabs'; 
import TabItem from '@theme/TabItem';

# 交易查詢

## 查詢接口端點

當商戶發起付款、退款或取消交易後，可使用本查詢接口獲取該交易的處理狀態。

支援的查詢方式：

* 透過 QFPay 訂單號 `syssn`
* 透過商戶自訂訂單號 `out_trade_no`
* 或者以時間區間 `start_time` / `end_time` 查詢

若查詢的是退款交易，回應中會額外返回欄位 `origssn`，對應原始交易的 `syssn`。

## HTTP 請求

* **端點**：`/trade/v1/query`
* **方法**：`POST`

Header：

```http
Content-Type: application/x-www-form-urlencoded
X-QF-APPCODE: <your-app-code>
X-QF-SIGN: <signature>
```

## 請求參數

完整格式請參見 [通用 API 請求格式](/docs/api-reference/request-format)，以下為交易查詢相關的主要欄位：

| 參數             | 類型          | 是否必填 | 說明                                  |
| -------------- | ----------- | ---- | ----------------------------------- |
| `mchid`        | String(16)  | 視情況  | 若系統有配置商戶編號則必填，否則不可填寫                |
| `syssn`        | String(128) | 否    | QFPay 訂單號，可為多筆，以逗號分隔                |
| `out_trade_no` | String(128) | 否    | 商戶訂單號，可為多筆，以逗號分隔                    |
| `pay_type`     | String(6)   | 否    | 支付類型，可多筆，用逗號分隔                      |
| `respcd`       | String(4)   | 否    | 指定回傳狀態碼（如：0000）                     |
| `start_time`   | String(20)  | 否    | 開始時間。格式：YYYY-MM-DD hh:mm:ss。跨月份查詢必填 |
| `end_time`     | String(20)  | 否    | 結束時間。格式：YYYY-MM-DD hh:mm:ss。跨月份查詢必填 |
| `page`         | Integer     | 否    | 預設為 1                               |
| `page_size`    | Integer     | 否    | 預設為 10，最大值為 100                     |



<Tabs>
<TabItem value="python" label="Python">

```python
import urllib.request, urllib.parse, urllib.error, urllib.request, urllib.error, urllib.parse, hashlib
import requests
from hashids import Hashids
import datetime
import string
import random

# 輸入用戶端憑證
environment = 'https://test-openapi-hk.qfapi.com'
app_code = 'D5589D2A1F2E42A9A60C37**********'
client_key = '0E32A59A8B454940A2FF39**********'

# 建立資料請求所需的參數值
current_time = datetime.datetime.now().replace(microsecond=0)         
random_string = ''.join(random.choices(string.ascii_uppercase + string.digits, k=32))                       


# 產生簽名
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


# 請求內容主體
mchid = 'ZaMVg*****' # 只適用於渠道商
syssn = '20191227000200020061752831' # 使用QFPay內部訂單號查詢
out_trade_no = '2019122722001411461404119764' # 使用商戶訂單號查詢
start_time = '2019-12-27 00:00:00'
end_time = '2019-12-27 23:59:59'
key = client_key


#data ={'mchid': mchid, 'syssn': syssn, 'out_trade_no': out_trade_no, 'start_time': start_time, 'end_time': end_time}
data ={'mchid': mchid, 'syssn': syssn}

r = requests.post(environment+"/trade/v1/query",data=data,headers={'X-QF-APPCODE':app_code,'X-QF-SIGN':make_req_sign(data, key)})

print(make_req_sign(data, key))  
print(r.json())
```

</TabItem>
<TabItem value="java" label="Java">

```java
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;


public class Enquiry {
    public static void main(String args[]){
        String appcode="D5589D2A1F2E42A9A60C37**********";
        String key="0E32A59A8B454940A2FF39*********";
        String mchid="ZaMVg*****"; // 只適用於渠道商

        SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String date=df.format(new Date());
        String txdtm=date;

        String syssn="20191227000300020061662295";
        String start_time = "2019-12-27 00:00:00";
        String end_time = "2019-12-27 23:59:59";

        Map<String, String> unsortMap = new HashMap<>();
        unsortMap.put("mchid", mchid);
        unsortMap.put("syssn", syssn);

        String data=QFPayUtils.getDataString(unsortMap);
        System.out.println("Data:\n"+data+key);
        String md5Sum=QFPayUtils.getMd5Value(data+key);
        System.out.println("Md5 Value:\n"+md5Sum);

        String url="https://test-openapi-hk.qfapi.com";
        String resp= Requests.sendPostRequest(url+"/trade/v1/query", data, appcode,key);
        System.out.println(resp);
    }
}
```

</TabItem>
<TabItem value="javascript" label="Javascript">

```javascript
// 輸入用戶端憑證
const environment = 'https://test-openapi-hk.qfapi.com'
const app_code = 'D5589D2A1F2E42A9A60C37**********'
const client_key = '0E32A59A8B454940A2FF39**********'

// 生成當前時間
var dateTime = new Date().toISOString().replace(/T/, ' ').replace(/\..+/, '')
console.log(dateTime)

// 請求內容主體
const key = client_key
var tradenumber = String(Math.round(Math.random() * 1000000000))
console.log(tradenumber)

var payload = {
'syssn': '20191231000300020063521806',
'start_time': '2019-12-27 00:00:00',
'end_time': '2019-12-31 23:59:59',
'mchid': 'ZaMVg*****'
};

// 產生簽名
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


// API 請求
var request = require("request");
request({
  uri: environment+"/trade/v1/query",
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
$api_type = '/trade/v1/query';
$syssn = '20200311066100020000977841';
//$out_trade_no = 'zCvo0IqTg0SaQkGnHd6w';
//$mchid = "MNxMp11FV35qQN"; // 只適用於渠道商
$app_code = 'FF2FF74F2F2E42769A4A73*********'; 
$app_key = '7BE791E0FD2E48E6926043B*********'; 
$now_time = date("Y-m-d H:i:s"); // 獲取目前時間

$fields_string = '';
$fields = array(
//'mchid' => urlencode($mchid),
'syssn' => urlencode($syssn),
//'out_trade_no' => urlencode($out_trade_no),
//'start_time' = '2020-03-01 00:00:00',
//'end_time' = '2020-03-04 23:59:59'
);
ksort($fields);
print_r($fields);

foreach($fields as $key=>$value) { 
	$fields_string .= $key.'='.$value.'&' ;
}
$fields_string = substr($fields_string , 0 , strlen($fields_string) - 1); 

$sign = strtoupper(md5($fields_string . $app_key));

$header = array();
$header[] = 'X-QF-APPCODE: ' . $app_code;
$header[] = 'X-QF-SIGN: ' . $sign;

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

## 回應參數

詳細欄位請見 [通用 API 回應格式](/docs/api-reference/response-format)，以下為交易查詢常見欄位：

| 欄位                     | 類型      | 說明                                                        |
| ---------------------- | ------- | --------------------------------------------------------- |
| `syssn`                | String  | QFPay 訂單號                                                 |
| `out_trade_no`         | String  | 商戶訂單號                                                     |
| `txamt`                | Integer | 訂單金額（單位為分）                                                |
| `txcurrcd`             | String  | 幣種代碼，如 HKD，詳見 [幣別表](/docs/preparation/paycode#currencies) |
| `respcd`               | String  | 交易結果代碼，詳見 [狀態碼](/docs/api-reference/status-codes)         |
| `errmsg`               | String  | 交易結果說明                                                    |
| `order_type`           | String  | 訂單類型：`payment` 或 `refund`                                 |
| `pay_type`             | String  | 支付通道代碼                                                    |
| `cancel`               | String  | 撤銷/退款標記。詳見 [退款說明](/docs/common-api/refund)              |
| `cash_fee`             | String  | 使用者實際付款金額（扣除折扣後）                                          |
| `cash_fee_type`        | String  | 實際支付幣別，如 CNY                                              |
| `cash_refund_fee`      | String  | 實際退款金額                                                    |
| `cash_refund_fee_type` | String  | 退款幣別                                                      |
| `exchange_rate`        | String  | 若為跨幣種交易，返回匯率                                              |
| `sysdtm`               | String  | QFPay 系統交易時間                                              |
| `txdtm`                | String  | 商戶請求交易時間                                                  |
| `chnlsn`               | String  | 錢包/通道交易號                                                  |
| `origssn`              | String  | 原始交易號（僅退款時提供）                                             |

```json
{
  "respcd": "0000",
  "resperr": "Request successful",
  "data": [
    {
      "syssn": "20230423000200020088888888",
      "out_trade_no": "YOUR_ORDER_001",
      "txamt": "100",
      "txcurrcd": "HKD",
      "respcd": "0000",
      "errmsg": "success",
      "pay_type": "801107",
      "order_type": "payment",
      "txdtm": "2023-04-23 12:00:00",
      "sysdtm": "2023-04-23 12:00:03",
      "cancel": "0",
      "cash_fee": "100",
      "cash_fee_type": "HKD"
    }
  ]
}
```
