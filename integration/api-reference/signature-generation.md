---
id: "signature-generation"
title: "Signature Generation"
sidebar_label: "Signature Generation"
---

All API requests must include a digital signature to ensure authenticity and data integrity.\
Unless otherwise specified, the signature must be passed in the HTTP header:

```
X-QF-SIGN: <your_signature>
```

---

## Step-by-Step Guide

To generate a valid signature, follow these steps:

### Step 1: Sort parameters

Sort all request parameters by **parameter name**, in **ASCII ascending order**.

**Example parameters:**

| Parameter  | Value        |
| ---------- | ------------ |
| `mchid`    | `ZaMVg12345` |
| `txamt`    | `100`        |
| `txcurrcd` | `HKD`        |

Sorted result:

```
mchid=ZaMVg12345&txamt=100&txcurrcd=HKD
```

---

### Step 2: Append your client key

Append your secret `client_key` (issued by QFPay) to the end of the string.

If `client_key = abcd1234`, then:

```
mchid=ZaMVg12345&txamt=100&txcurrcd=HKDabcd1234
```

---

### Step 3: Hash the string

Hash the final string using one of the supported algorithms.\
**SHA256 is recommended**, but MD5 is also supported.

Example:

```
SHA256("mchid=ZaMVg12345&txamt=100&txcurrcd=HKDabcd1234")
```

---

### Step 4: Add to the request header

Include the hash result in the HTTP header:

```
X-QF-SIGN: <your_hashed_signature>
```

---

## Notes

- Do **not** insert any line breaks, tabs, or extra spaces when building the string.
- Parameter names and values are case-sensitive.
- If the signature is incorrect, double-check parameter order, encoding, and spacing.

For code instructions select Python, Java, Node.js or PHP with the tabs below.

<Tabs>
  <Tab title="Tab">
    ```python
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
    ```
  </Tab>
  <Tab title="Tab">
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
  </Tab>
  <Tab title="Tab">
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
  </Tab>
  <Tab title="Tab">
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
  </Tab>
</Tabs>

> The above command returns JSON structured like this:

```json
{
  "signature": "B3B251B202801388BE4AC8E5537B81B1"
}
```