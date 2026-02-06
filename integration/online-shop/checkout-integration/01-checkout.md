---
title: "Hosted Checkout Page"
---

## Introduction

This section introduces QFPay’s hosted Checkout Page, which enables merchants to integrate multiple payment methods via a prebuilt interface. Sample code snippets are provided below.

## Checkout Page Design

<Link href="/img/shouyintai.png" target="_blank">
  ![Checkout Page](@site/static/img/shouyintai.png)
</Link>

The hosted checkout page is fully responsive and adapts to all screen sizes. It supports English, Simplified Chinese, and Traditional Chinese. Payment options and the UI can be customised based on merchant requirements. Contact `technical.support@qfpay.com` for support.

## API Environment

:::warning Remember to immediately refund test transactions via the Merchant Portal, QFPay APP or API. :::

For base URLs and environment details, please refer to the [Environment Documentation](/docs/preparation/environment).

## Process Flow

<Link href="/img/flowchart.png" target="_blank">
  ![Checkout Flow](@site/static/img/flowchart.png)
</Link>

1. Customer browses the merchant's website and proceeds to checkout.
2. Merchant redirects to the QFPay hosted checkout page.
3. Customer selects a payment method and completes the transaction.
4. After payment, customer is redirected back to the merchant’s website with the result.

## API Request Parameters

**Endpoint** : `https://<base-url>/checkstand/#/?`\
**Method** : `GET`

| Attribute               | Type        | Mandatory | Description                                                                                                                                  |
| ----------------------- | ----------- | --------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `appcode`               | String(64)  | Yes       | API credential from QFPay                                                                                                                    |
| `sign_type`             | String      | Yes       | `SHA256` (recommended) or `MD5`                                                                                                              |
| `sign`                  | String      | Yes       | Signature string                                                                                                                             |
| `paysource`             | String      | Yes       | Must end with `_checkout` (e.g. `remotepay_checkout`)                                                                                        |
| `txamt`                 | Int         | Yes       | Payment amount in cents (e.g. 1099)                                                                                                          |
| `txcurrcd`              | String(3)   | Yes       | Currency (e.g. HKD)                                                                                                                          |
| `out_trade_no`          | String(128) | Yes       | Unique transaction ID                                                                                                                        |
| `txdtm`                 | String(32)  | Yes       | Transaction timestamp (e.g. `YYYY-MM-DD hh:mm:ss`)                                                                                           |
| `return_url`            | String      | Yes       | Redirect URL after successful payment                                                                                                        |
| `failed_url`            | String      | Yes       | Redirect URL after failed payment                                                                                                            |
| `notify_url`            | String      | Yes       | URL for asynchronous notification                                                                                                            |
| `mchntid`               | String(16)  | No        | QFPay merchant ID (for agents)                                                                                                               |
| `goods_name`            | String(64)  | No        | Product name (≤20 chars; no special chars)                                                                                                   |
| `udid`                  | String(40)  | No        | Device ID                                                                                                                                    |
| `expired_time`          | String(3)   | No        | QRC expiration (5–120 minutes; WeChat, Alipay only)                                                                                          |
| `checkout_expired_time` | String(13)  | No        | Checkout page expiration time. Accepts minutes (up to 3 digits, e.g. 120) or Unix timestamp in milliseconds (13 digits, e.g., 1715686118000) |
| `limit_pay`             | String      | No        | `no_credit` to block credit cards (WeChat only)                                                                                              |
| `lang`                  | String(5)   | No        | UI language: `zh-hk`, `zh-cn`, `en`                                                                                                          |
| `cancel_url`            | String      | No        | URL when user cancels checkout                                                                                                               |

## Creating a Checkout Order

:::info Each `out_trade_no` must be unique to avoid duplicate transactions. :::

Construct a URL with signed parameters using your `app_key` and redirect the user to the checkout page. See example code in the downloadable [Checkout Boilerplate HTML](@site/static/files/qfpay_online_checkout.html).

```html
<!-- Simplified HTML signature generation and redirection example -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/js-sha256/0.9.0/sha256.min.js"></script>
<script>
  const params = {
    appcode: "CC6FB660837E49F7A675D2**********",
    goods_name: "checkout_product",
    out_trade_no: "TXN1234567890",
    paysource: "remotepay_checkout",
    return_url: "https://merchant.com/success",
    failed_url: "https://merchant.com/failed",
    notify_url: "https://merchant.com/notify",
    sign_type: "sha256",
    txamt: "1000",
    txcurrcd: "HKD",
    txdtm: "2025-01-01 12:00:00"
  };
  const apiKey = "YOUR_SECRET_KEY";
  const sorted = Object.keys(params).sort().map(k => `${k}=${params[k]}`).join('&');
  const sign = sha256(sorted + apiKey);
  const url = `https://test-openapi-hk.qfapi.com/checkstand/#/?${sorted}&sign=${sign}`;
  window.location.href = url;
</script>
```