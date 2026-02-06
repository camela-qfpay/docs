---
id: customs-declaration
title: 報關 API 指南
description: 提交跨境電商交易資訊至支付寶或微信的報關系統。
sidebar_label: 報關 API
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# 報關 API 指南

透過報關 API，商戶可將跨境交易資訊自動提交至支付寶或微信對應的海關系統，協助完成合規要求，加快清關速度，並提升用戶體驗。

---

## 1. 發起報關申請

此接口用於支付成功後提交報關資訊。

### HTTP 請求

`POST ../custom/v1/declare`

### 請求參數

| 參數名稱 | 必填 | 類型 | 說明 |
|----------|------|------|------|
| `trade_type` | 是 | String(8) | 支付平台類型，取值：`weixin` 或 `alipay` |
| `syssn` | 是 | String(32) | QFPay 交易流水號 |
| `customs` | 是 | String(20) | 報關地海關代碼，例如：`SHANGHAI_ZS` |
| `mch_customs_no` | 是 | String(20) | 商戶海關登記編號 |
| `action_type` | 否 | String(256) | 報關動作（僅適用於微信）：`"ADD"` 新增，`"MODIFY"` 修改 |
| `mch_customs_name` | 否 | String(256) | 支付寶報關登記商戶名稱，例如：`jwyhanguo_card` |
| `out_request_no` | 否 | String(32) | 商戶端報關請求流水號（支付寶專用） |
| `amount` | 否 | String(20) | 報關金額（支付寶專用），例如 `2.00` |

### 子訂單參數（拆單或修改報關時使用）

| 參數名稱 | 條件必填 | 類型 | 說明 |
|----------|----------|------|------|
| `sub_order_no` | 條件 | String(64) | 若為子訂單，需填寫子訂單編號 |
| `fee_type` | 條件 | String(8) | 幣別（微信僅支持 `CNY`） |
| `order_fee` | 條件 | String(8) | 子訂單總金額（單位為分）= `transport_fee` + `product_fee` |
| `product_fee` | 條件 | String(8) | 商品金額（分） |
| `transport_fee` | 條件 | String(8) | 運費金額（分） |

### 回應參數

| 參數名稱 | 類型 | 說明 |
|----------|------|------|
| `syssn` | String(40) | QFPay 交易流水號 |
| `respcd` | String(4) | 回應碼：`0000` 成功；`1143/1145` 需重查；其他表示失敗 |
| `resperr` | String(128) | 錯誤描述 |
| `respmsg` | String(128) | 附加訊息 |
| `verify_department` | String | 海關受理部門 |
| `verify_department_trade_id` | String | 海關返回的交易編號 |

---

## 2. 查詢報關結果

可查詢報關是否成功處理，是否被海關受理等狀態。

### HTTP 請求

`POST/GET ../custom/v1/query`

### 請求參數

| 參數名稱 | 必填 | 類型 | 說明 |
|----------|------|------|------|
| `trade_type` | 是 | String(8) | 支付平台類型，取值：`weixin` 或 `alipay` |
| `customs` | 是 | String(20) | 海關代碼，例如：`SHANGHAI_ZS` |
| `syssn` | 是 | String(32) | QFPay 交易流水號 |
| `sub_order_no` | 否 | String(40) | 若為子訂單，需提供子訂單編號 |

### 回應參數

| 參數名稱 | 類型 | 說明 |
|----------|------|------|
| `syssn` | String(40) | QFPay 交易流水號 |
| `respcd` | String(4) | 回應碼 |
| `resperr` | String(128) | 錯誤訊息 |
| `respmsg` | String(128) | 附加訊息 |
| `data` | Array | 報關資料陣列，包含：`resperr`、`errmsg`、`sub_order_no`、`verify_department`、`verify_department_trade_id` 等欄位 |

---

## 3. 重新發送報關資料

若海關端未收到報關資訊，可使用此接口重新推送。

### HTTP 請求

`POST ../custom/v1/redeclare`

### 請求參數

| 參數名稱 | 必填 | 類型 | 說明 |
|----------|------|------|------|
| `trade_type` | 是 | String(8) | 支付平台類型，取值：`weixin` 或 `alipay` |
| `customs` | 是 | String(20) | 海關代碼 |
| `syssn` | 是 | String(32) | QFPay 交易流水號 |
| `mch_customs_no` | 是 | String(20) | 商戶海關登記編號 |
| `sub_order_no` | 否 | String(40) | 拆單時需提供子訂單編號 |

### 回應格式

與 [發起報關申請](#1-發起報關申請) 相同。

---

## 注意事項

:::tip
- 僅可對已完成的成功交易（`respcd`=`0000`）進行報關。
- 支付寶報關需確認「報關商戶名稱」與「報關編號」準確無誤。
- 微信支付若需拆單，應提供正確的商品金額與運費欄位。
- 各地區報關合規要求不同，請依所在地法規準備報關資料。
:::

完整回應碼請參考：[交易狀態碼](/docs/api-reference/status-codes)