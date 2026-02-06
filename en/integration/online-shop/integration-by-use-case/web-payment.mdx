---
id: web-payment
title: Web Payment
description: This document explains how to integrate Web-based QR code payments across different wallets including WeChat Pay, Alipay, UnionPay, and PayMe. Suitable for desktop and mobile web checkout flows.
sidebar_label: Web Payment
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# Web Payment

## Overview

Web payments allow customers to complete purchases directly from a merchant website. Users scan a QR code, confirm the payment amount, and complete payment using their selected wallet. After payment, the `return_url` will redirect the user to a predefined merchant page.

The selected wallet will deduct the amount in HKD from the customer, and QFPay will settle the same HKD amount to the merchant.

---

## HTTP Request

`POST /trade/v1/payment`

Supported `pay_type` values:

| PayType  | Description                                                                                                                                 |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `800201` | WeChat Web QR Payment (HK merchants) – see [WeChat Web Payment](/docs/online-shop/integration-by-payment-type/wechat/wechat-web-qr-payment) |
| `801514` | Alipay Web Payment (HK merchants) – see [Alipay Web Payment](/docs/online-shop/integration-by-payment-type/alipay/alipay-web-payments)      |
| `800714` | UnionPay Web Payment (Cloud QuickPass for HK merchants)                                                                                     |
| `805814` | PayMe Web Payment (HK merchants)                                                                                                            |

---

## Request Parameters

| Parameter Name    | Parameter Code | Required | Type        | Description                                                                                              |
| ----------------- | -------------- | -------- | ----------- | -------------------------------------------------------------------------------------------------------- |
| Order Amount      | `txamt`        | Yes      | Int(11)     | Integer amount in smallest currency unit. E.g. 100 = $1. Recommended to use >200 to avoid risk triggers. |
| Currency          | `txcurrcd`     | Yes      | String(3)   | Transaction currency. See [Supported Currencies](/docs/api-reference/currencies).                        |
| Payment Type      | `pay_type`     | Yes      | String(6)   | e.g. `805814` for PayMe.                                                                                 |
| External Order ID | `out_trade_no` | Yes      | String(128) | Unique transaction ID within the merchant system. Must be unique per transaction/refund.                 |
| Transaction Time  | `txdtm`        | Yes      | String(20)  | Format: `yyyy-MM-dd HH:mm:ss`                                                                            |
| Expiry Time       | `expired_time` | No       | String(3)   | QR code expiry in minutes. Default is 30, range: 5–120. (Applicable for static QR payments)              |
| Product Name      | `goods_name`   | No       | String(64)  | Optional label shown on payment page. If using Chinese, encode in UTF-8. Required for App payments.      |
| Sub-Merchant ID   | `mchid`        | No       | String(16)  | Assigned by QFPay. Use this when operating as a sub-merchant.                                            |
| Device ID         | `udid`         | No       | String(40)  | Optional device identifier shown in backend.                                                             |
| Return URL        | `return_url`   | No       | String(512) | User will be redirected here after payment completion.                                                   |

---

## Response Parameters

| Parameter Name    | Parameter Code | Type        | Description                                                                                     |
| ----------------- | -------------- | ----------- | ----------------------------------------------------------------------------------------------- |
| Payment Type      | `pay_type`     | String(6)   | e.g. `805814` for PayMe                                                                         |
| System Time       | `sysdtm`       | String(20)  | Format: `YYYY-MM-DD hh:mm:ss`. Used as settlement cut-off.                                      |
| Transaction Time  | `txdtm`        | String(20)  | Format: `YYYY-MM-DD hh:mm:ss`                                                                   |
| Response Message  | `resperr`      | String(128) | Text description of the result                                                                  |
| Order Amount      | `txamt`        | Int(11)     | Transaction amount                                                                              |
| Debug Info        | `respmsg`      | String(128) | Optional technical message for debugging                                                        |
| External Order ID | `out_trade_no` | String(128) | Matches request value                                                                           |
| QFPay Order ID    | `syssn`        | String(40)  | QFPay-generated transaction ID                                                                  |
| Response Code     | `respcd`       | String(4)   | `0000` = success, `1143/1145` = pending. See [Response Codes](/docs/api-reference/status-codes). |
| Payment URL       | `pay_url`      | String(512) | The generated payment URL. Can be used to generate QR code on desktop UI.                       |

---
