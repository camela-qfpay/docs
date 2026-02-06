---
id: in-app-payment
title: In-App Payment (Native App Integration)
description: This page explains how to integrate e-wallet payment functions within a native App environment, supporting various mainstream wallets and standard payment flows.
sidebar_label: In-App Payment
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# In-App Payment (Native App Integration)

Integrating e-wallet payment functions into native iOS or Android apps allows users to complete payments seamlessly within the app, without being redirected to external browsers.

---

## Supported Scenarios

In-App payments are suitable for:

* Native apps with built-in shopping or payment features
* Invoking wallet payment modules via SDKs (e.g. WeChat, Alipay)
* Letting users initiate payment inside the app and redirect to a wallet to complete payment

---

## Common Wallets and Transaction Types

Refer to the following table for `pay_type` values and related documentation:

| Transaction Type (`pay_type`) | Wallet         | Description                                                                                         |
|------------------------------|----------------|-----------------------------------------------------------------------------------------------------|
| `800210`                     | WeChat         | [WeChat In-App Payment](/docs/online-shop/integration-by-payment-type/wechat/wechat-in-app-payments) (via SDK) |
| `801510`                     | AlipayHK       | [Alipay Hong Kong In-App Payment](/docs/online-shop/integration-by-payment-type/alipay/alipay-in-app) (via URL Scheme or SDK) |
| `805810`                     | PayMe (HK)     | PayMe In-App Payment                                                                                |
| `800710`                     | UnionPay       | UnionPay In-App Payment                                                                             |

---

## Common Integration Methods

* **SDK Mode**: The merchant app uses a third-party SDK to call wallet APIs
* **URL Scheme / Deep Link**: Merchant generates a payment link and redirects to the wallet App
* **Redirect Back to App**: After payment is completed, the wallet redirects back to the merchant app (requires setting a return URL or deep link binding)

---

## General API Flow

Most In-App payment types use the `POST /trade/v1/payment` endpoint to initiate payment requests. The response will include `pay_params` or a `pay_url`, which is then passed to the SDK or used to trigger a redirect.

Refer to the documentation of each wallet for specific request and response formats.

---

## Important Notes

* Each wallet has different requirements — check if App payment permission needs to be pre-approved (e.g. WeChat SDK AppID, Alipay Merchant Setup)
* Always test redirect and return flow thoroughly to ensure the app receives the payment result properly
* It's recommended to use the Transaction Enquiry API to confirm the final payment status, especially if the app is closed before receiving the callback

---

## Related Resources

* [Transaction Enquiry API](/docs/common-api/transaction-enquiry)
* [Payment Status Code Reference](/docs/api-reference/status-codes)
* [Developer FAQ](/docs/faq)