---
id: preauth
title: 線上預授權支付 API（Pre-authorisation）
description: 本文件說明如何使用 QFPay 線上預授權支付 API，包含預授權建立、扣款（Capture）、解凍（Unfreeze）、退款（Refund）及相關注意事項。
sidebar_label: 線上預授權支付
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# 線上預授權支付 API（Pre-authorisation）

線上預授權支付（Pre-authorisation）允許商戶**先向消費者帳戶凍結一筆資金**，在實際提供商品或服務後，再於指定期限內進行實際扣款（Capture）或解凍未使用金額（Unfreeze）。

---

## 預授權適用場景

預授權支付特別適合以下業務場景：

- **酒店 / 住宿業**：入住前凍結押金，退房後依實際消費扣款
- **租車 / 設備租賃**：取車或取件時凍結保證金
- **電商延後出貨**：訂單成立時保留金額，出貨後再扣款
- **訂閱 / 延期確認服務**：先確認付款能力，再於服務完成後扣款
- **高風險或可變金額交易**：避免事後扣款失敗造成損失

---

## 預授權扣款期限（Capture 時效）

依不同產業與渠道設定，預授權交易支援不同的**最長扣款期限**：

| 行業類型 | 最長 Capture 天數 |
|---------|----------------|
| 一般行業 | 最多 **7 天** |
| 特定行業（如酒店、租車） | 最多 **28 天** |


:::note
實際可用天數依商戶簽約設定與支付渠道為準，請於上線前向 QFPay 確認。
:::
---

## 常用 API

對接與驗證相關說明請先參考：
[API 對接總覽](/docs/api-request)

建議在開始前熟悉以下內容：

- API 憑據（AppCode / AppKey）
- 測試環境與正式環境差異
- API 簽名生成方式
- 常見錯誤碼與處理方式

相關 API：

- [交易查詢](/docs/common-api/transaction-enquiry)
- [交易退款](/docs/common-api/refund)

---

## 建立與扣款流程

![Pre-authorisation payment flow](https://www.plantuml.com/plantuml/png/XOynJWKX441xJZ6r2HUmCDzu0HihOp61mIM1WSpE57fwTv4biJ0_eHZ8UpouxOgYLelRSYIWslKB8kr1SjVSsBq_V83tJ_0gz6owDSdV51-X2tcSUpn1m33uFzmmNx2hoIc5t-b_z8sJ48s0pN72SAnafG3MPgoEcn8KIWejhOBRhVSc2Xr5CvOhw8WZd8Qxo54xlhOExjU5AcRE_0dSs8VfpVU0M_Aw-dPKhPOV)

---

### 第一步：建立預授權交易

建立預授權交易需透過 **（支付組件）Payment Element** 完成。  
此步驟會凍結消費者資金，但**不會實際扣款**。

:::note
請參閱[支付組件 (Element) SDK](/docs/online-shop/checkout-integration/payment-element)以完成前端整合。
:::
---

### 第二步：預授權扣款（Capture）

在預授權有效期限內，商戶可對已凍結的金額進行實際扣款。

> ✅ **扣款可執行多次**（累計金額不可超過預授權金額）

**URL**：`/trade/v1/authtrade`  
**Method**：`POST`

#### HTTP Header

| Header | 必填 | 說明 |
|------|------|------|
| `X-QF-APPCODE` | 是 | App Code |
| `X-QF-SIGN` | 是 | API 簽名 |

#### 請求參數

| 參數 | 必填 | 說明 |
|-----|-----|-----|
| `txamt` | 是 | 本次扣款金額（建議 > 200） |
| `txcurrcd` | 否 | 扣款幣別 |
| `mchid` | 否 | 商戶編號（僅部分渠道） |
| `syssn` | 是 | 預授權交易的系統訂單號 |

#### 回應範例

 ```json
{
"sysdtm": "2024-02-26 15:04:12",
"paydtm": "2024-02-26 15:04:12",
"udid": "qiantai2",
"txcurrcd": "HKD",
"txdtm": "2024-02-26 07:04:11",
"txamt": "500",
"resperr": "交易成功",
"respmsg": "Capture received",
"out_trade_no": "",
"syssn": "20240226180500020000014116",
"orig_syssn": "20240226180500020000014079",
"respcd": "0000",
"chnlsn": "",
"cardcd": ""
}
```

## 預授權金額解凍 (Unfreeze)

> 只限預授權未被扣款的金額。只能操作一次，操作後無法再次解凍或扣款

**API Endpoint**: `POST /trade/v1/unfreeze`

**Body**:

| 參數             | 是否必填 | 描述            |
| -------------- | ---- | ------------- |
| `txamt`        | 是    | 解凍金額          |
| `txdtm`        | 是    | 解凍時間          |
| `syssn`        | 是    | 預授權 syssn     |
| `out_trade_no` | 是    | 預授權商戶定義編號     |
| `mchid`        | 否    | 商戶 ID (限個別通道) |

### 回應範例
```json
{
"sysdtm": "2024-02-26 17:17:05",
"paydtm": "2024-02-26 17:17:06",
"udid": "qiantai2",
"txcurrcd": "HKD",
"txdtm": "2024-02-26 09:17:05",
"txamt": "2000",
"resperr": "交易成功",
"respmsg": "Void received",
"out_trade_no": "",
"syssn": "20240226180500020000014222",
"orig_syssn": "20240226180500020000014220",
"respcd": "0000",
"chnlsn": "",
"cardcd": ""
}
```

---

## 已扣款交易退款 (Refund)

預授權扣款後，如需退款，請使用退款 API。

*Refund* 操作針對 *Capture* 產生的 syssn 進行退款。

請參閱相關文件：[Refund API](/docs/common-api/refund)

---

## 當前操作的當事通知

預授權操作支援非同步同樣通知

當成功執行下列操作時，會發送 notify_url 通知：

| 操作          | notify_type 值 |
| ----------- | ------------- |
| Capture 扣款  | `payment`     |
| Unfreeze 解凍 | `unfreeze`    |
| Refund 退款   | `refund`      |

**Notify 格式範例**:

```json
{
  "status": "1",
  "pay_type": "800101",
  "sysdtm": "2020-05-14 12:32:56",
  "paydtm": "2020-05-14 12:33:56",
  "goods_name": "",
  "txcurrcd": "HKD",
  "txdtm": "2020-05-14 12:32:56",
  "mchid": "",
  "txamt": "10",
  "exchange_rate": "",
  "chnlsn2": "",
  "out_trade_no": "YEPE7WTW46NVU30JW5N90H7DHD94N56B",
  "syssn": "20200514000300020093755455",
  "cash_fee_type": "",
  "cancel": "0",
  "respcd": "0000",
  "goods_info": "",
  "cash_fee": "0",
  "notify_type": "payment",
  "chnlsn": "",
  "cardcd": ""
}
```
---

## 最佳實踐與注意事項

:::warning
若您未在有效期限內執行扣款（Capture），預授權的資金將會自動解凍。
如顧客撤回授權或帳戶餘額變動，未及時扣款可能導致資金損失。
:::

:::note
以下操作皆可透過 API 或 QFPay 商戶後台執行：
- 扣款（Capture）
- 解凍（Unfreeze）
- 退款（Refund）
:::

---