---
id: wechat-pay-h5
title: WeChat H5 Payment (Third-Party Browser)
description: This guide helps merchants integrate WeChat H5 payment for non-WeChat browser environments, including scene configuration and redirect handling.
sidebar_label: WeChat H5 Payment
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# WeChat H5 Payment (Third-Party Browser)

<Link href="/img/wechat-h5.png" target="_blank">![WeChat H5 process-flow](@site/static/img/wechat-h5.png)</Link>

---

## HTTP Request

`POST /trade/v1/payment`
`pay_type: 800212` (WeChat H5 Payment)

---

## Request Parameters

| Field Name            | Field Code    | Required | Type   | Description                                                    |
| --------------------- | ------------- | -------- | ------ | -------------------------------------------------------------- |
| Common Payment Params | —             | Yes      | —      | See [Request Format](/docs/api-reference/request-format) |
| Extended Info         | `extend_info` | Yes      | Object | See below                                                      |

### `extend_info` Structure

```js
extend_info: {
  "scene_info": { // Scene info
    "h5_info": {
      "type": "Wap", // Scene type
      "wap_url": "https://qfpay.global/h5/pay", // WAP site URL
      "wap_name": "qfpay" // WAP site name
    }
  },
  "spbill_create_ip": "192.168.1.10" // User's real IP address
}
```

For IP acquisition details, see [WeChat Docs](https://pay.weixin.qq.com/wiki/doc/api/H5.php?chapter=15_5)

### Extended Info Parameters

| Field Code         | Subfield Code | Sub-subfield Code | Required | Type   | Description             |
| ------------------ | ------------- | ----------------- | -------- | ------ | ----------------------- |
| `scene_info`       |               |                   | Yes      | Object | —                       |
|                    | `h5_info`     |                   | Yes      | Object | —                       |
|                    |               | `type`            | Yes      | String | Fixed value `Wap`       |
|                    |               | `wap_url`         | Yes      | String | URL of the mobile site  |
|                    |               | `wap_name`        | Yes      | String | Name of the mobile site |
| `spbill_create_ip` |               |                   | Yes      | String | Customer's IP address   |

---

## Response Parameters

| Field Code             | Subfield Code | Type   | Field Name  | Description                                                    |
| ---------------------- | ------------- | ------ | ----------- | -------------------------------------------------------------- |
| Common Response Params | —             | —      | —           | See [Response Format](/docs/api-reference/response-format) |
| Payment URL            | `pay_url`     | String | Payment URL | URL to redirect user to complete payment                       |

### Example `pay_url`

```
https://wx.tenpay.com/cgi-bin/mmpayweb-bin/checkmweb?prepay_id=wx20161110163838f231619da20804912345&package=1037687096
```

### Appended with `redirect_url`

```
https://wx.tenpay.com/cgi-bin/mmpayweb-bin/checkmweb?prepay_id=wx20161110163838f231619da20804912345&package=1037687096&redirect_url=https%3A%2F%2Fwww.wechatpay.com.cn
```

---

## Additional Notes

* Applicable for App-embedded browsers, mobile websites, etc. **Not supported within the WeChat in-app browser**.
* The fields `wap_url` and `wap_name` must be correctly set, or WeChat may reject the payment request.
* Make sure the IP address submitted is the real IP of the user device, to avoid risk rejection by WeChat.
