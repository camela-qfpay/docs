---
title: 訂閱支付 Webhook（定期付款通知）
description: 本文件說明 QFPay 訂閱支付（定期付款）的 Webhook 非同步通知機制，包含支付令牌建立、訂閱狀態變更，以及定期扣款結果通知。
---

# 訂閱支付 Webhook（定期付款通知）

本頁說明 QFPay 如何針對訂閱支付系統發送非同步通知（Webhook），涵蓋以下事件類型：

- 支付令牌（Token）建立結果
- 訂閱狀態變更
- 定期扣款（訂閱支付）結果

## 如何設定通知 URL

<Note>
如需設定 Webhook 接收端點，請將以下資訊寄送至 `technical.support@qfpay.com`：
- Webhook URL
- 商戶 ID
- 門店 ID
</Note>

---

## 支付令牌建立通知（Token Creation Notification）

當透過 Element SDK 成功建立支付令牌後，系統會發送此通知。

### 通知欄位說明

| 欄位名稱                | 說明                              |
| ------------------- | ------------------------------- |
| `userid`            | QFPay 門店 ID                     |
| `notify_type`       | 通知類型，固定為 `payment_token`        |
| `event`             | 令牌事件類型：`NEW`、`MATCH`、`CONFLICT` |
| `tokenid`           | 建立完成的支付令牌 ID                    |
| `token_expiry_date` | 支付令牌的到期時間                       |
| `cardcd`            | 遮罩後的卡號                          |
| `card_scheme`       | 卡組織，例如 VISA、MASTERCARD          |
| `respcd`            | 回應碼，`0000` 表示成功                 |
| `respmsg`           | 回應訊息，例如 `success`               |
| `sysdtm`            | 事件發生的系統時間                       |
| `customer_id`       | 若已關聯，則為對應的 Customer ID          |
| `token_reason`      | 令牌化原因                           |
| `token_reference`   | 系統內部的令牌參考值                      |

### 範例

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

## 訂閱狀態變更通知

當訂閱狀態發生變更時，系統會發送此通知。

### 通知欄位說明

| 欄位名稱              | 說明                                          |
| ----------------- | ------------------------------------------- |
| `notify_type`     | 通知類型，固定為 `subscription`                     |
| `subscription_id` | 訂閱物件的唯一識別碼                                  |
| `state`           | 訂閱目前狀態（如 `ACTIVE`、`COMPLETED`、`INCOMPLETE`） |
| `sysdtm`          | 狀態變更的系統時間                                   |


### 範例

```json
{
  "state": "COMPLETED",
  "sysdtm": "2024-04-24 15:19:39",
  "notify_type": "subscription",
  "subscription_id": "sub_e51bb914919*****f6b0fe36d"
}
```

---

## 訂閱扣款結果通知（Subscription Payment Result）

每次定期扣款嘗試（成功或失敗）後，系統都會發送此通知。

### 通知欄位說明

| 欄位名稱                    | 說明                                             |
| ----------------------- | ---------------------------------------------- |
| `notify_type`           | 通知類型，固定為 `subscription_payment`                |
| `subscription_id`       | 訂閱 ID                                          |
| `subscription_order_id` | 扣款訂單 ID，格式為 `sub_ord_{subscription_id}_{0001}` |
| `respcd`                | 回應碼，`0000` 表示成功                                |
| `respmsg`               | 回應訊息                                           |
| `syssn`                 | 系統交易流水號                                        |
| `txdtm`                 | 交易時間                                           |
| `txamt`                 | 交易金額                                           |
| `txcurrcd`              | 交易幣別                                           |
| `customer_id`           | Customer ID                                    |
| `product_id`            | Product ID，多筆時以逗號分隔                            |
| `cardcd`                | 遮罩後卡號                                          |
| `card_scheme`           | 卡組織（僅在成功時提供）                                   |
| `current_iteration`     | 此訂閱目前的扣款次數                                     |


### 範例

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

## 範例

<Tip>
請確保您的 Webhook 接收端具備**冪等性（Idempotency）**設計。
在通知傳送失敗時，相同事件可能會被重複發送。
</Tip>

若接收端未回傳 **HTTP 200 OK**，系統將自動重試通知。

重試採用 **指數退避（Exponential Backoff）** 機制，實際最大重試次數可能依內部設定而有所調整。
