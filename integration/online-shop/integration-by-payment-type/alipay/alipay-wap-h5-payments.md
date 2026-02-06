---
title: Alipay WAP / H5 Payments
description: This guide covers integration for Alipay Cross-Border (801107), AlipayHK (801512), and Alipay Service Window H5 (800107) payment types.
---

# Alipay WAP / H5 Payments

## Use Cases

- Payment initiated via **mobile browsers** (not inside apps)
- Suitable for mobile websites, WebApps, or cases where merchants guide users to open links in mobile browsers
- Supported wallets:
  - Alipay Cross-Border (`801107`)
  - AlipayHK (`801512`)
  - Alipay Service Window H5 (`800107`)

<Note>
Social apps like WeChat or Facebook Messenger often block redirection to external wallet apps. It's recommended to guide users to open payment links in Chrome, Safari, or other mobile browsers.
</Note>

---

## HTTP Request

- **Endpoint**: `/trade/v1/payment`
- **Method**: `POST`
- **PayType Reference Table**:

| PayType   | Wallet Name         | Description                       |
|-----------|---------------------|-----------------------------------|
| `801107`  | Alipay Cross-Border | Web or WAP international payment  |
| `801512`  | AlipayHK            | Alipay Hong Kong WAP payments     |
| `800107`  | Alipay H5 (JSAPI)   | Requires user authorization code  |

---

## Request Parameters

For the full list of shared request parameters, see [Public Request Parameters](/docs/api-reference/request-format). Below are fields specific to Alipay WAP / H5 payments:

| Parameter       | Required | Description                                          |
|-----------------|----------|------------------------------------------------------|
| `txamt`         | Yes      | Transaction amount in cents (e.g. 100 = $1). Recommended to be > 200 |
| `txcurrcd`      | Yes      | Currency code (e.g., HKD)                            |
| `pay_type`      | Yes      | See PayType table above                              |
| `out_trade_no`  | Yes      | Unique transaction ID from merchant system           |
| `txdtm`         | Yes      | Transaction timestamp in `YYYY-MM-DD hh:mm:ss` format |
| `return_url`    | Yes      | URL to redirect users after successful payment       |
| `notify_url`    | Yes      | Endpoint to receive asynchronous payment result      |
| `goods_name`    | Yes      | Product name (required by some wallets)              |
| `mchid`         | Yes      | QFPay merchant ID (required if assigned)             |
| `openid`        | Conditional | Only required for `800107` (Alipay H5 / Service Window) |
| `limit_pay`     | No       | Required only for Mainland China restrictions        |

---

## Sample Request  

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
pay_type = '801107' # Alipay Wap Payment = 801107
auth_code='283854702356157409' #CPM only
out_trade_no = '01234567890123'
txdtm = current_time
goods_name = 'test1'   
mchid = 'ZaMVg*****'
key = client_key


#data ={'txamt': txamt, 'txcurrcd': txcurrcd, 'pay_type': pay_type, 'out_trade_no': out_trade_no, 'txdtm': txdtm, 'goods_name': goods_name, 'mchid': mchid}
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

        String pay_type="801107";
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
'pay_type': '801107', // Alipay Wap Payment = 801107
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
$pay_type = '801107'; //Alipay Wap Payment = 801107
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

## Response Parameters

Refer to [Public Response Parameters](/docs/api-reference/response-format) for shared fields.

## Sample Response  

```json
{
  "sysdtm": "2020-04-13 11:32:03", 
  "paydtm": "2020-04-13 11:32:03", 
  "txcurrcd": "HKD", 
  "respmsg": "", 
  "pay_type": "801107", 
  "cardcd": "", 
  "udid": "qiantai2", 
  "txdtm": "2020-04-13 11:32:03", 
  "txamt": "300", 
  "resperr": "success", 
  "out_trade_no": "BUFB3PT9ZDUWEUAE4ATD21JKNHVEIIPV", 
  "syssn": "20200413000200020087171988", 
  "respcd": "0000", 
  "pay_url": "https://globalmapi.alipay.com/gateway.do?total_fee=3.0&secondary_merchant_name=###merchant_name###&out_trade_no=20200413000200020087171988&secondary_merchant_industry=7011&service=create_forex_trade_wap&_input_charset=UTF-8&sign=f16ef36efbb55058d1c1d36fef89bcf8&currency=THB&timeout_rule=30m&notify_url=https%3A%2F%2Fo2-hk.qfapi.com%2Fonline-test%2Ftrade%2Falipay%2Fv1%2Fonline_notify&secondary_merchant_id=2565075&sign_type=MD5&partner=2088631377368888&product_code=NEW_WAP_OVERSEAS_SELLER&return_url=&subject=###merchant_name###", 
  "chnlsn": ""
}
```
---

## Sequence Diagram  

[![Alipay H5 process-flow](https://qfpay-8e347952.mintlify.app/img/alipay-in-app.png)](/img/alipay_h5_process.jpg)

---

## Asynchronous Notification

After payment is completed, QFPay will notify your backend via the `notify_url`.

- See format and signature verification in [Asynchronous Notification](/docs/common-api/async-notifications)
- Use [Transaction Enquiry](/docs/common-api/transaction-enquiry) API to verify final status

:::warning
Never rely solely on frontend redirection. Always validate with backend notification and signature verification.
:::

---

## Security Considerations

- Signatures must follow [Signature Verification Rules](/docs/api-reference/response-format#signature-verification)
- Never expose credentials or secret keys in frontend code
- For error codes, refer to [Transaction Response Codes](/docs/api-reference/status-codes)

---

## Additional References

- [Alipay H5 Authorization Flow (Official)](https://docs.open.alipay.com/289/105656)
- [Alipay Cashier Documentation (Official)](https://docs.open.alipay.com/common/105591)