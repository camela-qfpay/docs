---
id: recurring-overview
title: 定期付款概覽
description: 簡介 QFPay 定期付款（訂閱支付）系統，包括使用情境、系統架構與關鍵組件。
sidebar_label: 概覽
-----------------

# 定期付款概覽

定期付款（Recurring Payments，又稱訂閱支付）讓商戶可依固定週期，自動向客戶收取費用，並以事先儲存的支付令牌（token）完成扣款。這類流程非常適合會籍收費、SaaS 軟體計費、公用事業帳單等需要周期性收費的業務。

---

## 什麼是訂閱支付？

訂閱支付是一種協議，允許商戶依固定週期（例如每月）向客戶收費，可為固定或變動金額。

在 QFPay 系統中，訂閱支付由 `subscription` 物件表示，並連結以下元素：

* `customer_id`：被收費的最終用戶
* `token_id`：儲存的卡片或付款方式
* `product_id`：定義收費金額與頻率

---

## 關鍵概念與對應物件

| 物件                | 說明                                                                                   |
| ----------------- | ------------------------------------------------------------------------------------ |
| `customer_id`     | 儲存用於定期付款的用戶資訊（如 email、電話）                                                            |
| `token_id`        | 通過 [Payment Element](/docs/online-shop/checkout-integration/payment-element) 儲存的支付令牌 |
| `product_id`      | 定義收費金額、頻率（月繳、年繳）與幣別                                                                  |
| `subscription_id` | 訂閱記錄，整合上述三項，並控制整個收費週期                                                                |

---

## 訂閱支付全流程

### 第一步：建立客戶

使用 `/customer/v1/create` 註冊用戶資料。

### 第二步：生成支付令牌

透過 [Payment Element](/docs/online-shop/checkout-integration/payment-element) 收集卡資料，獲得 `token_id`。

### 第三步：建立產品（Product）

使用 `/product/v1/create` 定義收費金額與頻率，例如每月 HKD 100。

### 第四步：建立訂閱（Subscription）

使用 `/subscription/v1/create`，傳入 `customer_id`、`product_id` 與 `token_id`。

### 第五步：自動或手動扣款

QFPay 系統會依排程自動扣款。若付款失敗，可透過 `/subscription/v1/charge` 進行手動補扣。

---

## 訂閱狀態列表

| 狀態           | 說明               |
| ------------ | ---------------- |
| `ACTIVE`     | 訂閱有效並正常排程扣款      |
| `INCOMPLETE` | 訂閱設定失敗或令牌無效      |
| `PAST_DUE`   | 曾有一次或多次扣款失敗      |
| `CANCELLED`  | 訂閱已取消（由商戶或系統觸發）  |
| `COMPLETED`  | 訂閱已完成所有扣款週期並自動結束 |

若需接收訂閱變更，請整合 [Webhook 通知](/docs/online-shop/integration-by-use-case/recurring-payments/recurring-webhook)。

---

## 使用情境範例

* **健身會籍**：每月自動扣 HKD 300
* **SaaS 軟體訂閱**：每月 HKD 98 或每年 HKD 999
* **設備租賃**：限期重複付款，支援補扣機制

---

## 常見問題 FAQ

### 可以更新訂閱的支付方式嗎？

可以。使用 `/subscription/v1/update` 替換新的 `token_id` 即可。

### 可以暫停訂閱嗎？

目前不支援暫停。可先取消，之後重新建立訂閱。

### 可以補扣失敗的款項嗎？

可以。使用 `/subscription/v1/charge` 搭配失敗的 `subscription_order_id` 即可。

---

## 下一步

* [API 參考文件](/docs/online-shop/integration-by-use-case/recurring-payments/recurring-api-reference)
* [Webhook 通知整合](/docs/online-shop/integration-by-use-case/recurring-payments/recurring-webhook)
