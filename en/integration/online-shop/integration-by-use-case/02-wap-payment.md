---
id: wap-payment
title: WAP Payment (Mobile Browser)
description: This document explains how to initiate WAP (H5) payments using different wallet types through QFPay from mobile browsers like Chrome or Safari.
sidebar_label: WAP Payment
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# WAP Payment (Mobile Browser)

WAP (or H5) payment allows merchants to trigger wallet payment flows from mobile browsers such as Chrome or Safari.

:::note
For WAP/H5 payments, merchants are advised to guide users to open the payment link in a mobile browser such as Chrome, Safari, or Edge. Due to uncontrollable restrictions in social apps like WhatsApp, Facebook Messenger, or WeChat, QFPay cannot guarantee that these apps can automatically invoke external wallet apps. For example, Alipay cannot be automatically opened within the WeChat app — this is a browser limitation beyond QFPay's control.
:::

---

## HTTP Request

`POST ../trade/v1/payment`

You can find the corresponding `pay_type` for each wallet in the table below:

| PayType | Description |
|---------|-------------|
| 800212 | WeChat H5 Payment — see [WeChat H5 Payment](/docs/online-shop/integration-by-payment-type/wechat/wechat-pay-h5) |
| 801512 | Alipay HK WAP Payment — see [Alipay H5 Payment](/docs/online-shop/integration-by-payment-type/alipay/alipay-wap-h5-payments) |
| 800712 | UnionPay WAP Payment |
| 805812 | PayMe WAP Payment |

---

## Request Parameters

| Name | Parameter | Required | Type | Description |
|------|-----------|----------|------|-------------|
| Transaction Amount | `txamt` | Yes | Int(11) | Amount in smallest unit (e.g. 100 = $1). Recommended to be > 200 to avoid risk control failures. |
| Currency | `txcurrcd` | Yes | String(3) | Transaction currency. See [Currency List](/docs/api-reference/currencies). |
| Payment Type | `pay_type` | Yes | String(6) | e.g. PayMe WAP Payment = 805812 |
| Order Number | `out_trade_no` | Yes | String(128) | Unique order number per merchant account across all payment/refund requests. |
| Transaction Time | `txdtm` | Yes | String(20) | Format: `YYYY-MM-DD hh:mm:ss` |
| Product Name | `goods_name` | No | String(64) | Product name/identifier. Max 20 alphanumeric chars or Chinese in UTF-8. |
| QFPay Merchant ID | `mchid` | No | String(16) | Assigned by QFPay. Required if present in backend configuration. |
| Device ID | `udid` | No | String(40) | Unique device ID shown in merchant dashboard. |
| Redirect URL | `return_url` | No | String(255) | URL the user is redirected to after payment completes. |
| Notification URL | `notify_url` | No | String(255) | URL to receive asynchronous notifications after payment. |

---

## Response Parameters

| Name | Parameter | Type | Description |
|------|-----------|------|-------------|
| Payment Type | `pay_type` | String(6) | e.g. PayMe WAP Payment |
| System Time | `sysdtm` | String(20) | `YYYY-MM-DD hh:mm:ss`. Used as settlement cutoff. |
| Transaction Time | `txdtm` | String(20) | As sent in request. |
| Response Message | `resperr` | String(128) | Description or status message. |
| Amount | `txamt` | Int(11) | Transaction amount. |
| Debug Info | `respmsg` | String(128) | Internal response/debug message. |
| External Order No. | `out_trade_no` | String(128) | Returned for reference. |
| QFPay Order No. | `syssn` | String(40) | QFPay system-generated order number. |
| Response Code | `respcd` | String(4) | `0000` = success, `1143/1145` = retry with status check, others = failure. See [Status Codes](/docs/api-reference/status-codes). |
| Payment URL | `pay_url` | String(512) | Redirect URL (mobile browser), or QR code display URL (PC browser). |

---

## Summary

* Suitable for mobile browser environments (not within WeChat or social apps).
* Ensure the `return_url` and `notify_url` are set correctly if redirection or backend notification is required.
* Use the `pay_url` to display the QR code or redirect the user to complete the payment.
* Consider polling or transaction enquiry APIs to confirm payment result if response code is not `0000`.