---

id: recurring-api-reference
title: 訂閱支付 API 參考
description: 本頁列出創建、更新和管理定期付款 (訂閱支付) 相關資源的 API 信息
sidebar_label: 訂閱 API 參考
------------------------

# 訂閱 API 參考

本頁記錄了定期付款所有可使用的 API 連線。使用前請先閱讀 [定期付款概覽](/docs/online-shop/integration-by-use-case/recurring-payments/recurring-overview)。

---

## 1. 創建 Customer

**POST** `/customer/v1/create`

創建一個新用戶資料。

### 請求參數

| 參數             | 類型   | 必填 | 說明                     |
|------------------|--------|------|--------------------------|
| `customer_id`    | String | 是   | 顧客 ID                 |
| `name`           | String | 否   | 姓名                     |
| `phone`          | String | 否   | 電話                     |
| `email`          | String | 否   | 電郵地址                 |
| `billing_address`| String (JSON)   | 否   | 帳單地址 JSON 物件      |

### 回應

返回 `customer_id`。

---

## 2. 更新 Customer

**POST** `/customer/v1/update`

更新現有用戶資料。

### 請求參數

| 參數             | 類型   | 必填 | 說明                     |
|------------------|--------|------|--------------------------|
| `customer_id`    | String | 是   | 顧客 ID                 |
| `name`           | String | 否   | 姓名                     |
| `phone`          | String | 否   | 電話                     |
| `email`          | String | 否   | 電郵地址                 |
| `billing_address`| JSON   | 否   | 帳單地址 JSON 物件      |

### 回應

返回更新記錄數。

---

## 3. 查詢 Customer

**POST** `/customer/v1/query`

查詢用戶資料。

### 請求參數

| 參數         | 類型   | 必填 | 說明               |
|--------------|--------|------|--------------------|
| `customer_id`| String | 否   | 顧客 ID           |
| `name`       | String | 否   | 姓名               |
| `phone`      | String | 否   | 聯絡電話           |
| `email`      | String | 否   | 電郵               |
| `page`       | Int    | 否   | 頁碼，預設為 1     |
| `page_size`  | Int    | 否   | 每頁筆數，預設為 10，最多 100 |

### 回應

返回符合條件的 customer 列表。

---

## 4. 刪除 Customer

**POST** `/customer/v1/delete`

刪除指定用戶。此操作無法還原。

### 請求參數

| 參數         | 類型   | 必填 | 說明         |
|--------------|--------|------|--------------|
| `customer_id`| String | 是   | 要刪除的顧客 ID |

### 回應

返回被刪除的 ID 和結果。

---

## 5. 創建 Product

**POST** `/product/v1/create`

創建定期購買產品。

### 請求參數

| 參數             | 類型   | 必填 | 說明                                                  |
|------------------|--------|------|-------------------------------------------------------|
| `name`           | String | 是   | 顯示給顧客的產品名稱                                 |
| `type`           | String | 是   | 固定為 `recurring`                                    |
| `description`    | String | 否   | 產品描述                                              |
| `txamt`          | Int    | 是   | 金額，單位為分（例如 $1 = 100）                      |
| `txcurrcd`       | String | 是   | 幣別，如 HKD                                          |
| `interval`       | String | 是   | 收費週期，如 `monthly`、`yearly` 等                  |
| `interval_count` | Int    | 是   | 週期間隔數，例如 1 = 每月，12 = 每年                 |
| `usage_type`     | String | 是   | 固定為 `licensed`                                     |

:::note
在 sandbox 環境中，`interval` 支援 `minutes`、`hours`用於測試
:::


### 回應

返回新創產品 `product_id`。

---

## 6. 更新 Product

**POST** `/product/v1/update`

更新現有產品資料。

### 請求參數

| 參數          | 類型   | 必填 | 說明                     |
|---------------|--------|------|--------------------------|
| `product_id`  | String | 是   | 要更新的產品 ID          |
| `name`        | String | 否   | 新的產品名稱             |
| `description` | String | 否   | 新的產品描述             |

### 回應

返回更新記錄數。

---

## 7. 查詢 Product

**POST** `/product/v1/query`

查詢定期購買產品。

### 請求參數

| 參數          | 類型   | 必填 | 說明                              |
|---------------|--------|------|-----------------------------------|
| `product_id`  | String | 否   | 產品 ID                           |
| `name`        | String | 否   | 產品名稱                          |
| `description` | String | 否   | 產品描述                          |
| `txcurrcd`    | String | 否   | 交易幣別                          |
| `interval`    | String | 否   | 收費週期（`monthly` / `yearly`）     |
| `page`        | Int    | 否   | 頁碼，預設為 1                    |
| `page_size`   | Int    | 否   | 每頁筆數，預設 10，最大 100       |

### 回應

返回符合條件的 product 列表。

---

## 8. 刪除 Product

**POST** `/product/v1/delete`

刪除指定產品 (需不被訂閱關聯)。

### 請求參數

| 參數         | 類型   | 必填 | 說明                                       |
|--------------|--------|------|--------------------------------------------|
| `product_id` | String | 是   | 要刪除的產品 ID（不可被任何訂閱使用中）   |

### 回應

返回被刪除的 ID 和結果。

---

## 9. 創建 Subscription

:::tip
預先確保已創建 `customer_id`、`product_id`、`token_id`
:::

**POST** `/subscription/v1/create`

創建新訂閱計劃

### 請求參數

| 參數                    | 類型   | 必填 | 說明                                                   |
|-------------------------|--------|------|--------------------------------------------------------|
| `customer_id`           | String | 是   | 顧客 ID                                                |
| `token_id`              | String | 是   | 支付令牌 ID                                            |
| `products`              | List   | 是   | 產品清單（product_id + quantity）                     |
| `total_billing_cycles`  | Int    | 否   | 總扣款次數，若為 null 則表示無限期                     |
| `start_time`            | String | 否   | 訂閱開始時間（ISO 格式），亦為首次扣款時間             |

### products 內部參數

| 欄位名稱     | 類型   | 必填 | 說明                                   |
|--------------|--------|------|----------------------------------------|
| `product_id` | String | 是   | QFPay 系統中的唯一產品 ID              |
| `quantity`   | Int    | 否   | 數量，預設為 1                         |

### 範例 request

```json
{
  "products": [
    {
      "product_id": "prod_54c3772d******9a54b236e09ec74f",
      "quantity": 1
    }
  ],
  "customer_id": "cust_aaf6aae94******982c54c9cae5ba32",
  "token_id": "tk_a99892fd*********d3417d168a18bb",
  "total_billing_cycles": 2,
  "start_time": "2020-05-14 12:32:56"
}
```

### 回應參數（`data` 欄位）

| 欄位名稱        | 類型   | 說明                                                   |
|-----------------|--------|--------------------------------------------------------|
| `subscription_id` | String | 訂閱 ID，例如 `sub_xxxxxxxx`                           |
| `state`           | String | 初始狀態：`ACTIVE` / `INCOMPLETE` / `COMPLETED`              |

---

## 10. 更新 Subscription

**POST** `/subscription/v1/update`

更新現有訂閱項目

### 請求參數

| 參數                    | 類型   | 必填 | 說明                                                     |
|-------------------------|--------|------|----------------------------------------------------------|
| `subscription_id`       | String | 是   | 訂閱 ID                                                  |
| `total_billing_cycles`  | Int    | 否   | 總扣款次數（null = 無限）                                |
| `start_time`            | String | 否   | 訂閱開始時間（影響首次扣款）                             |
| `token_id`              | String | 否   | 新的支付令牌 ID                                          |
| `products`              | Object | 否   | 產品清單（`product_id` + `quantity`）                       |

### 回應參數（`data` 欄位）

| 欄位名稱        | 類型 | 說明                 |
|-----------------|------|----------------------|
| `subscription_id` | String | 訂閱 ID              |
| `rowAffected`     | Int    | 被更新的筆數         |

---

## 11. 查詢 Subscription

查詢訂閱對象。

**POST** `/subscription/v1/query`

### 請求參數

| 參數            | 類型   | 必填 | 說明                                      |
|-----------------|--------|------|-------------------------------------------|
| `page`          | Int    | 否   | 頁碼，預設為 1                            |
| `page_size`     | Int    | 否   | 每頁筆數，預設 10，最大 100               |
| `subscription_id` | String | 否 | 訂閱 ID                                  |
| `customer_id`   | String | 否   | 顧客 ID                                  |
| `state`         | String | 否   | 訂閱狀態（`ACTIVE` / `INCOMPLETE` / 等）      |

### 回應

| 欄位名稱                     | 類型   | 說明                                   |
|------------------------------|--------|----------------------------------------|
| `subscription_id`            | String | 訂閱 ID                                |
| `customer_id`                | String | 顧客 ID                                |
| `token_id`                   | String | 支付令牌 ID                            |
| `products`                   | Object | 產品清單                               |
| `total_billing_cycles`       | Int    | 總扣款次數（null = 無限）              |
| `state`                      | String | 訂閱狀態                               |
| `next_billing_time`          | String | 下次扣款時間                           |
| `last_billing_time`          | String | 上次扣款時間                           |
| `completed_billing_iteration`| Int    | 已完成扣款次數                         |
| `start_time`                 | String | 訂閱開始時間                           |

---

## 12. 取消 Subscription

立即取消訂閱。

**POST** `/subscription/v1/cancel`

### 請求參數

| 參數            | 類型   | 必填 | 說明               |
|-----------------|--------|------|--------------------|
| `subscription_id` | String | 是   | 要取消的訂閱 ID    |

---

## 13. 查詢訂閱按期購買記錄

查詢特定訂閱下的所有扣款訂單。

**POST** `/subscription/billing_order/v1/list`

### 請求參數

| 參數            | 類型   | 必填 | 說明                            |
|-----------------|--------|------|---------------------------------|
| `subscription_id` | String | 是   | 所屬訂閱的 ID                    |
| `page`          | Int    | 否   | 頁碼，預設為 1                   |
| `page_size`     | Int    | 否   | 每頁筆數，預設 10，最大 100     |

### 回應參數（`data` 欄位）

包含以下屬性的訂閱對象陣列：

| 欄位名稱               | 類型   | 說明                                                                                         |
|------------------------|--------|----------------------------------------------------------------------------------------------|
| `subscription_order_id`| String | 扣款訂單 ID，格式：`sub_ord_{訂閱ID}_{0001}` 表示訂閱的第 1 次扣款                            |
| `subscription_id`      | String | 訂閱 ID                                                                                      |
| `trigger_by`           | String | 觸發類型：`auto`（系統自動）或 `manual`（手動）                                               |
| `sequence_no`          | Int    | 此筆訂單為該訂閱的第幾次扣款                                                                  |

---

## 14. 手動重新訂閱收款

對失敗的訂閱訂單發起手動扣款。

**POST** `/subscription/v1/charge`

### 請求參數

| 參數                  | 類型   | 必填 | 說明                                                           |
|-----------------------|--------|------|----------------------------------------------------------------|
| `subscription_id`     | String | 是   | 訂閱 ID                                                        |
| `subscription_order_id` | String | 否 | （可選）特定要重試的訂單 ID，格式與上方相同                   |


:::note
本 API 只適用於不成功訂閱按期購買事件，且狀態為 `UNPAID`、`INCOMPLETE`、`PAST_DUE`。
	•	如果重試日期 於下一次支付日前，訂閱繼續按計劃進行
	•	如果在**下一次日期**之後，訂閱將被**取消**
	•	如重試失敗，狀態保持**不變**
:::