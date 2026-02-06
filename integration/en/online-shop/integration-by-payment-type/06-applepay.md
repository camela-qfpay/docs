---
title: "Apple Pay Support"
---

## Introduction

QFPay Checkout and Payment Element SDK both support Apple Pay as an embedded payment option. Merchants must first enable this payment method in the QFPay backend before customers can use it. Apple Pay helps improve authorisation rates and streamlines the checkout experience on Apple-supported devices.

---

## Supported Devices

Apple Pay is available on:

- iPhone, iPad, and Mac (via Safari)
- Third-party browsers on Windows and other platforms (via QR Code)

:::info Apple Pay is **not supported on Android devices**. If a customer selects Apple Pay on an unsupported device, the Apple Pay button will be hidden automatically. :::

---

## QR Code Payment Mode

If the customer is using a device that doesn't support Apple Pay natively—such as a Windows desktop—a **QR code** will be displayed automatically. The customer can then scan this code using a supported iPhone or iPad (iOS version ≥ 18) to complete the payment securely.

:::warning This feature is **not available** if the customer's Apple Store region or device language is set to mainland China. :::

<Link href="https://sdk.qfapi.com/images/applepay_qrcode.png" target="_blank">
  ![Apple Pay QR Code Usage](@site/static/img/applepay_qrcode.png)
</Link>