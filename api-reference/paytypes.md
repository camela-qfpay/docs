---
id: "paytypes"
title: "Payment Types"
sidebar_label: "Payment Types"
---

The `pay_type` parameter specifies which payment method is used to process a transaction. Each value represents a distinct payment channel, such as Alipay, WeChat Pay, or a card scheme. The correct code must be passed in the request body when calling `/trade/v1/payment` or similar endpoints.

This value directly affects how the transaction is routed, and whether specific UI or authorisation is required (e.g., scanning a QR code, invoking a native wallet, etc.).

### Where to use

Include `pay_type` in your payment request payload. For example:

```json
{
  "pay_type": "800101",
  "txamt": "1000",
  ...
}
```

<Note>
  Not all `pay_type` values are enabled for every merchant. Please refer to your onboarding configuration or contact [technical.support@qfpay.com](mailto:technical.support@qfpay.com) for clarification.
</Note>

## Supported Payment Types

| Code   | Description                                                                                     |
| ------ | ----------------------------------------------------------------------------------------------- |
| 800008 | Consumer Present QR Code Mode (CPM) for WeChat, Alipay, UNIONPAY Quick Pass, PayMe              |
| 800101 | Alipay Merchant Presented QR Code Payment in store (MPM) (Overseas Merchants)                   |
| 800108 | Alipay Consumer Presented QR Code Payment (CPM) (Overseas & HK Merchants)                       |
| 801101 | Alipay Online WEB (in browser Chrome etc.) Payment (Overseas Merchants)                         |
| 801107 | Alipay Online WAP (in mobile browser Chrome etc.) Payment (Overseas Merchants)                  |
| 801110 | Alipay in-APP Payments (Overseas Merchants)                                                     |
| 801501 | Alipay Merchant Presented QR Code (MPM) Payment (HK Merchants)                                  |
| 801510 | Alipay In-App Payment (HK Merchants)                                                            |
| 801512 | Alipay Online WAP Payment (HK Merchants)                                                        |
| 801514 | Alipay Online WEB Payment (HK Merchants)                                                        |
| 800201 | WeChat Merchant Presented QR Code Payment (MPM) (Overseas & HK Merchants)                       |
| 800208 | WeChat Consumer Presented QR Code Payment (CPM) (Overseas & HK Merchants)                       |
| 800207 | WeChat JSAPI Payment - WeChat Official Account Payment (in Wechat App)(Overseas & HK Merchants) |
| 800212 | WeChat H5 Payment (In mobile browser)                                                           |
| 800210 | WeChat in-APP Payment (Overseas & HK Merchants)                                                 |
| 800213 | WeChat Mini-Program Payment (Overseas & HK Merchants)                                           |
| 801008 | WeChat Pay HK Consumer Presented QR Code Payment (CPM) (Direct Settlement, HK Merchants)        |
| 801010 | WeChat Pay HK In-App Payment (Direct Settlement, HK Merchants)                                  |
| 805801 | PayMe Merchant Presented QR Code Payment in store (MPM) (HK Merchants)                          |
| 805808 | PayMe Consumer Presented QR Code Payment (CPM) (HK Merchants)                                   |
| 805814 | PayMe Online WEB (in browser Chrome etc.) Payment (HK Merchants)                                |
| 805812 | PayMe Online WAP (in mobile browser Chrome etc.) Payment (HK Merchants)                         |
| 800701 | UNIONPAY Quick Pass Merchant Presented QR Code Payment (MPM)                                    |
| 800708 | UNIONPAY Quick Pass Consumer Presented QR Code Payment (CPM)                                    |
| 800712 | UNIONPAY WAP Payment (HK Merchants)                                                             |
| 800714 | UNIONPAY PC-Web Payment (HK Merchants)                                                          |
| 802001 | FPS Merchant Presented QR Code Payment (MPM) (HK Merchants)                                     |
| 803701 | Octopus dynamic QRC Payment - Merchant Present Mode (MPM) (HK Merchants)                        |
| 803712 | Octopus WAP Payment (HK Merchants)                                                              |
| 803714 | Octopus PC-Web Payment (HK Merchants)                                                           |
| 802801 | Visa / Mastercard Online Payments                                                               |
| 802808 | Visa / Mastercard Offline Payments                                                              |
| 806527 | ApplePay Online Payments                                                                        |
| 806708 | UnionPay Card Offline Payments                                                                  |
| 806808 | American Express Card Offline Payments                                                          |

{/* 
804101 | Maybank dynamic QRC Payment - Merchant Present Mode (MPM) (MY Merchants)
804108 | Maybank dynamic QRC Payment - Consumer Present Mode (CPM) (MY Merchants)
804114 | Maybank Online Payment (MY Merchants)**
804201 | GrabPay dynamic QRC Payment - Merchant Present Mode (MPM) (MY and SG Merchants)
804208 | GrabPay dynamic QRC Payment - Consumer Present Mode (CPM) (MY Merchants)
804214 | GrabPay Online Payment (MY Merchant)** (SG Merchants)*
803801 | Dash dynamic QRC Payment - Merchant Present Mode (MPM) (SG Merchants)
803808 | Dash dynamic QRC Payment - Consumer Present Mode (CPM) (SG Merchants)
804001 | Boost dynamic QRC Payment - Merchant Present Mode (MPM) (MY Merchants)
804008 | Boost dynamic QRC Payment - Consumer Present Mode (CPM) (MY Merchants)
804014 | Boost Online Payment (MY Merchants)**
805208 | TrueMoney dynamic QRC Payment - Consumer Present Mode (CPM) (TH Merchants)
805401 | ThaiQR dynamic QRC Payment - Merchant Present Mode (MPM) (SG and MY Merchants)***
803101 | VIA dynamic QRC Payment - Merchant Present Mode (MPM) (JP and HK Merchants)
803108 | VIA dynamic QRC Payment - Consumer Present Mode (CPM) (JP and HK Merchants)
803208 | Touch 'n Go (TNG) dynamic QRC Payment - Consumer Present Mode (CPM) (MY Merchants)
803214 | Touch 'n Go (TNG) Online Payment (MY Merchants)**
803301 | Razer dynamic QRC Payment - Merchant Present Mode (MPM) (MY Merchants)
803308 | Razer dynamic QRC Payment - Consumer Present Mode (CPM) (MY Merchants)
803314 | Razer Online Payment ** (MY Merchants)
802201 | AIRPAY Online Payment (TH Merchants)
802301 | PayNow Merchant Presented QR Code Payment (MPM) (SG Merchants)***
802901 | PromptPay dynamic QRC Payment - Merchant Present Mode (MPM) (TH Merchants)***
801908 | Origami Consumer Presented QR Code Payment (CPM)
801208 | LINEPAY dynamic QRC Payment - Consumer Present Mode (CPM) (TH Merchants)
801301 | LINEPAY Online Payment (TH Merchants)
801408 | AIRPAY dynamic QRC Payment - Consumer Present Mode (CPM) (TH Merchants)
801701 | NETSPAY Merchant Presented QR Code Payment (MPM)
805601 | GoPay dynamic QRC Payment - Merchant Present Mode (MPM)***
805612 | GoPay WAP Payment***
800710 | UNIONPAY In-App Payment (HK Merchants)
801801 | Alipay Pre-Authorization dynamic QRC Payment - Consumer Present Mode (CPM)
801808 | Alipay Pre-Authorization dynamic QRC Payment - Merchant Present Mode (MPM)  
801810 | Alipay Pre-Authorization in-APP Payment
801814 | Alipay Pre-Authorization Online Payment
803001 | eWallet dynamic QRC Payment - Merchant Present Mode (MPM)
803008 | eWallet dynamic QRC Payment - Consumer Present Mode (CPM)
805508 | Credit Card: first_data Quick Payment Mode (HK Merchant)
805514 | Credit Card: first_data Security Verification Payment Mode (HK Merchants)
text 
*/}

## Special Remarks

{/*
- ****: `return_url` is a **mandatory** request parameter (unlike public payment parameters).  
Also note that the response field `web_url` contains the redirect payment URL.
*/}

- **801101**: Transaction amount must be **greater than 1 HKD**.
- **802001**:  This payment method **does not support refunds**.

{/*
- ****:  For complete request and response details, please refer to the [API Reference section](https://sdk.qfapi.com/docs/online-shop/alipay/alipay-web-payments).
*/}