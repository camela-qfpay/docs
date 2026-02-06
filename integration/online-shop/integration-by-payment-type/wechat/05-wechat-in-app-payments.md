---
id: wechat-in-app-payments
title: WeChat In-App Payments (Native App)
description: This document explains how to integrate WeChat In-App payments using the official SDK within iOS or Android apps.
sidebar_label: WeChat In-App Payment
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# WeChat In-App Payments (Native App)

<Link href="/img/wechat-in-app.png" target="_blank">![WeChat App Payment Flow](@site/static/img/wechat-in-app.png)</Link>

WeChat In-App Payment is designed for **native mobile apps (iOS / Android)**. This method allows users to complete payments directly within the app using the official WeChat SDK.

---

## Prerequisites

To use WeChat In-App payments, merchants must:

* Register an account on the **WeChat Open Platform**
* Create an App and obtain the corresponding **AppID**
* Complete all WeChat approval processes

More details: [WeChat Official In-App Payment Guide](https://pay.weixin.qq.com/wiki/doc/api/wxpay/en/pay/In-AppPay/chapter6_2.shtml#menu1)

---

## Real-name Verification (Optional)

Merchants may choose to enable **real-name verification**.

* Applies to Mainland China citizens only
* The user's WeChat Wallet (e.g., linked bank card) must match the submitted identity information
* Users can still pay without a linked bank card
* This feature depends on the merchant account and PayType support

---

## SDK Downloads

You can download the official SDK from the [WeChat SDK Download Page](https://developers.weixin.qq.com/doc/oplatform/Downloads/iOS_Resource.html)

---

## API Request

### HTTP Request

* **Method**: `POST`
* **Endpoint**: `/trade/v1/payment`
* **PayType**: `800210`

### Request Parameters

| Field Name        | Param Code     | Required | Type      | Description                                                        |
| ----------------- | -------------- | -------- | --------- | ------------------------------------------------------------------ |
| Merchant ID       | `mchid`        | No       | String    | Unique merchant ID assigned by QFPay                               |
| External Order ID | `out_trade_no` | Yes      | String    | Unique transaction ID within merchant system                       |
| Amount            | `txamt`        | Yes      | Int       | Amount in cents. Suggested: > 200 to avoid risk flags              |
| Currency          | `txcurrcd`     | Yes      | String(3) | Currency code. See [Currency List](/docs/api-reference/currencies) |
| RMB Tag           | `rmb_tag`      | No       | String(1) | Use `rmb_tag=Y` and `txcurrcd=CNY` to indicate RMB transaction     |
| Transaction Time  | `txdtm`        | Yes      | String    | Format: `YYYY-MM-DD hh:mm:ss`                                      |
| Device ID         | `udid`         | No       | String    | Unique identifier of the mobile device                             |
| Return URL        | `return_url`   | No       | String    | Redirect URL after payment (required for some channels)            |
| Real-name Info    | `extend_info`  | No       | Object    | Required only for Mainland China real-name flows                   |

:::note
`extend_info` detailed format

If you need to submit real‑name verification information for users in Mainland China, please use the following format:
```json
{
  "user_creid": "430067798868676871",
  "user_truename": "\u5c0f\u6797"
}
```
- `user_creid` contains the consumer’s **Mainland China ID card number**
- `user_truename` must contain the payer’s **real name**, provided either as **Unicode‑encoded text** or **Chinese characters**
:::

### Sample Request (JSON format)

```json
{
  "goods_info": "test_app",
  "goods_name": "qfpay",
  "out_trade_no": "O5DNgEgL1XpvbvQSfPhN",
  "pay_type": "800210",
  "txamt": "10",
  "txcurrcd": "HKD",
  "txdtm": "2019-09-13 04:53:03",
  "udid": "AA"
}
```

---

## API Response

### Response Parameters

| Param Code     | Type       | Description                                               |
| -------------- | ---------- | --------------------------------------------------------- |
| `syssn`        | String(40) | QFPay-generated transaction ID                            |
| `out_trade_no` | String     | Merchant's external order ID                              |
| `txdtm`        | String     | Transaction request time                                  |
| `txamt`        | Int        | Transaction amount                                        |
| `sysdtm`       | String     | QFPay system processing time (used for settlement cutoff) |
| `respcd`       | String(4)  | Response code. `0000` indicates success                   |
| `respmsg`      | String     | Message description                                       |
| `resperr`      | String     | Error description (if any)                                |
| `cardcd`       | String     | Card number (if available)                                |
| `txcurrcd`     | String     | Currency code                                             |
| `pay_params`   | Object     | Data to be passed into the WeChat SDK                     |

### Sample Response

```json
{
  "sysdtm": "2019-09-13 12:53:04",
  "paydtm": "2019-09-13 12:53:04",
  "txcurrcd": "HKD",
  "respmsg": "",
  "pay_params": {
    "package": "Sign=WXPay",
    "timestamp": 1568350384,
    "sign": "[sign string]",
    "partnerid": "316525492",
    "appid": "wx3c6896fa9b351f2a",
    "prepayid": "wx131253044253463a81dc336e1254149882",
    "noncestr": "7786db42d9a245c2b1cfc717ac59376e"
  },
  "pay_type": "800210",
  "cardcd": "",
  "udid": "AA",
  "txdtm": "2019-09-13 04:53:03",
  "txamt": "10",
  "resperr": "Transaction successful",
  "out_trade_no": "O5DNgEgL1XpvbvQSfPhN",
  "syssn": "20190913152100020001567741",
  "respcd": "0000",
  "chnlsn": ""
}
```

---

## Calling the WeChat SDK

After receiving `pay_params`, pass the values into the WeChat SDK according to [WeChat SDK documentation](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=9_12&index=2) for Android or iOS.

Ensure the parameters are mapped correctly:

* `appid`
* `partnerid`
* `prepayid`
* `package`
* `noncestr`
* `timestamp`
* `sign`

---

## Summary

* This payment method is for **native app integrations only**
* Requires prior WeChat platform registration and AppID approval
* All payment handling is passed to the official SDK
* For final transaction status confirmation, use the [Transaction Enquiry API](/docs/common-api/transaction-enquiry)
