---
id: alipay-in-app
title: 支付寶 App 內支付（In-App）
description: 使用 AlipayHK 或 AlipayCN SDK 進行 App 內支付的整合指南。
sidebar_label: 支付寶 In-App
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# 支付寶 App 內支付（Alipay In-App）

本文件說明如何使用支付寶 SDK（AlipayHK 或 AlipayCN）整合 App 內支付。此支付方式適用於原生 App 環境，提供流暢且無跳轉的使用者支付體驗。

[![Alipay APP Payment process-flow](https://qfpay-8e347952.mintlify.app/img/alipay-in-app.png)](/img/alipay-in-app.png)


---

## SDK 下載

請先下載官方 SDK 套件以開始整合：

- [支付寶海外 SDK 文件](https://global.alipay.com/docs/ac/app/client_integration)
- [AlipayHK SDK 下載](https://global.alipay.com/docs/ac/app_hk/download)
- [AlipayHK SDK 觸發支付指南](https://global.alipay.com/docs/ac/hkapi/securitypay_pay)

---

## HTTP 請求

**接口位址**：`/trade/v1/payment`  
**HTTP 方法**：`POST`

| PayType  | 說明                              |
|----------|-----------------------------------|
| `801110` | 支付寶 App 內支付（海外商戶）     |
| `801510` | 支付寶 App 內支付（香港商戶）     |

---

## 必填參數

| 參數名稱        | 是否必填 | 類型        | 說明                                                               |
|------------------|----------|-------------|--------------------------------------------------------------------|
| `txamt`          | 是       | Int         | 交易金額（單位：分），例如 100 = $1                                |
| `txcurrcd`       | 是       | String(3)   | 幣別代碼（如：HKD）                                                 |
| `pay_type`       | 是       | String(6)   | Alipay CN 使用 801110，AlipayHK 使用 801510                        |
| `out_trade_no`   | 是       | String(128) | 商戶唯一交易訂單號                                                 |
| `txdtm`          | 是       | String(20)  | 交易時間格式：`YYYY-MM-DD hh:mm:ss`                                |
| `goods_name`     | 是       | String      | 商品名稱                                                           |
| `return_url`     | 是       | String      | 支付完成後的跳轉網址                                               |
| `seller_id`      | 是       | String      | 支付寶商戶帳號                                                     |
| `mchid`          | 是       | String(16)  | QFPay 指派的商戶 ID                                                |

共用欄位請參考：
[公共支付參數](/docs/api-reference/request-format#公共支付請求參數)


---

## 選填參數

| 參數名稱       | 參數編碼        | 是否必填                  | 參數類型     | 描述                                                                 |
|----------------|------------------|----------------------------|---------------|----------------------------------------------------------------------|
| 商品描述       | `goods_info`     | 否                         | String        | 支付寶必填，且不得包含特殊字符                                        |
| 支付標記       | `pay_tag`        | 否                         | String(16)    | 預設為 ALIPAYHK<br/>若為支付寶大陸版本請傳值：ALIPAYCN                |
| 訂單過期時間   | `expired_time`   | 否<br/>(僅限主掃支付)     | String(3)     | 以分鐘為單位的過期時間。預設為 30 分鐘，最小值為 5 分鐘，最大值為 120 分鐘<br/>適用於微信與支付寶 |
---

## 範例請求（Form 方式）

```plaintext
txamt=1
&txcurrcd=HKD
&pay_type=801510
&out_trade_no=052711570017898
&txdtm=2021-05-27 11:57:00
&goods_name=goods_name
&goods_info=goods_info
&mchid=nDB64h9qJ1An
&trade_name=trade_name
&goods_detail=goods_detail
&return_url=https://www.qfpay.global/
&pay_tag=ALIPAYHK
&seller_id=testoverseas9191@alipay.com
```

## 回傳欄位（pay_params）

以下欄位由 QFPay 回傳，需原樣傳入支付寶 SDK 使用：

| 參數編碼       | 二級參數編碼                    | 參數名稱              |
|----------------|----------------------------------|------------------------|
| `pay_params`   | `partner`                        | 合作夥伴              |
|                | `seller_id`                      | 收款支付寶帳號對應的支付寶唯一用戶號 |
|                | `subject`                        | 商品標題／交易標題／訂單標題／關鍵詞等 |
|                | `body`                           | 對該筆交易的具體描述；若為多件商品，請將商品描述合併傳入 body |
|                | `total_fee`                      | 訂單總金額             |
|                | `notify_url`                     | 通知位址               |
|                | `service`                        | 服務名稱               |
|                | `cardcd`                         | 卡號                   |
|                | `payment_type`                   | 支付類型               |
|                | `_input_charset`                 | 編碼格式               |
|                | `it_b_pay`                       | 自訂超時參數           |
|                | `return_url`                     | 回跳頁面目標位址       |
|                | `payment_inst`                   | 支付機構               |
|                | `currency`                       | 幣別                   |
|                | `product_code`                   | 產品代碼               |
|                | `sign`                           | RSA 簽名值（必填）     |
|                | `sign_type`                      | 簽名類型               |
|                | `secondary_merchant_id`          | 二級商戶編號           |
|                | `secondary_merchant_name`        | 二級商戶名稱           |
|                | `secondary_merchant_industry`    | 二級商戶所屬行業       |
| `chnlsn`       |                                  | 通道交易編號           |
| 常用回應參數   | —                                | —                      |


---

## QFPay 回傳範例

```json
{
  "pay_type": "801510",
  "sysdtm": "2021-05-27 11:57:02",
  "paydtm": "2021-05-27 11:57:02",
  "udid": "qiantai2",
  "txcurrcd": "HKD",
  "txdtm": "2021-05-27 11:57:00",
  "txamt": "1",
  "resperr": "交易成功",
  "respmsg": "",
  "out_trade_no": "052711570017898",
  "syssn": "20210527154100020004180921",
  "pay_params": {
    "body": "goods_info",
    "forex_biz": "FP",
    "seller_id": "2088231067382451",
    "secondary_merchant_id": "1000007081",
    "service": "mobile.securitypay.pay",
    "payment_inst": "ALIPAYHK",
    "it_b_pay": "30m",
    "secondary_merchant_name": "IFlare Hong Kong Limited (external) - online",
    "_input_charset": "UTF-8",
    "sign": "iU1yXUnsCK7rJAu0DoN61arVexbIfo3GLR5jr3QzjkZ29INSPhcA4e%2F2%2BdPrsf5huzQAkxVKP0CTfvaGPMYqNkxmhoaJWUH0ZhgYDgKugMvtweBvRqOX2W0h3A%2F%2FIdJuxeyOAuh7bHiuazSB3ZH%2BEQwRGP%2Bkk8Jpha930gHwPtw%3D",
    "currency": "HKD",
    "out_trade_no": "20210527154100020004180921",
    "payment_type": "1",
    "total_fee": 0.01,
    "sign_type": "RSA",
    "notify_url": "https://test-o2-hk.qfapi.com/trade/alipay_hk/v1/notify",
    "partner": "2088231067382451",
    "secondary_merchant_industry": "5941",
    "product_code": "NEW_WAP_OVERSEAS_SELLER",
    "return_url": "https://www.qfpay.global/",
    "subject": "goods_name"
  },
  "respcd": "0000",
  "chnlsn": "",
  "cardcd": ""
}
```
---

## 使用支付寶 SDK

取得回傳的 `pay_params` 後，需依以下格式組合成 `orderInfo` 字串：

1. 依格式拼接：`key="value"`
2. 依 key 值進行字母排序（A-Z）
3. 使用 `&` 連接
4. 將 `sign` 與 `sign_type` 放在最後

### 範例

```plaintext
_input_charset="UTF-8"&body="goods_info"&currency="HKD"&forex_biz="FP"&it_b_pay="30m"&notify_url="https://test-o2-hk.qfapi.com/trade/alipay_hk/v1/notify"&out_trade_no="20210527154100020004180921"&partner="2088231067382451"&payment_inst="ALIPAYHK"&payment_type="1"&product_code="NEW_WAP_OVERSEAS_SELLER"&return_url="https://www.qfpay.global/"&secondary_merchant_id="1000007081"&secondary_merchant_industry="5941"&secondary_merchant_name="IFlare Hong Kong Limited (external) - online"&seller_id="2088231067382451"&service="mobile.securitypay.pay"&subject="goods_name"&total_fee="0.01"&sign="iU1yXUnsCK7rJAu0DoN61arVexbIfo3GLR5jr3QzjkZ29INSPhcA4e%2F2%2BdPrsf5huzQAkxVKP0CTfvaGPMYqNkxmhoaJWUH0ZhgYDgKugMvtweBvRqOX2W0h3A%2F%2FIdJuxeyOAuh7bHiuazSB3ZH%2BEQwRGP%2Bkk8Jpha930gHwPtw%3D"&sign_type="RSA"

```

:::note
請務必注意：
- 使用正確地區對應的 SDK（HK 或 CN）
- `pay_params` 內的所有 `key` 與 `value` 必須完全一致
- 簽名邏輯需與支付寶 SDK 規範完全相符
:::