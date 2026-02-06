---
id: signature-generation
title: 簽名生成方式
sidebar_label: 簽名生成
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

## 簽名生成

所有 API 請求都必須包含數位簽名以確保資料真實性與完整性。  
除非另有說明，簽名應放入 HTTP Header 中：

```
X-QF-SIGN: <你的簽名>
```
---

## 簽名生成步驟

請依照以下步驟產生正確簽名：

### 步驟一：排序參數

將所有請求參數依「參數名稱的 ASCII 升冪順序」排序。

**範例參數：**

| 參數名稱     | 值            |
|--------------|---------------|
| `mchid`      | `ZaMVg12345`  |
| `txamt`      | `100`         |
| `txcurrcd`   | `HKD`         |

排序結果為：
```
mchid=ZaMVg12345&txamt=100&txcurrcd=HKD
```
---

---

### 步驟二：接上 client key

於排序後的字串末尾，附加商戶的私鑰 `client_key`（由 QFPay 提供）。

若 `client_key = abcd1234`，則：
```
mchid=ZaMVg12345&txamt=100&txcurrcd=HKDabcd1234
```
---

### 步驟三：進行雜湊處理

使用支援的演算法將字串進行雜湊。  
**建議使用 SHA256**，亦支援 MD5。

範例：
```
SHA256(“mchid=ZaMVg12345&txamt=100&txcurrcd=HKDabcd1234”)
```
---

### 步驟四：加入請求標頭

將雜湊結果加到請求 Header：
```
X-QF-SIGN: <你的雜湊結果>
```

---

## 備註

- 建立簽名字串時請勿加入換行符、縮排或額外空格。
- 參數名稱與參數值皆區分大小寫。
- 若簽名驗證失敗，請檢查參數排序、編碼與格式。

請使用下方標籤切換語言以參考 Python、Java、Node.js 或 PHP 的範例程式碼。


<Tabs groupId="signature-generation">
<TabItem value="python" label="Python">

```python
# 簽名生成
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
```

</TabItem>
<TabItem value="java" label="Java">

```java
public class QFPayUtils {
    public static String getMd5Value(String input) {
        try {
            java.security.MessageDigest md = java.security.MessageDigest.getInstance("MD5");
            byte[] array = md.digest(input.getBytes("UTF-8"));
            StringBuffer sb = new StringBuffer();
            for (int i = 0; i < array.length; i++) {
                sb.append(String.format("%02x", array[i]));
            }
            return sb.toString().toUpperCase();
        } catch (Exception e) {
            return null;
        }
    }
}
```
</TabItem>
<TabItem value="javascript" label="Javascript">

```javascript
const crypto = require("crypto");

const payload = {
  txamt: "10",
  txcurrcd: "HKD",
  pay_type: "800101",
  out_trade_no: "ORDER12345",
  txdtm: "2025-11-17 18:00:00",
  mchid: "ZaMVg*****",
};

const key = "client_key_here";
const ordered = Object.keys(payload).sort().map(k => `${k}=${payload[k]}`).join("&");
const signString = ordered + key;
const signature = crypto.createHash("md5").update(signString).digest("hex").toUpperCase();
console.log(signature);
```
</TabItem>
<TabItem value="php" label="PHP">

```php
<?php
function generateSignature($fields, $key) {
    ksort($fields);
    $str = '';
    foreach ($fields as $k => $v) {
        $str .= $k . '=' . $v . '&';
    }
    $str = rtrim($str, '&') . $key;
    return strtoupper(md5($str));
}

$fields = array(
    'pay_type' => '800101',
    'out_trade_no' => 'ORDER12345',
    'txcurrcd' => 'HKD',
    'txamt' => '2200',
    'txdtm' => '2025-11-17 18:00:00',
    'mchid' => 'ZaMVg*****'
);

$signature = generateSignature($fields, 'client_key_here');
echo $signature;
?>
```
</TabItem>
</Tabs>

> 上述指令會回傳如下結構的 JSON：

```json
{
  "signature": "B3B251B202801388BE4AC8E5537B81B1"
}
```