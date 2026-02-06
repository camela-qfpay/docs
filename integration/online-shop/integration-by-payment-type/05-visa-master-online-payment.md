---
title: "Visa / Mastercard Online Payments"
---

This page provides integration guidance for online credit card payments using Visa and Mastercard. Our solution currently supports credit card payments in the [Hong Kong environment](/docs/preparation/environments), covering all major card issuers.

---

## Integration Methods

Merchants can choose from two integration options:

1. [**QFPay Checkout Services**](/docs/online-shop/checkout-integration/checkout)\
   A hosted payment page solution—ideal for merchants who prefer minimal PCI scope and faster setup.
2. [**QFPay Payment Element SDK**](/docs/online-shop/checkout-integration/payment-element)\
   A client-side SDK that embeds payment input fields directly into your website, offering full UX control and 3DS support.

---

## Asynchronous Notification

:::info QFPay will send an asynchronous notification to your backend to confirm the transaction result. :::

Refer to [Asynchronous Notification](/docs/common-api/async-notifications) for details on the notification format and signature verification.

> Example Notification Payload:

```json
{
  "cardtp": "5",
  "cancel": "0",
  "pay_type": "802801",
  "order_type": "payment",
  "clisn": "054256",
  "txdtm": "2021-12-08 07:04:15",
  "out_trade_no": "354267281",
  "syssn": "20211208180500020000001637",
  "sysdtm": "2021-12-08 15:04:16",
  "paydtm": "2021-12-08 15:06:51",
  "txcurrcd": "HKD",
  "udid": "qiantai2",
  "userid": "1130000355",
  "txamt": "1",
  "respcd": "0000",
  "errmsg": "success"
}
```

:::warning Always validate the notification using the provided signature, and never trust the notification result blindly. :::

:::tip You can use [**Transaction Enquiry**](/docs/common-api/transaction-enquiry) API as a fallback to confirm transaction status when:

- Callback is delayed
- Signature verification fails
- Merchant server missed the notification :::

---

## Test Cards

The following test cards are available in the **Sandbox** environment. Use them to simulate different transaction results, including 3D Secure (3DS) flows.

| Card Brand | Card Number         | Simulation Result                 |
| ---------- | ------------------- | --------------------------------- |
| Mastercard | 5200 0000 0000 1096 | Successful payment                |
| Visa       | 4000 0000 0000 1091 | Successful payment                |
| Mastercard | 5200 0000 0000 1005 | Successful (3DS frictionless)     |
| Visa       | 4000 0000 0000 1000 | Successful (3DS frictionless)     |
| Mastercard | 5200 0000 0000 1120 | Failed (during verification)      |
| Visa       | 4000 0000 0000 1125 | Failed (during verification)      |
| Mastercard | 5200 0000 0000 1013 | Failed (3DS frictionless failure) |
| Visa       | 4000 0000 0000 1018 | Failed (3DS frictionless failure) |

:::tip If you're unsure which integration method fits your use case (Checkout vs Element SDK), refer to your onboarding document or contact QFPay support. :::