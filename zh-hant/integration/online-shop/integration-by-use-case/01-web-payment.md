---
id: web-payment
title: 網頁支付（Web Payment）
description: 本文件說明如何透過 API 整合微信、支付寶、PayMe、銀聯等網頁支付流程。
sidebar_label: Web 支付
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# Web 支付（線上網頁付款）

## 簡介

Web 支付允許用戶在商戶網站中直接使用第三方電子錢包付款。用戶可掃描二維碼、確認金額並完成付款。支付完成後，透過 `return_url` 參數將用戶導回商戶指定頁面。

目前支持的錢包支付將即時計價為港幣（HKD）從用戶電子錢包扣款，QFPay 亦會以港幣結算給商戶。

## HTTP 請求

`POST /trade/v1/payment`

| 支付類型   | 說明                                                                                                        |
| ------ | --------------------------------------------------------------------------------------------------------- |
| 800201 | 微信 Web 掃碼支付（香港商戶），詳見[微信掃碼支付](/docs/online-shop/integration-by-payment-type/wechat/wechat-web-qr-payment)  |
| 801514 | 支付寶 Web 支付（香港商戶），詳見[支付寶 Web 支付](/docs/online-shop/integration-by-payment-type/alipay/alipay-web-payments) |
| 800714 | 銀聯雲閃付 PC-Web 支付（香港商戶）                                                                                     |
| 805814 | PayMe Web 支付（香港商戶）                                                                                        |

---

## 請求參數

| 參數名稱    | 參數編碼           | 是否必填         | 類型          | 說明                                        |
| ------- | -------------- | ------------ | ----------- | ----------------------------------------- |
| 訂單金額    | `txamt`        | 是            | Int(11)     | 單位為最小幣值（100 = $1），建議金額大於 200，以避免風控。       |
| 幣別      | `txcurrcd`     | 是            | String(3)   | 請參見[幣別列表](/docs/preparation/paycode#支付币种) |
| 支付方式    | `pay_type`     | 是            | String(6)   | 如：805814 = PayMe Web 支付                   |
| 商戶訂單號   | `out_trade_no` | 是            | String(128) | 商戶自訂，單一商戶下需唯一                             |
| 交易時間    | `txdtm`        | 是            | String(20)  | 格式：`yyyy-MM-dd HH:mm:ss`                  |
| 二維碼過期時間 | `expired_time` | 否<br/>（僅限正掃） | String(3)   | 單位為分鐘，預設 30 分鐘。可設為最小 5 分鐘，最大 120 分鐘。      |
| 商品名稱    | `goods_name`   | 否            | String(64)  | 商品簡稱，限 20 字以內英數與中文 UTF-8 編碼，不能包含特殊符號。     |
| 子商戶號    | `mchid`        | 否            | String(16)  | 由 QFPay 分配，用於識別子商戶身份。                     |
| 設備 ID   | `udid`         | 否            | String(40)  | 唯一設備識別碼，顯示於後台管理頁面。                        |
| 返回網址    | `return_url`   | 否            | String(512) | 支付完成後跳轉頁面網址。                              |

---

## 回應參數

| 參數名稱      | 參數編碼           | 類型          | 說明                                                                                   |
| --------- | -------------- | ----------- | ------------------------------------------------------------------------------------ |
| 支付方式      | `pay_type`     | String(6)   | 例如：805814 代表 PayMe Web 支付                                                            |
| 系統時間      | `sysdtm`       | String(20)  | QFPay 系統交易時間，格式：`YYYY-MM-DD hh:mm:ss`，用作結算截止依據。                                      |
| 請求交易時間    | `txdtm`        | String(20)  | 商戶提交的交易時間。                                                                           |
| 錯誤描述      | `resperr`      | String(128) | 回應中的錯誤說明。                                                                            |
| 金額        | `txamt`        | Int(11)     | 訂單金額。                                                                                |
| 訊息內容      | `respmsg`      | String(128) | 回應說明內容。                                                                              |
| 商戶訂單號     | `out_trade_no` | String(128) | 商戶提交的外部訂單號。                                                                          |
| QFPay 訂單號 | `syssn`        | String(40)  | 系統產生之交易編號。                                                                           |
| 回應碼       | `respcd`       | String(4)   | `0000` 為成功；`1143/1145` 表示需輪詢查詢交易結果；其他為失敗，詳見[交易狀態碼](/docs/api-reference/status-codes)。 |
| 支付 URL    | `pay_url`      | String(512) | 使用該連結可生成對應錢包之支付二維碼。                                                                  |
