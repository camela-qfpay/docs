---
id: recurring-webhook
title: Subscription Webhook (Recurring Payment Notifications)
description: This document describes the webhook notification system for recurring payments (subscriptions), including token creation, subscription status changes, and billing result notifications.
sidebar_label: Webhook Notifications
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# Subscription Webhook (Recurring Payment Notifications)

This page describes how QFPay sends asynchronous notifications related to the subscription system. This includes:

- Token creation results
- Subscription state changes
- Recurring billing results

## How to Configure Notification URL

:::note
To set your webhook endpoint, please email your webhook URL along with your merchant ID and store ID to: `technical.support@qfpay.com`
:::

---

## Token Creation Notification

Triggered after a payment token is created via Element SDK.

### Notification Fields

| Field           | Description                                                   |
|----------------|---------------------------------------------------------------|
| `userid`       | QFPay Store ID                                                |
| `notify_type`  | Notification type. Value: `payment_token`                    |
| `event`        | One of: `NEW`, `MATCH`, `CONFLICT`                            |
| `tokenid`      | ID of the generated payment token                             |
| `token_expiry_date` | Expiration date of the token                                |
| `cardcd`       | Masked card number                                            |
| `card_scheme`  | Card scheme, e.g., VISA, MASTERCARD                           |
| `respcd`       | Response code, `0000` = success                               |
| `respmsg`      | Message, e.g., "success"                                     |
| `sysdtm`       | System timestamp of the event                                 |
| `customer_id`  | Customer ID if linked                                        |
| `token_reason` | Reason for tokenisation                                       |
| `token_reference` | Token reference in system                                    |

### Example

```json
{
  "respmsg": "success",
  "card_scheme": "ECMC_DEBIT",
  "cardcd": "5200****1096",
  "tokenid": "tk_6a699aae75094caeb066f****988daa32de",
  "respcd": "0000",
  "token_expiry_date": "2024-04-30 00:00:00",
  "sysdtm": "2024-04-29 15:37:17",
  "notify_type": "payment_token",
  "event": "CONFLICT"
}
```

---

## Subscription Status Change

Sent when the status of a subscription changes.

### Notification Fields

| Field              | Description                                                             |
|-------------------|-------------------------------------------------------------------------|
| `notify_type`      | Notification type. Value: `subscription`                                |
| `subscription_id`  | ID of the subscription object                                           |
| `state`            | Current subscription state (e.g., `ACTIVE`, `COMPLETED`, `INCOMPLETE`) |
| `sysdtm`           | System timestamp of the status change                                   |

### Example

```json
{
  "state": "COMPLETED",
  "sysdtm": "2024-04-24 15:19:39",
  "notify_type": "subscription",
  "subscription_id": "sub_e51bb914919*****f6b0fe36d"
}
```

---

## Subscription Payment Result

Sent when a billing attempt occurs (either success or failure).

### Notification Fields

| Field                  | Description                                                                                   |
|------------------------|-----------------------------------------------------------------------------------------------|
| `notify_type`          | Notification type. Value: `subscription_payment`                                              |
| `subscription_id`      | Subscription ID                                                                               |
| `subscription_order_id`| Billing order ID, format: `sub_ord_{subscription_id}_{0001}`                                 |
| `respcd`               | Response code. `0000` = success                                                               |
| `respmsg`              | Response message                                                                              |
| `syssn`                | System transaction ID                                                                         |
| `txdtm`                | Transaction timestamp                                                                         |
| `txamt`                | Transaction amount                                                                            |
| `txcurrcd`             | Currency                                                                                      |
| `customer_id`          | Customer ID                                                                                   |
| `product_id`           | Product ID(s), comma-separated if multiple                                                    |
| `cardcd`               | Masked card number                                                                            |
| `card_scheme`          | Card scheme, shown only if success                                                            |
| `current_iteration`    | Billing iteration number                                                                      |

### Example

```json
{
  "txcurrcd": "HKD",
  "reason": "AUTHORISED",
  "cardcd": "",
  "subscription_order_id": "sub_ord_a360f06eb*****ad6aff24c3a",
  "product_id": "prod_8c838c17ddb043b9***11f1a85c30",
  "txdtm": "2024-04-24 15:19:37",
  "txamt": "300",
  "card_scheme": "VISA_DEBIT-SSL",
  "syssn": "20240424180500020000015704",
  "respcd": "0000",
  "subscription_id": "sub_e51bb914919***31d800f6b0fe36d",
  "customer_id": "cust_a9c0bcf2717f4***786a10e5f8f2",
  "notify_type": "subscription_payment",
  "current_iteration": "1"
}
```

---

## Retry Logic & Idempotency

:::tip
Ensure your webhook receiver is idempotent. The same notification may be sent multiple times in the case of delivery failure.
:::

If the receiving server does not respond with HTTP 200 OK, the message will be retried.

Retries use an exponential backoff strategy, but maximum retries may be subject to internal configuration.