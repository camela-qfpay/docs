---
id: transaction-enquiry
title: Transaction Enquiry
description: Query the status of payments, refunds, and cancellations via QFPay's transaction enquiry API.
sidebar_label: Transaction Enquiry
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Transaction Enquiry

## API Endpoint for Transaction Enquiry

After making a payment, refund or cancellation request, merchants can use the enquiry interface to obtain the transaction status.

This API allows querying by:

* `syssn` (QFPay transaction number)
* `out_trade_no` (merchant-side order number)
* Or with `start_time` / `end_time` for time-based filtering

If querying a refund transaction, the response will also return `origssn`, which refers to the original transaction's `syssn`.

## HTTP Request

* **Endpoint** : `/trade/v1/query`
* **Method** : `POST`

Request Headers:

```http
Content-Type: application/x-www-form-urlencoded
X-QF-APPCODE: <your-app-code>
X-QF-SIGN: <signature>
```

## Request Parameters

Full details are documented in [Common API Request Format](/docs/api-reference/request-format), but key parameters for transaction enquiry include:

| Parameter      | Type        | Required    | Description                                                      |
| -------------- | ----------- | ----------- | ---------------------------------------------------------------- |
| `mchid`        | String(16)  | Conditional | Required if assigned to the merchant. See warning above.         |
| `syssn`        | String(128) | No          | QFPay transaction number(s), comma-separated.                    |
| `out_trade_no` | String(128) | No          | Merchant platform transaction number(s), comma-separated.        |
| `pay_type`     | String(6)   | No          | Payment type(s), comma-separated.                                |
| `respcd`       | String(4)   | No          | Filter by specific return code (e.g. `0000`).                    |
| `start_time`   | String(20)  | No          | Format: `YYYY-MM-DD hh:mm:ss`. Required for cross-month queries. |
| `end_time`     | String(20)  | No          | Format: `YYYY-MM-DD hh:mm:ss`. Required for cross-month queries. |
| `page`         | Integer     | No          | Defaults to 1.                                                   |
| `page_size`    | Integer     | No          | Defaults to 10. Max is 100.                                      |

---

Request Body:

```http
mchid=<merchant-id>&syssn=<qfpay-tx-id>&start_time=2022-12-01 00:00:00&end_time=2022-12-01 23:59:59
```

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
mchid = 'ZaMVg*****' #(Agent ID, Merchant ID)
syssn = '20191227000200020061752831' #Search by transaction number only
out_trade_no = '2019122722001411461404119764' #Search by out_trade_no only
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
        String mchid="ZaMVg*****"; // Only Agents must provide the mchid

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
// Enter Client Credentials
const environment = 'https://test-openapi-hk.qfapi.com'
const app_code = 'D5589D2A1F2E42A9A60C37**********'
const client_key = '0E32A59A8B454940A2FF39**********'

// Generate Timestamp
var dateTime = new Date().toISOString().replace(/T/, ' ').replace(/\..+/, '')
console.log(dateTime)

// Body Payload
const key = client_key
var tradenumber = String(Math.round(Math.random() * 1000000000))
console.log(tradenumber)

var payload = {
'syssn': '20191231000300020063521806',
'start_time': '2019-12-27 00:00:00',
'end_time': '2019-12-31 23:59:59',
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
//$mchid = "MNxMp11FV35qQN"; //Only agents must provide this parameter
$app_code = 'FF2FF74F2F2E42769A4A73*********'; //API credentials provided by QFPay
$app_key = '7BE791E0FD2E48E6926043B*********'; //API credentials provided by QFPay
$now_time = date("Y-m-d H:i:s"); //Get the current date-time  

$fields_string = '';
$fields = array(
//'mchid' => urlencode($mchid),
'syssn' => urlencode($syssn),
//'out_trade_no' => urlencode($out_trade_no),
//'start_time' = '2020-03-01 00:00:00',
//'end_time' = '2020-03-04 23:59:59'
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

## Response Parameters

These fields are explained in [Common API Response Format](/docs/api-reference/response-format). The most relevant response fields for transaction enquiry include:

| Field                  | Type    | Description                                                                                         |
| ---------------------- | ------- | --------------------------------------------------------------------------------------------------- |
| `syssn`                | String  | QFPay transaction ID                                                                                |
| `out_trade_no`         | String  | Merchant order ID                                                                                   |
| `txamt`                | Integer | Transaction amount (in cents)                                                                       |
| `txcurrcd`             | String  | Currency code (e.g. HKD)                                                                            |
| `respcd`               | String  | Transaction result code. See [Status Codes](/docs/api-reference/status-codes).                      |
| `errmsg`               | String  | Result message                                                                                      |
| `order_type`           | String  | `payment` or `refund`                                                                               |
| `pay_type`             | String  | Payment channel used                                                                                |
| `cancel`               | String  | Cancellation/refund flag (0–5). See [Refund Guide](/docs/common-api/refund) for full explanation. |
| `cash_fee`             | String  | Actual paid amount (after discount)                                                                 |
| `cash_fee_type`        | String  | Actual payment currency (e.g. CNY)                                                                  |
| `cash_refund_fee`      | String  | Actual refund amount (if applicable)                                                                |
| `cash_refund_fee_type` | String  | Refund currency                                                                                     |
| `exchange_rate`        | String  | Applied exchange rate (if cross-currency)                                                           |
| `sysdtm`               | String  | QFPay system transaction time                                                                       |
| `txdtm`                | String  | Merchant transaction time                                                                           |
| `chnlsn`               | String  | Wallet-side transaction number                                                                      |
| `origssn`              | String  | Original transaction ID (for refunds only)                                                          |

---

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