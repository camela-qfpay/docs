---
title: "Visa / Mastercard 線上支付"
---

# Visa / Mastercard 線上支付

本頁提供使用 Visa 與 Mastercard 進行線上信用卡支付的整合說明。我們目前於 [香港環境](/docs/preparation/environments) 支援所有主要發卡機構的信用卡交易。

---

## 整合方式選擇

商戶可根據需求選擇以下其中一種整合方式：

1. [**託管結帳頁面（收銀台）**](/docs/online-shop/checkout-integration/checkout)\
   託管式支付頁方案，適合希望降低 PCI 負擔並快速上線的商戶。
2. [**支付組件 (Element) SDK**](/docs/online-shop/checkout-integration/payment-element)\
   客戶端嵌入式 SDK，將輸入欄位直接嵌入商戶網頁中，提供完全的 UX 控制與 3DS 支援。

---

## 非同步通知機制

:::info QFPay 將透過非同步通知方式將交易結果傳送至商戶後台。 :::

詳情請參閱 [非同步通知說明文件](/docs/common-api/async-notifications)，了解通知格式與簽名驗證方式。

> 範例通知資料：

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

:::warning 請務必驗證通知簽名，避免依賴未驗證的交易結果進行業務處理。 :::

:::tip 如出現以下情況，建議使用 [**交易查詢**](/docs/common-api/transaction-enquiry) API 作為補充確認手段：

- 未收到回調通知
- 通知延遲
- 簽名驗證失敗 :::

---

## 測試卡資訊

以下測試卡號可於 **Sandbox 測試環境** 中使用，模擬各類交易結果（包括 3D Secure 驗證流程）：

| Card Brand | Card Number         | Simulation Result |
| ---------- | ------------------- | ----------------- |
| Mastercard | 5200 0000 0000 1096 | 成功付款              |
| Visa       | 4000 0000 0000 1091 | 成功付款              |
| Mastercard | 5200 0000 0000 1005 | 成功（3DS 無干預流程）     |
| Visa       | 4000 0000 0000 1000 | 成功（3DS 無干預流程）     |
| Mastercard | 5200 0000 0000 1120 | 驗證階段失敗            |
| Visa       | 4000 0000 0000 1125 | 驗證階段失敗            |
| Mastercard | 5200 0000 0000 1013 | 3DS 無干預流程中失敗      |
| Visa       | 4000 0000 0000 1018 | 3DS 無干預流程中失敗      |

:::tip 若不確定應採用 Checkout 或 Element SDK 整合方式，請參閱您的上線文件，或聯繫 QFPay 支援團隊協助評估。 :::