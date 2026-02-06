---
id: alipay-web-payments
title: 支付寶 Web 支付
sidebar_label: 支付寶 Web 支付
description: 支付寶 Web 支付（香港與海外）整合指南
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# 支付寶線上支付（Web）

客戶可在商戶網站上使用支付寶完成交易。使用者掃描顯示的二維碼確認金額並付款。付款完成後，系統可透過 `return_url` 參數將使用者導回商戶指定頁面。支付寶以人民幣即時從用戶錢包扣除金額，QFPay 則以港幣或本地貨幣結算給商戶。

### HTTP 請求
**端口**:  
`POST ../trade/v1/payment`

## PayType 對應類型:

| 編碼   | 描述                                  |
|-----------|---------------------------------------|
| 801101    | 支付寶線上掃碼支付（海外商戶）        |
| 801501    | 支付寶線上掃碼支付（香港商戶）        |

---

## 請求參數（Request Parameters）

| 參數名稱       | 參數代碼       | 必填 | 資料型別     | 描述 |
|----------------|----------------|------|--------------|------|
| 訂單金額       | `txamt`        | 是   | Int(11)      | 以最小單位（如 100 = $1）計算。建議大於 200 以避免風控。 |
| 幣種           | `txcurrcd`     | 是   | String(3)    | 請參閱[幣種列表](/docs/preparation/paycode#支付幣種)。 |
| 支付方式       | `pay_type`     | 是   | String(6)    | 支付寶 Web 支付：801101。 |
| 外部訂單編號   | `out_trade_no` | 是   | String(128)  | 商戶自訂交易編號，於同一商戶帳戶中必須唯一。 |
| 交易時間       | `txdtm`        | 是   | String(20)   | 時間格式：`YYYY-MM-DD hh:mm:ss` |
| 過期時間       | `expired_time` | 否（正掃限定） | String(3) | 單位分鐘，預設 30 分鐘，允許範圍為 5–120 分鐘。<br/>適用於：800201（微信掃碼） |
| 商品名稱       | `goods_name`   | 否   | String(64)   | 最多 20 個字元（含中英文及數字）。若為中文需使用 UTF-8 編碼。 |
| 子商戶編號     | `mchid`        | 否   | String(16)   | 若提供則為必填，否則請勿傳入。由 QFPay 分配。 |
| 設備唯一 ID    | `udid`         | 否   | String(40)   | 裝置代碼，將顯示於商戶後台。 |
| 成功跳轉網址   | `return_url`   | 否   | String(512)  | 支付完成後導向的 URL。 |

---

## 回應參數（Response Parameters）

| 參數名稱       | 參數代碼       | 資料型別   | 描述 |
|----------------|----------------|------------|------|
| 支付方式       | `pay_type`     | String(6)  | 回傳支付方式，例如：801101 表示 Web 支付。 |
| 系統時間       | `sysdtm`       | String(20) | 格式：`YYYY-MM-DD hh:mm:ss`，作為結算截止依據。 |
| 交易時間       | `txdtm`        | String(20) | 發送請求時的時間。 |
| 錯誤訊息       | `resperr`      | String(128) | 如有錯誤則此欄回傳描述。 |
| 交易金額       | `txamt`        | Int(11)    | 付款金額（單位為最小單位）。 |
| 附加訊息       | `respmsg`      | String(128) | 其他訊息（可用於除錯）。 |
| 外部訂單編號   | `out_trade_no` | String(128) | 商戶自訂交易編號。 |
| QFPay 訂單號   | `syssn`        | String(40) | QFPay 系統交易編號。 |
| 回應碼         | `respcd`       | String(4)  | `0000` 表成功；<br/>`1143` 或 `1145` 表示仍處理中；<br/>其他為失敗。詳見[交易狀態碼](/docs/preparation/paycode#交易狀態碼)。 |
| 支付連結       | `pay_url`      | String(512) | 用戶端用於生成 QR Code 的支付連結。 |

---

## 程式碼範例

```plaintext
請從下方選擇語言查看示例程式碼：Python、Java、Node.js 或 PHP。
```

<Tabs>
<TabItem value="python" label="Python">

```python
#coding=utf8
import urllib.request, urllib.parse, urllib.error, urllib.request, urllib.error, urllib.parse, hashlib
import requests
import datetime
import string

# API 憑證
environment = 'https://test-openapi-hk.qfapi.com'
app_code = 'D5589D2A1F2E42A9A60C37*********'
client_key = '0E32A59A8B454940A2FF39**********'


# 當前時間
current_time = datetime.datetime.now().replace(microsecond=0)                                

print(current_time)

# 建立簽名函數
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


# 請求參數
txamt = '10'
txcurrcd = 'HKD'
pay_type = '801101'
auth_code='283854702356157409' #CPM only
out_trade_no = '01234567890123'
txdtm = current_time
goods_name = 'test1'   
mchid = 'ZaMVg*****'
key = client_key


#data ={'txamt': txamt, 'txcurrcd': txcurrcd, 'pay_type': pay_type, 'out_trade_no': out_trade_no, 'txdtm': txdtm, 'goods_name': goods_name, 'mchid': mchid}
data ={'txamt': txamt, 'txcurrcd': txcurrcd, 'pay_type': pay_type, 'out_trade_no': out_trade_no, 'txdtm': txdtm, 'mchid': mchid}

# 發送 POST 請求
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

        String pay_type="801101";
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
<TabItem value="javascript" label="Javascript">

```javascript
// API 憑證
const environment = 'https://test-openapi-hk.qfapi.com'
const app_code = 'D5589D2A1F2E42A9A60C37*********'
const client_key = '0E32A59A8B454940A2FF39*********'

// 當前時間
var dateTime = new Date().toISOString().replace(/T/, ' ').replace(/\..+/, '')
console.log(dateTime)

// 請求參數
const key = client_key
var tradenumber = String(Math.round(Math.random() * 1000000000))
console.log(tradenumber)

var payload = {
'txamt': '10', 
'txcurrcd': 'HKD',
'pay_type': '801101',
'out_trade_no': tradenumber,
'txdtm': dateTime,
'mchid': 'ZaMVg*****'
};

// 建立簽名
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


// 發送 POST 請求
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
     $pay_type = '801101';
     //$mchid = "MNxMp11FV35qQN"; 
     $app_code = 'FF2FF74F2F2E42769A4A73*********'; 
     $app_key = '7BE791E0FD2E48E6926043B*********';
     $now_time = date("Y-m-d H:i:s");
     
     $fields_string = '';
     $fields = array(
      //'mchid' => urlencode($mchid),
      'pay_type' => urlencode($pay_type),
      'out_trade_no' => urlencode(GetRandStr(20)),
      'txcurrcd' => urlencode('HKD'),
      'txamt' => urlencode(2200),
      'txdtm' => $now_time
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

---

## JSON 回應範例

```json
{
  "sysdtm": "2020-04-13 10:30:34", 
  "paydtm": "2020-04-13 10:30:34", 
  "txcurrcd": "HKD", 
  "respmsg": "", 
  "pay_type": "801101", 
  "cardcd": "", 
  "udid": "qiantai2", 
  "txdtm": "2020-04-13 10:30:34", 
  "txamt": "300", 
  "resperr": "success", 
  "out_trade_no": "4K35N374II7UJJ8RGIAE45O2CVHGHFF0", 
  "syssn": "20200413000300020087033882", 
  "respcd": "0000", 
  "pay_url": "https://globalmapi.alipay.com/gateway.do?total_fee=3.0&secondary_merchant_name=###merchant_name###&out_trade_no=20200413000300020087033882&secondary_merchant_industry=7011&service=create_forex_trade&_input_charset=UTF-8&sign=02beb99974ce6167666280b9727c4444&currency=THB&notify_url=https%3A%2F%2Fo2-hk.qfapi.com%2Fonline-test%2Ftrade%2Falipay%2Fv1%2Fonline_notify&order_valid_time=1800&secondary_merchant_id=2565075&sign_type=MD5&partner=2088631377368888&product_code=NEW_OVERSEAS_SELLER&order_gmt_create=2020-04-13+10%3A30%3A34&return_url=&subject=###merchant_name###", 
  "chnlsn": ""
}
```

## 備註

:::tip
請將 QR Code 或 `iframe` 指向回應中的 `pay_url`。
:::

:::warning
請勿重複使用 `out_trade_no`。
:::

:::info
若回傳 `respcd` = `1143`/`1145`，請使用 `/trade/v1/query` API 查詢最終交易結果。
:::