---
id: reversal
title: Reversal API Guide
description: Use the Reversal API to void a transaction that has not yet been completed. If the transaction has already succeeded, please use the Refund API instead.
sidebar_label: Reversal API
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Reversal API Guide

This page explains how to use the **Reversal API** to cancel a transaction that has **not yet been successfully completed**. Please note that a reversal is **not** a refund.

:::warning
A **reversal** is not a **refund**. It can only be initiated if the **original transaction has not been completed successfully**.
:::

## Supported Scenarios

:::note
The `/trade/v1/reversal` API is currently only supported in the following scenarios. It applies to transactions in a non-completed state (e.g. scanned but not paid).
:::

### Supported payment flows and PayTypes

Alipay MPM (Merchant Presented Mode)
- `800101`：Alipay Merchant Presented QR Code Payment in store (MPM) (Overseas Merchants)
- `801501`：Alipay Merchant Presented QR Code (MPM) Payment (HK Merchants)

WeChat Pay MPM (Merchant Presented Mode)
- `800201`：WeChat Merchant Presented QR Code Payment (MPM) (Overseas & HK Merchants)

Alipay CPM (Consumer Presented Mode)
- `800108`：Alipay Consumer Presented QR Code Payment (CPM) (Overseas & HK Merchants)

> For cancellations involving other wallets, please refer to the [Close API](#reversal-vs-close) or contact QFPay Support for integration guidance.

## API Endpoint

* **Endpoint**: `/trade/v1/reversal`
* **Method**: `POST`

A successful reversal will return `respcd=0000`.

If the transaction has already completed successfully (`respcd=0000` in payment), then it **cannot** be reversed. Instead, refer to the [Refund API](/docs/common-api/refund).

---

## Request Parameters

| Parameter      | Type        | Required | Description                                                       |
| -------------- | ----------- | -------- | ----------------------------------------------------------------- |
| `mchid`        | String(16)  | No       | QFPay Merchant ID. Required only for agents.                      |
| `syssn`        | String(40)  | Yes*     | QFPay transaction number                                          |
| `out_trade_no` | String(128) | Yes*     | Merchant transaction number                                       |
| `txamt`        | Int(11)     | Yes      | Transaction amount in cents (e.g. 100 = $1). Suggest value > 200. |
| `txdtm`        | String(20)  | Yes      | Original transaction time. Format: `YYYY-MM-DD hh:mm:ss`          |
| `udid`         | String(40)  | No       | Unique terminal ID (used for traceability)                        |

> *Either `syssn` or `out_trade_no` must be provided.

---

## Response Parameters

| Parameter    | Type      | Description                                                           |
| ------------ | --------- | --------------------------------------------------------------------- |
| `syssn`      | String    | New QFPay transaction number for the reversal                         |
| `orig_syssn` | String    | QFPay transaction number of the original (to-be-reversed) transaction |
| `txamt`      | Int       | Reversed amount (in cents)                                            |
| `txcurrcd`   | String(3) | Currency code, e.g. HKD                                               |
| `txdtm`      | String    | Transaction time                                                      |
| `sysdtm`     | String    | QFPay system time of the reversal                                     |
| `chnlsn`     | String    | Payment channel transaction number (may be empty if not processed)    |
| `respcd`     | String(4) | Response code (`0000` = success, others = failure or in progress)     |
| `resperr`    | String    | Result message                                                        |
| `respmsg`    | String    | Additional description (if any)                                       |

---

## Code Examples

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
<TabItem value="javascript" label="JavaScript">

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
<TabItem value="php" label="PHP">

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

## Sample Response

```json
{
  "orig_syssn": "20200305066100020000977813",
  "syssn": "20200305066100020000977814",
  "txamt": "2500",
  "txcurrcd": "EUR",
  "txdtm": "2020-03-05 16:50:30",
  "sysdtm": "2020-03-05 16:54:38",
  "chnlsn": "",
  "respcd": "0000",
  "resperr": "success",
  "respmsg": ""
}
```

---

## Additional Notes

* A reversal **does not guarantee** that the user has not been charged. Always verify using [Transaction Enquiry](/docs/common-api/transaction-enquiry).
* Reversal is intended for **real-time failures** only. Do not use it to refund successful transactions.
* If `respcd=1143` or `1145`, the reversal is in progress. You should poll the [Transaction Enquiry](/docs/common-api/transaction-enquiry) endpoint until the status is confirmed.

---


> The above command returns JSON structured like this:

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

## Reversal vs Close

Some wallets (e.g. WeChat Pay MPM) support the `/trade/v1/close` endpoint instead of reversal.

### Supported payment flows and PayTypes

WeChat Pay CPM (Consumer Presented Mode)
- `800008`：Consumer Present QR Code Mode (CPM) for WeChat
- `800208`：WeChat Consumer Presented QR Code Payment (CPM) (Overseas & HK Merchants)
- `801008`：WeChat Pay HK Consumer Presented QR Code Payment (CPM) (Direct Settlement, HK Merchants)

**Method** : `GET`

*If you would like to use this endpoint on a wallet other than Alipay & Wechat Pay please contact us for instructions.

### Request Parameters

Attribute | Mandatory | Type | Description  
--------- | ------- | --------- | -------
`mchid` | No | String(16) | Merchant ID allocated by QFPay
`syssn` | Yes* | String(40) | QFPay transaction number, returned by the system once payment is completed
`out_trade_no` | Yes* | String(128) | External transaction number
`txamt` | Yes | Int(11) | Amount of the transaction. Unit in cents (i.e. 100 = $1). Suggest value > 200 to avoid risk control.
`txdtm` | Yes | String(20) | Transaction time format: YYYY-MM-DD hh:mm:ss
`udid` | No | String(40) | Unique transaction device ID. Is displayed on the merchant portal.

*Either the `syssn` or `out_trade_no` must be provided.

### Response Parameters

Attribute | Type | Description  
--------- | --------- | -------
`orig_syssn` | String(40) | Refers to the original QFPay transaction number
`syssn` | String(40) | QFPay transaction number of the cancel/ reversal
`out_trade_no` | String(128) | External transaction number
`txamt` | Int(11) | Amount of the transaction. Unit in cents (i.e. 100 = $1)
`txcurrcd` | String(3) | Transaction currency. View the [Currencies](/docs/preparation/paycode#currencies) table for a complete list of available currencies
`txdtm` | String(20) | Transaction time. Format: YYYY-MM-DD hh:mm:ss
`sysdtm` | String(20) | System transaction time. Format: YYYY-MM-DD hh:mm:ss <br/> This parameter value is used as the cut-off time for settlements.
`chnlsn` | String | Transaction number from payment channel (wallet side)
`respcd` | String(4) | Response code <br/> 0000 = Reversal/ cancel successul <br/> 1143/1145 = Reversal/ cancel in progress <br/> others = Reversal/ cancel failed
`resperr` | String(128) | Result description
`respmsg` | String(128) | Information description
