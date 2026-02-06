---
title: "Integration Guide for Recurring Payments"
---

## Prerequisites

Before you begin integration, ensure the following:

- You have API credentials (`AppCode`, `AppKey`) for your environment.
- Your notify URL has been configured with QFPay for receiving asynchronous webhook notifications.
- Your frontend supports rendering and submitting [Payment Element](/docs/online-shop/checkout-integration/payment-element).

---

## Integration Flow Overview

The complete recurring payment integration consists of the following steps:

1. **Set up webhook endpoint** to receive `payment_token`, `subscription`, and `subscription_payment` notifications.
2. **Create Customer** object and obtain `customer_id`.
3. **Render Payment Element** in frontend to create token using `payment.pay()` and `customer_id`.
4. **`Capture the token_id`** from webhook notification or token API response.
5. **Create Product** object to define billing amount, frequency, and interval.
6. **Create Subscription** with `customer_id`, `token_id`, and `product_id`.
7. (Optional) Monitor subscription status via webhook or [Query API](/docs/recurring-api-reference#query-subscription).

![recurring-payment-flow](https://www.plantuml.com/plantuml/png/fLJDZYCr4BxxAKfp9n8iS9W3MYq8YTkYvRPQeOfTdPXnbpqkUqBpzCJT7NSxz92ivkPqVLN-VifvBmbZohrJg9EFeBCatrC4b7gUI-UJFY8dGAbdfGB6PBKDfT3_AOEKyaF5oY29-eS6zjm574ROxxz-n64JSmeZu5pkYHCSCD49XnPZHJB54VVRTFo0_FIWr27w7E3RtSVeJTP9OKwUSx-dg2gnRtwQw3w2ZeI98CpWyMifZpIlo-3tVv5E4EavaoJ5FX54UpWcbIAoe4xMCs3lCxUVT8wHM0-As41fKvCF2v58AKUkDrbJe1Srt-r-hd7S8wU6jwsdrlz7K8KmzgHwlxUE5FLedJeVdUK3e36HH6vgggDQKUzsbu3_y4_4yDbsPGmCb6QUvijQRXspN60vv3COem7BdT-zfZUD5xHYuMJJ4S94eL6EqHozCxDs63y0dwVJty760KmPUUq2g4RchPXdvHCnM-WNlEsinh8mQv--tttAUz7HXbAvuOXCq3s11D9b7WI7_8en4tmQlBK4pae2twss4f0DF6VaPDFGIBwMDEQ98JYhSU_eIpLC3zgHGC28FIMAjnT85lrbykmBoi3w63txB40l9SNh_f3bs7RFYpNZYPkD_67NsIW9Bam_B_hO9elE_aFGGcRLRrxKvOAjbHopHBVox55Tcs8JnMbtgfsB7wVmU9bRSqQNuDqldyRVDf81ZKBg52ghwdz1wICwHtoW1Hz9WcUvZktkjkh1nR3IQMmaNRRegZtWWHfdhKWkBTfpsSqMhgRgmgaEHZPhVki_QMl-4yiBHTiDA-iak-fWOsyBhPYAtVAr7RjbumQAd11qqOwS7OcqkRrzFXlrbI-iosp0K8c1pDrFiRZ-GjilySVTX_RVOls-dFS1CYVhOBz6GLby7q4ZQtAEJ1lGH70YkvbA8-DkfPNwUsAJU_Sl)

---

## Best Practices

:::note Use [Query API](/docs/recurring-api-reference#query-subscription) for  syncing subscription states to your CRM. :::

:::tip After each subscription creation, confirm receipt of webhook with `ACTIVE` or `INCOMPLETE` state to ensure it was created successfully. :::

:::warning Always store `subscription_id` and `token_id` in your database for tracking and later billing actions such as retry, cancellation, or reporting. :::

---

## Next Steps

- [Recurring API Reference](/docs/recurring-api-reference)
- [Webhook Reference](/docs/recurring-webhook)
- [Element SDK Integration](/docs/online-shop/checkout-integration/payment-element)