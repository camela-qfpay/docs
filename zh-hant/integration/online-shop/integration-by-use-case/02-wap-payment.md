---
id: wap-payment
sidebar_label: WAP 支付
title: WAP支付
description: WAP 或 H5 支付允許商戶將用戶引導致手機瀏覽器使用電子錢包進行付款
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# WAP 支付

## 簡介

WAP 或 H5 支付允許商戶將用戶引導至 Chrome / Safari 等手機瀏覽器開啟支付流程。

:::note
WAP／H5 支付建議商戶平台引導用戶使用手機瀏覽器（如 Chrome／Safari／Edge）開啟支付流程。

由於 WhatsApp、Facebook Messenger、WeChat 等社交 App 的瀏覽器內嵌限制，QFPay 無法保證能自動調用其他電子錢包 App。

例如：在 WeChat App 中無法自動開啟 Alipay App，這類限制屬於非瀏覽器環境的技術限制。
:::

## HTTP 請求

`POST ../trade/v1/payment` <br/>

| 支付類型   | 描述                                                                                                                     |
| ------ | ---------------------------------------------------------------------------------------------------------------------- |
| 800212 | 微信 H5 支付，請參閱 [WeChat Pay H5](/docs/online-shop/integration-by-payment-type/wechat/wechat-pay-h5)                       |
| 801512 | 支付宝香港 WAP 支付，請參閱 [Alipay WAP H5 Payments](/docs/online-shop/integration-by-payment-type/alipay/alipay-wap-h5-payments) |
| 800712 | 銀聯 WAP 支付                                                                                                              |
| 805812 | PayMe WAP 支付                                                                                                           |

## 請求參數

| 參數名稱      | 參數編碼           | 是否必填 | 類型          | 說明                                         |
| --------- | -------------- | ---- | ----------- | ------------------------------------------ |
| 交易金額      | `txamt`        | 是    | Int(11)     | 以最小單位表示（如 100 = $1）。建議超過 200 以避免被風控。       |
| 交易貨幣      | `txcurrcd`     | 是    | String(3)   | 請參閱 [貨幣列表](/docs/api-reference/currencies) |
| 付款類型      | `pay_type`     | 是    | String(6)   | 例如：805812 = PayMe WAP 支付                   |
| 外部訂單號     | `out_trade_no` | 是    | String(128) | 商戶系統內唯一訂單號                                 |
| 交易時間      | `txdtm`        | 是    | String(20)  | 格式: YYYY-MM-DD hh:mm:ss                    |
| 商品名稱      | `goods_name`   | 否    | String(64)  | 最多 20 個字符，不可含特殊符號                          |
| QFPay 商戶號 | `mchid`        | 否    | String(16)  | QFPay 分配                                   |
| 設備 ID     | `udid`         | 否    | String(40)  | 唯一設備識別碼                                    |
| 重定向 URL   | `return_url`   | 否    | String(255) | 付款完成後重定向                                   |
| 靜態通知 URL  | `notify_url`   | 否    | String(255) | 付款成功後訊息接收地址                                |

### 回應參數

| 參數名稱      | 參數編碼           | 類型          | 說明                                                                                      |
| --------- | -------------- | ----------- | --------------------------------------------------------------------------------------- |
| 付款類型      | `pay_type`     | String(6)   | e.g. PayMe WAP 支付                                                                       |
| 系統交易時間    | `sysdtm`       | String(20)  | 格式: YYYY-MM-DD hh:mm:ss <br/>用於清算截止日期                                                   |
| 請求交易時間    | `txdtm`        | String(20)  | 格式: YYYY-MM-DD hh:mm:ss                                                                 |
| 返回說明      | `resperr`      | String(128) |                                                                                         |
| 交易金額      | `txamt`        | Int(11)     |                                                                                         |
| 其他資訊      | `respmsg`      | String(128) |                                                                                         |
| 外部訂單號     | `out_trade_no` | String(128) | 商戶訂單號                                                                                   |
| QFPay 訂單號 | `syssn`        | String(40)  |                                                                                         |
| 回應碼       | `respcd`       | String(4)   | 0000 = 成功<br/>1143/1145 = 需繼續查詢結果<br/>其他錯誤請參閱 [回應碼列表](/docs/api-reference/status-codes) |
| 支付 URL    | `pay_url`      | String(512) | WAP 場景中用於重定向的支付 URL                                                                     |