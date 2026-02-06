---
title: "Alipay In-App Payments"
description: "Integration guide for Alipay in-app payments using AlipayHK or AlipayCN SDK."
---

This guide explains how to integrate Alipay In-App payments using the Alipay SDK (AlipayHK or AlipayCN). This payment method is used within a native app environment and provides a seamless user experience.

## SDK Download

To begin integration, download the official SDK packages below:

- [Alipay Overseas SDK Documentation](https://global.alipay.com/docs/ac/app/client_integration)
- [AlipayHK SDK Download](https://global.alipay.com/docs/ac/app_hk/download)
- [AlipayHK SDK Trigger Guide](https://global.alipay.com/docs/ac/hkapi/securitypay_pay)

---

## HTTP Request

**Endpoint** : `/trade/v1/payment`\
**Method** : `POST`

| PayType  | Description                        |
| -------- | ---------------------------------- |
| `801110` | Alipay In-App (Overseas Merchants) |
| `801510` | Alipay In-App (HK Merchants)       |

## Required Parameters

| Parameter      | Required | Type        | Description                                     |
| -------------- | -------- | ----------- | ----------------------------------------------- |
| `txamt`        | Yes      | Int         | Transaction amount in cents. Example: 100 = \$1 |
| `txcurrcd`     | Yes      | String(3)   | Currency code (e.g. HKD)                        |
| `pay_type`     | Yes      | String(6)   | Use 801110 for Alipay CN, 801510 for AlipayHK   |
| `out_trade_no` | Yes      | String(128) | Unique merchant transaction ID                  |
| `txdtm`        | Yes      | String(20)  | Format: `YYYY-MM-DD hh:mm:ss`                   |
| `goods_name`   | Yes      | String      | Product name                                    |
| `return_url`   | Yes      | String      | Redirect URL after payment                      |
| `seller_id`    | Yes      | String      | Alipay seller account                           |
| `mchid`        | Yes      | String(16)  | QFPay assigned Merchant ID                      |

See [Public Payment Parameters](/docs/api-reference/request-format/#public-payment-request-parameters) for shared fields.

---

## Optional Parameters

| Parameter Name      | Parameter Key  | Required                 | Type       | Description                                                                                                   |
| ------------------- | -------------- | ------------------------ | ---------- | ------------------------------------------------------------------------------------------------------------- |
| Product Description | `goods_info`   | No                       | String     | Required by Alipay. Special characters are not allowed.                                                       |
| Payment Tag         | `pay_tag`      | No                       | String(16) | Default: ALIPAYHK<br />For Alipay Mainland use: ALIPAYCN                                                      |
| Order Expiry Time   | `expired_time` | No<br />(Main-scan only) | String(3)  | Expiration time in minutes. Default is 30 minutes. Min: 5, Max: 120.<br />Applicable to WeChat Pay and Alipay |

## Sample Request (Form Payload)

```plaintext
txamt=1
&txcurrcd=HKD
&pay_type=801510
&out_trade_no=052711570017898
&txdtm=2021-05-27 11:57:00
&goods_name=goods_name
&goods_info=goods_info
&mchid=nDB64h9qJ1An
&trade_name=trade_name
&goods_detail=goods_detail
&return_url=https://www.qfpay.global/
&pay_tag=ALIPAYHK
&seller_id=testoverseas9191@alipay.com
```

## Response Fields (pay_params)

These fields are returned by QFPay and should be passed into the Alipay SDK.

<table style="min-width: 75px">
<colgroup><col style="min-width: 25px"><col style="min-width: 25px"><col style="min-width: 25px"></colgroup><tbody><tr><th colspan="1" rowspan="1"><p>Parameter Key</p></th><th colspan="1" rowspan="1"><p>Sub Parameter Key</p></th><th colspan="1" rowspan="1"><p>Description</p></th></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">pay_params</code></p></td><td colspan="1" rowspan="1"><p><code spellcheck="false">partner</code></p></td><td colspan="1" rowspan="1"><p>Partner ID</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">seller_id</code></p></td><td colspan="1" rowspan="1"><p>Unique Alipay user ID of the receiving account</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">subject</code></p></td><td colspan="1" rowspan="1"><p>Product title / trade title / order title</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">body</code></p></td><td colspan="1" rowspan="1"><p>Detailed description of the transaction. For multiple items, concatenate descriptions into this field</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">total_fee</code></p></td><td colspan="1" rowspan="1"><p>Total order amount</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">notify_url</code></p></td><td colspan="1" rowspan="1"><p>Notification callback URL</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">service</code></p></td><td colspan="1" rowspan="1"><p>Service name</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">cardcd</code></p></td><td colspan="1" rowspan="1"><p>Card number</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">payment_type</code></p></td><td colspan="1" rowspan="1"><p>Payment type</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">_input_charset</code></p></td><td colspan="1" rowspan="1"><p>Character encoding format</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">it_b_pay</code></p></td><td colspan="1" rowspan="1"><p>Custom timeout parameter</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">return_url</code></p></td><td colspan="1" rowspan="1"><p>Redirect URL after payment</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">payment_inst</code></p></td><td colspan="1" rowspan="1"><p>Payment institution</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">currency</code></p></td><td colspan="1" rowspan="1"><p>Currency code</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">product_code</code></p></td><td colspan="1" rowspan="1"><p>Product code</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">sign</code></p></td><td colspan="1" rowspan="1"><p>RSA signature (Required)</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">sign_type</code></p></td><td colspan="1" rowspan="1"><p>Signature type</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">secondary_merchant_id</code></p></td><td colspan="1" rowspan="1"><p>Secondary merchant ID</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">secondary_merchant_name</code></p></td><td colspan="1" rowspan="1"><p>Secondary merchant name</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">secondary_merchant_industry</code></p></td><td colspan="1" rowspan="1"><p>Secondary merchant industry</p></td></tr><tr><td colspan="1" rowspan="1"><p><code spellcheck="false">chnlsn</code></p></td><td colspan="1" rowspan="1"><p>Channel transaction number</p></td></tr><tr><td colspan="1" rowspan="1"><p>Common Response Parameters</p></td><td colspan="1" rowspan="1"><p>—</p></td><td colspan="1" rowspan="1"><p>—</p></td></tr></tbody>
</table>

## Sample QFPay Response

```json
{
  "pay_type": "801510",
  "sysdtm": "2021-05-27 11:57:02",
  "paydtm": "2021-05-27 11:57:02",
  "udid": "qiantai2",
  "txcurrcd": "HKD",
  "txdtm": "2021-05-27 11:57:00",
  "txamt": "1",
  "resperr": "交易成功",
  "respmsg": "",
  "out_trade_no": "052711570017898",
  "syssn": "20210527154100020004180921",
  "pay_params": {
    "body": "goods_info",
    "forex_biz": "FP",
    "seller_id": "2088231067382451",
    "secondary_merchant_id": "1000007081",
    "service": "mobile.securitypay.pay",
    "payment_inst": "ALIPAYHK",
    "it_b_pay": "30m",
    "secondary_merchant_name": "IFlare Hong Kong Limited (external) - online",
    "_input_charset": "UTF-8",
    "sign": "iU1yXUnsCK7rJAu0DoN61arVexbIfo3GLR5jr3QzjkZ29INSPhcA4e%2F2%2BdPrsf5huzQAkxVKP0CTfvaGPMYqNkxmhoaJWUH0ZhgYDgKugMvtweBvRqOX2W0h3A%2F%2FIdJuxeyOAuh7bHiuazSB3ZH%2BEQwRGP%2Bkk8Jpha930gHwPtw%3D",
    "currency": "HKD",
    "out_trade_no": "20210527154100020004180921",
    "payment_type": "1",
    "total_fee": 0.01,
    "sign_type": "RSA",
    "notify_url": "https://test-o2-hk.qfapi.com/trade/alipay_hk/v1/notify",
    "partner": "2088231067382451",
    "secondary_merchant_industry": "5941",
    "product_code": "NEW_WAP_OVERSEAS_SELLER",
    "return_url": "https://www.qfpay.global/",
    "subject": "goods_name"
  },
  "respcd": "0000",
  "chnlsn": "",
  "cardcd": ""
}
```

## Using Alipay SDK

After receiving the `pay_params` in the response, construct the `orderInfo` string required by the SDK in the following format:

1. Join all fields in the format: `key="value"`
2. Sort keys in ascending order (alphabetical)
3. Concatenate with `&`
4. Place `sign` and `sign_type` at the end

### Sample

```plaintext
_input_charset="UTF-8"&body="goods_info"&currency="HKD"&forex_biz="FP"&it_b_pay="30m"&notify_url="https://test-o2-hk.qfapi.com/trade/alipay_hk/v1/notify"&out_trade_no="20210527154100020004180921"&partner="2088231067382451"&payment_inst="ALIPAYHK"&payment_type="1"&product_code="NEW_WAP_OVERSEAS_SELLER"&return_url="https://www.qfpay.global/"&secondary_merchant_id="1000007081"&secondary_merchant_industry="5941"&secondary_merchant_name="IFlare Hong Kong Limited (external) - online"&seller_id="2088231067382451"&service="mobile.securitypay.pay"&subject="goods_name"&total_fee="0.01"&sign="iU1yXUnsCK7rJAu0DoN61arVexbIfo3GLR5jr3QzjkZ29INSPhcA4e%2F2%2BdPrsf5huzQAkxVKP0CTfvaGPMYqNkxmhoaJWUH0ZhgYDgKugMvtweBvRqOX2W0h3A%2F%2FIdJuxeyOAuh7bHiuazSB3ZH%2BEQwRGP%2Bkk8Jpha930gHwPtw%3D"&sign_type="RSA"
```

<Note>
  Make sure you:

  - Follow the correct SDK version and region (HK vs CN)
  - Use the exact `key` names and `values` returned in `pay_params`
  - Keep your sign logic consistent with Alipay SDK format
</Note>