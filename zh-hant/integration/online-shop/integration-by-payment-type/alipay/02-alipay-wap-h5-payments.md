---
id: alipay-wap-h5-payments
title: 支付寶 WAP / H5 支付
description: 本文檔涵蓋支付寶海外（801107）、支付寶香港（801512）、以及服務窗 H5（800107）的整合方式。
sidebar_label: 支付寶 WAP / H5 支付
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# 支付寶 WAP / H5 支付

## 適用場景

- 使用 **手機瀏覽器**（非 App）進行付款
- 適用於商戶的 Mobile Web、WebApp、或引導用戶於瀏覽器開啟付款連結
- 支援錢包包括：
  - 支付寶海外（`801107`）
  - 支付寶香港（`801512`）
  - 支付寶服務窗 H5（`800107`）

:::note
社交 App（如 WeChat、Facebook Messenger）常常無法跳轉至其他錢包 App，建議引導用戶在手機瀏覽器中開啟付款連結。
:::

---

## HTTP 請求

- **API 端點**：`/trade/v1/payment`
- **請求方法**：`POST`
- **支付編碼對應表**：

| 編碼   | 錢包名稱           | 描述                      |
|-----------|--------------------|---------------------------|
| `801107`  | 支付寶海外         | Web 或 WAP 跨境支付       |
| `801512`  | 支付寶香港         | 香港用戶 WAP 支付         |
| `800107`  | 支付寶服務窗 H5    | JSAPI + 授權碼機制支付    |

---

## 請求參數

請求參數格式請參考[公共支付請求參數](/docs/api-reference/request-format)，以下僅列出與支付寶 WAP / H5 相關的部分參數：


| 參數名稱       | 是否必填 | 描述                                       |
|----------------|----------|--------------------------------------------|
| `txamt`        | 是       | 交易金額（單位為分），建議大於 200        |
| `txcurrcd`     | 是       | 貨幣代碼，例如 HKD                         |
| `pay_type`     | 是       | 請參考上述 PayType 表                      |
| `out_trade_no` | 是       | 商戶自訂訂單編號，需唯一                   |
| `txdtm`        | 是       | 交易時間，格式：YYYY-MM-DD hh:mm:ss       |
| `return_url`   | 是       | 成功付款後跳轉用戶的頁面                  |
| `notify_url`   | 是       | 非同步通知商戶後端付款結果的接收端點        |
| `goods_name`   | 是       | 商品名稱（僅部分錢包強制）                 |
| `mchid`        | 是       | 商戶號，如由 QFPay 分配則為必填           |
| `openid`       | 視情況    | 僅適用於 `800107`，即服務窗 H5 授權碼     |
| `limit_pay`    | 否       | 僅適用於中國大陸支付場景                   |

---

## 請求範例

<Tabs>
<TabItem value="python" label="Python">

```python
#coding=utf8
import urllib.request, urllib.parse, urllib.error, urllib.request, urllib.error, urllib.parse, hashlib
import requests
import datetime
import string

# 輸入用戶端憑證
environment = 'https://test-openapi-hk.qfapi.com'
app_code = 'D5589D2A1F2E42A9A60C37*********'
client_key = '0E32A59A8B454940A2FF39**********'


# 建立資料請求所需的參數值
current_time = datetime.datetime.now().replace(microsecond=0)                                

print(current_time)

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
txamt = '10' # 以分為單位，建議金額大於 200 以避免風控攔截
txcurrcd = 'HKD'
pay_type = '801107' 
auth_code='283854702356157409'
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
// 輸入用戶端憑證
const environment = 'https://test-openapi-hk.qfapi.com'
const app_code = 'D5589D2A1F2E42A9A60C37*********'
const client_key = '0E32A59A8B454940A2FF39*********'

// 生成當前時間
var dateTime = new Date().toISOString().replace(/T/, ' ').replace(/\..+/, '')
console.log(dateTime)

// 建立資料請求所需的參數值
const key = client_key
var tradenumber = String(Math.round(Math.random() * 1000000000))
console.log(tradenumber)

// 請求內容主體
var payload = {
'txamt': '10', // 以分為單位，建議金額大於 200 以避免風控攔截
'txcurrcd': 'HKD',
'pay_type': '801107',
'out_trade_no': tradenumber,
'txdtm': dateTime,
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
$pay_type = '801107'; 
//$mchid = "MNxMp11FV35qQN"; // 只有渠道商需要提供
$app_code = 'FF2FF74F2F2E42769A4A73*********'; 
$app_key = '7BE791E0FD2E48E6926043B*********'; 
$now_time = date("Y-m-d H:i:s"); // 獲取當前時間

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

$header = array();
$header[] = 'X-QF-APPCODE: ' . $app_code;
$header[] = 'X-QF-SIGN: ' . $sign;

//POST 數據
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

## 響應參數
回應格式請參考 [ 公共支付響應參數](/docs/api-reference/response-format)。

## 響應範例

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

## 流程圖

<Link to="/img/alipay_h5_process.jpg" target="_blank">![Alipay H5 process-flow](@site/static/img/alipay_h5_process.jpg)</Link>

## 非同步通知說明

支付完成後，QFPay 將透過非同步通知（`notify_url`）發送交易結果。

- 通知格式請參考：[非同步通知 API 文檔](/docs/common-api/async-notifications)
- 建議使用 [交易查詢 API](/docs/common-api/transaction-enquiry) 進行最終結果確認

:::warning
請勿僅依賴前端跳轉結果，應以後端通知為準，並驗證簽名。
:::

## 安全注意事項

- 簽名計算請依照 [公共參數規則](/docs/api-reference/response-format#%E5%9B%9E%E6%87%89%E7%B0%BD%E5%90%8D%E9%A9%97%E8%AD%89)
- 請勿將密鑰寫死於前端代碼中
- 錯誤代碼請參考：[交易返回碼](/docs/api-reference/status-codes)

## 延伸參考

- [支付寶 H5 授權流程](https://docs.open.alipay.com/289/105656)
- [支付寶官方收銀台文檔](https://docs.open.alipay.com/common/105591)
