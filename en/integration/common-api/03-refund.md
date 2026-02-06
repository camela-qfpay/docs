---
id: refund
title: Refund API Guide
description: API guide for processing refunds through QFPay's system.
sidebar_label: Refund API
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Refund API Guide

:::tip
Only transactions with the return code `0000` (transaction successful) can be refunded.
:::

:::note
For credit card payments, QFPay performs **automatic capture**. If a refund is submitted on the **same day**, it will be treated as a **void** and handled by this same refund API.
:::

## API Endpoint

**Endpoint** : `/trade/v1/refund`

**Method** : `POST`

### Request Headers

```http
Content-Type: application/x-www-form-urlencoded
X-QF-APPCODE: <your-app-code>
X-QF-SIGN: <signature>
```

---

## Request Parameters

| Parameter         | Mandatory | Type        | Description |
|------------------|-----------|-------------|-------------|
| `syssn`          | Yes       | String(128) | QFPay transaction number of the **original** transaction to be refunded |
| `out_trade_no`   | Yes       | String(128) | Unique refund transaction ID (must not repeat across refund requests) |
| `txamt`          | Yes       | Int(11)     | Refund amount in cents (e.g. 100 = $1). Suggest > 200 to avoid risk control. |
| `txdtm`          | Yes       | String(20)  | Refund request time. Format: `YYYY-MM-DD hh:mm:ss` |
| `mchid`          | Conditional | String(16) | Merchant ID. Required only if one is issued. |
| `udid`           | No        | String(40)  | Unique transaction device ID |

---

## Response Parameters

| Parameter             | Type         | Description |
|-----------------------|--------------|-------------|
| `syssn`               | String(40)   | New refund transaction ID |
| `orig_syssn`          | String(128)  | Original transaction ID |
| `txamt`               | Int(11)      | Refunded amount in cents |
| `sysdtm`              | String(20)   | Refund system time (`YYYY-MM-DD hh:mm:ss`) |
| `respcd`              | String(4)    | Response code: <br/> `0000` = success <br/> `1143`, `1145` = processing <br/> other = failed |
| `resperr`             | String(128)  | Response message |
| `cash_fee`            | String       | Actual amount paid by user (after discounts) |
| `cash_fee_type`       | String       | Payment currency (e.g. CNY) |
| `cash_refund_fee`     | String       | Actual refunded amount |
| `cash_refund_fee_type`| String       | Refund currency (e.g. CNY) |

---

## Sample HTTP Body

```http
txamt=10&syssn=20191227000200020061752831&out_trade_no=12345678&txdtm=2019-12-27 10:39:39&mchid=ZaMVg*****
```

---

## SDK Code Examples

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

---

## Sample JSON Response

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

---

## Notes

- Ensure refund amount does not exceed the original transaction value.
- Some wallets may not support partial refunds.
- Refund time limits vary by channel. Contact QFPay support for details.
- For failed refunds (`respcd` not `0000`), retry logic or query via [Transaction Enquiry](/docs/common-api/transaction-enquiry) is advised.