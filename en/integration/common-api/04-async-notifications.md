---
id: async-notifications
title: Asynchronous Notifications
description: Learn how to handle QFPay's payment and refund asynchronous notifications.
sidebar_label: Async Notifications
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Asynchronous Notifications

:::note
To configure your notification URL, please send your endpoint address along with your merchant and store information to `technical.support@qfpay.com`.
:::

QFPay supports asynchronous notifications for:

- Payments (`"notify_type": "payment"`)
- Refunds (`"notify_type": "refund"`)

These notifications allow merchants to receive real-time updates on transaction results. Since parameters may expand in future versions, your integration should be able to gracefully handle unknown fields.

---

## Overview

When a payment or refund is completed, QFPay will POST a JSON-formatted notification to the merchant-defined callback URL.

:::tip
It is strongly recommended to **verify** the status of a transaction using the [Transaction Enquiry API](/docs/common-api/transaction-enquiry) in addition to handling the callback.
:::

---

## Notification Rules

1. Only **successful** payments and refunds will trigger a notification.

2. Please register your notification endpoint via email to `technical.support@qfpay.com`. Our team will configure it for you.

3. Merchants must validate the notification using the **signature verification** process below. After successful verification, return:
   - HTTP Status Code: `200 OK`
   - Response Body: `SUCCESS`

4. If the expected response is not received, QFPay will retry the callback at the following intervals:
   - 2 minutes → 10 minutes → 10 minutes → 60 minutes → 2 hours → 6 hours → 15 hours
   - Retry stops after receiving `200 OK` and `SUCCESS`

5. One `app_code` + `client_key` pair can only be bound to **one notification URL**. Agents should use a shared endpoint for sub-merchants.

6. **Method:** `POST`  
   **Content-Type:** `application/json`  
   **Allowed Ports:** `80` and `443` only (for security)

---

## Signature Verification

The verification process differs from regular API requests.

**Steps:**

1. Extract the value from the `X-QF-SIGN` header.

2. Concatenate the raw request body (JSON string) **+** your `client_key`.

3. Generate an MD5 hash of this combined string.

4. If the hash matches the `X-QF-SIGN` value, the message is valid. Return `200 OK` with body `SUCCESS`.

---

## Signature Example

```python
import hashlib
import json

# Client Credentials
client_key = "3ABB1BFFE2E0497BB9270978B0BXXXXX"

# Raw Content Data
data = {"status": "1", "pay_type": "800101", "sysdtm": "2020-06-15 10:32:58", "paydtm": "2020-06-15 10:33:35", "goods_name": "", "txcurrcd": "HKD", "txdtm": "2020-06-15 10:32:58", "mchid": "O37MRh6Qq5", "txamt": "10", "exchange_rate": "", "chnlsn2": "", "out_trade_no": "9G3ZIWTG1R3IVSC2AH2O5EGKJQ7I72QO", "syssn": "20200615000200020000641807", "cash_fee_type": "", "cancel": "0", "respcd": "0000", "goods_info": "", "cash_fee": "0", "notify_type": "payment", "chnlsn": "2020061522001453561406303428", "cardcd": "2088032341453564"}

combine_str = (json.dumps(data)+client_key).encode()

signature = hashlib.md5(combine_str).hexdigest().upper()

print(signature)
```

> Sample Signature Output:

```
"A4021A3B1EBBB0F05451EF94E9EXXXXX"
```

## Notification Response Example

```json
{
  "status": "1",
  "pay_type": "800101",
  "sysdtm": "2020-05-14 12:32:56",
  "paydtm": "2020-05-14 12:33:56",
  "goods_name": "",
  "txcurrcd": "HKD",
  "txdtm": "2020-05-14 12:32:56",
  "mchid": "lkbqahlRYj",
  "txamt": "10",
  "exchange_rate": "",
  "chnlsn2": "",
  "out_trade_no": "YEPE7WTW46NVU30JW5N90H7DHD94N56B",
  "syssn": "20200514000300020093755455",
  "cash_fee_type": "",
  "cancel": "0",
  "respcd": "0000",
  "goods_info": "",
  "cash_fee": "0",
  "notify_type": "payment",
  "chnlsn": "2020051422001453561444935817",
  "cardcd": "2088032341453564"
}
```

## Response Field Reference

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `status` | Yes | String | `1` = Payment Successful |
| `notify_type` | Yes | String | `payment` or `refund` |
| `pay_type` | Yes | String | QFPay payment code (see [Payment Types](/docs/api-reference/paytypes)) |
| `syssn` | Yes | String | QFPay Transaction Number |
| `out_trade_no` | Yes | String | Merchant Order Number |
| `txamt` | Yes | String | Amount in cents. Suggest value > 200 |
| `txcurrcd` | Yes | String | Transaction currency (see [Currencies](/docs/api-reference/currencies)) |
| `txdtm` | Yes | String | Merchant-side transaction time |
| `sysdtm` | Yes | String | System transaction time (used for settlement cutoff) |
| `paydtm` | Yes | String | Payment time |
| `cancel` | Yes | String | Transaction cancel status: <br/> 0 = Not cancelled <br/> 1–5 = See below for meanings |
| `respcd` | Yes | String | Should always be `0000` in async notifications |
| `mchid` | No | String | QFPay Merchant ID (for agent access) |
| `goods_name` | No | String | Product name (UTF-8 if Chinese) |
| `goods_info` | No | String | Product description |
| `exchange_rate` | No | String | Currency exchange rate used |
| `chnlsn` | No | String | Channel transaction number |
| `chnlsn2` | No | String | Additional channel transaction reference |
| `cardcd` | No | String | Card number |
| `cash_fee` | No | String | Actual paid amount (after discount) |
| `cash_fee_type` | No | String | Currency of `cash_fee` |
| `cash_refund_fee` | No | String | Actual refund amount |
| `cash_refund_fee_type` | No | String | Refund currency |

---

## Cancel Field Definitions

Value | Meaning
------|--------
0 | Not Cancelled  
1 | **CPM:** Reversed or Refunded  
2 | **MPM:** Cancelled  
3 | Refunded  
4 | Preauth order completed (Alipay)  
5 | Partially Refunded  

---

## Notification IP Addresses

Ensure your server allows POST requests from:

- `13.228.112.115`
- `18.138.115.47`
- `18.166.202.92`