---
id: wechat-in-app-payments
title: 微信 In-App 支付（App 內微信支付）
description: 本文件說明如何整合微信 In-App 支付（WeChat App Payment），適用於原生 App 內透過微信 SDK 完成付款流程。
sidebar_label: 微信 In-App 支付
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# 微信 In-App 支付（App 內微信支付）

<Link href="/img/wechat-in-app.png" target="_blank">![WeChat App Payment Flow](@site/static/img/wechat-in-app.png)</Link>

微信 In-App 支付適用於 **原生 App（iOS / Android）** 內的支付場景，透過微信官方 SDK 呼叫支付模組，讓使用者在 App 內直接完成付款。

---

## 前置申請與條件

* 於 **微信開放平台** 正式申請 App 支付能力
* 註冊 App 並取得對應的 **AppID**
* App 必須完成微信平台審核

詳細申請流程請參考微信官方文件：
[WeChat In-App Payment 官方說明](https://pay.weixin.qq.com/wiki/doc/api/wxpay/en/pay/In-AppPay/chapter6_2.shtml#menu1)

---

## 實名認證（選用）

商戶可選擇啟用 **微信實名認證（Real-name Verification）**。

實名認證規則：

* 若已提供身份資料，付款人微信錢包（如綁定銀行卡）需與資料一致
* 即使未綁定銀行卡，仍可完成付款
* 是否強制實名，依商戶與 PayType 開通狀態為準

---

## SDK 下載

請依平台下載並整合對應的微信 SDK：
[微信官方 SDK 下載頁](https://developers.weixin.qq.com/doc/oplatform/Downloads/iOS_Resource.html)

---

## API 呼叫說明

### HTTP 請求

* **Method**：`POST`
* **Endpoint**：`/trade/v1/payment`
* **PayType**：`800210`（微信 In-App 支付）

---

### 請求參數

| 參數名稱   | 參數編碼           | 是否必填 | 類型        | 說明                                             |
| ------ | -------------- | ---- | --------- | ---------------------------------------------- |
| 商戶 ID  | `mchid`        | 否    | String    | QFPay 於商戶入網時分配的唯一商戶識別碼                         |
| 外部訂單號  | `out_trade_no` | 是    | String    | 商戶系統內唯一的交易訂單編號                                 |
| 交易金額   | `txamt`        | 是    | Int       | 以最小幣值單位表示（例如 100 = $1），**建議金額大於 200** 以避免風控    |
| 交易幣別   | `txcurrcd`     | 是    | String(3) | 交易貨幣，請參閱[支援貨幣](/docs/api-reference/currencies) |
| 人民幣標記  | `rmb_tag`      | 否    | String(1) | 香港微信支付使用 `rmb_tag=Y` 且 `txcurrcd=CNY` 表示人民幣交易  |
| 交易時間   | `txdtm`        | 是    | String    | 格式：`YYYY-MM-DD hh:mm:ss`                       |
| 裝置識別碼  | `udid`         | 否    | String    | App 裝置唯一識別碼                                    |
| 回傳網址   | `return_url`   | 否    | String    | 付款完成後跳轉網址（部分支付方式為必填）                           |
| 客戶擴展資訊 | `extend_info`  | 否    | Object    | 實名認證資料（`user_creid`, `user_truename`），僅限中國大陸公民 |

:::note
`extend_info` 詳細格式

如需提交中國大陸用戶的實名資訊，請使用下列格式：
```json
{
  "user_creid": "430067798868676871",
  "user_truename": "\u5c0f\u6797"
}
```
`user_creid` 中包含消費者**身分證號碼**，`user_truename`中必須提供編碼形式或漢字書寫的付款人**真實姓名**。 
:::

### 請求範例

```html
{
  goods_info=test_app&goods_name=qfpay&out_trade_no=O5DNgEgL1XpvbvQSfPhN&pay_type=800210&txamt=10&txcurrcd=HKD&txdtm=2019-09-13 04:53:03&udid=AA
}
```

---

## API 回應說明

### 回應參數

| 參數名稱           | 類型         | 說明                     |
| -------------- | ---------- | ---------------------- |
| `syssn`        | String(40) | QFPay 系統產生的交易訂單編號      |
| `out_trade_no` | String     | 商戶提供的外部訂單號             |
| `txdtm`        | String     | 商戶提交的交易時間              |
| `txamt`        | Int        | 實際交易金額                 |
| `sysdtm`       | String     | QFPay 系統交易完成時間（作為清算依據） |
| `respcd`       | String(4)  | 回應代碼，`0000` 表示成功       |
| `respmsg`      | String     | 回應訊息說明                 |
| `resperr`      | String     | 錯誤描述（若有）               |
| `cardcd`       | String     | 卡號（遮罩後）                |
| `txcurrcd`     | String     | 交易貨幣                   |
| `pay_params`   | Object     | 提供給微信 SDK 的支付參數資料      |

### 回應範例

```json
{
  "sysdtm": "2019-09-13 12:53:04",
  "paydtm": "2019-09-13 12:53:04",
  "txcurrcd": "HKD",
  "respmsg": "",    
  "pay_params": 
        {
        "package": "Sign=WXPay",
        "timestamp": 1568350384,
        "sign": "XwFjohEKWdkhhT4ueg7BxeDn8tT9LcqoZYdXzifTMYyDGe3/tRchpii6vWgOn21tPSaAtqo766gvifXgDEOwR+ILKN8t97r624IJlrH0EkvSUSLh9E/cga9scXGVy0jPWHM/oVvVzJIvXew79CwZFCNTSJok2KmpSm9X9oPg7PGXbqvNMHltf+YlIOsuiz391qVmFtTE5A/cpA50+06T7iW8GYsOJQTTJed75VY+aSzNo5C6ju6WSgJKpAJJ0ocl+ONtmOp6GLVBSQXaMC4PitQcebcoP2J6fFgQ+YcPwHXasCYEnn4LaFN7zT/AjGg3E3gdCx3ksGNBOazYBRVz+g==",
        "partnerid": "316525492",
        "appid": "wx3c6896fa9b351f2a",
        "prepayid": "wx131253044253463a81dc336e1254149882",
        "noncestr": "7786db42d9a245c2b1cfc717ac59376e"
        },
  "pay_type": "800210",
  "cardcd": "",    
  "udid": "AA",
  "txdtm": "2019-09-13 04:53:03",
  "txamt": "10",
  "resperr": "交易成功",
  "out_trade_no": "O5DNgEgL1XpvbvQSfPhN",
  "syssn": "20190913152100020001567741",   
  "respcd": "0000",
  "chnlsn": ""
}
```
---

## 呼叫微信 SDK

成功取得 `pay_params` 後，商戶需依照 **微信官方 SDK 規範**，將回傳資料傳入 SDK 並觸發支付流程。

---

## 小結

* 微信 In-App 支付僅適用於 **原生 App 內場景**
* 必須先於微信開放平台完成 App 支付申請與審核
* 成功交易後，實際付款流程由微信 SDK 負責
* 建議搭配交易查詢 API 確認最終交易狀態