---

id: in-app-payment
title: In-App 支付（原生 App 內支付）
description: 本頁介紹如何整合 App 原生環境下的電子錢包支付功能，支援多種主流錢包類型與標準支付流程。
sidebar_label: In-App 支付
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# In-App 支付（原生 App 內支付）

在原生 App（iOS / Android）中整合電子錢包的支付功能，可讓用戶在應用程式內無縫完成付款流程，不需跳出至外部瀏覽器。

---

## 支援場景

In-App 支付適用於以下情境：

* 原生應用程式內建購物或付款功能
* 搭配電子錢包 SDK 呼叫支付模組（如微信、支付寶）
* 用戶在 App 內點選「付款」後，直接跳轉至電子錢包支付頁完成交易

---

## 常見錢包與交易類型

您可以從下表中查閱各種主流錢包的 `pay_type` 值與對應技術文檔：

| 交易類型（pay_type） | 支付錢包      | 說明                                                                                                         |
| -------------- | --------- | ---------------------------------------------------------------------------------------------------------- |
| `800210`       | 微信        | [微信 In-App 支付](/docs/online-shop/integration-by-payment-type/wechat/wechat-in-app-payments)（使用微信 SDK 呼叫付款） |
| `801510`       | 支付寶香港     | [支付寶香港 In-App 支付](/docs/online-shop/integration-by-payment-type/alipay/alipay-in-app)（透過 URL Scheme 或 SDK） |
| `805810`       | PayMe（香港） | PayMe In-App 支付     |
| `800710`       | 銀聯雲閃付     | 雲閃付 In-App 支付     |

---

## 常見整合方式

* **SDK 模式**：由商戶 App 透過第三方 SDK 呼叫錢包 API
* **URL Scheme / Deep Link 模式**：商戶生成包含付款資訊的跳轉連結，引導至錢包 App 完成支付
* **跳轉後返回原 App**：成功支付後導回商戶 App（需設置 redirect URL 或應用綁定）

---

## 通用 API 呼叫流程

大部分 In-App 支付方式會使用 `POST /trade/v1/payment` 端點提交支付請求。回應中的 `pay_params` 或 `pay_url` 將提供給 SDK 或 App 呼叫。

具體參數格式請參見各支付方式的個別文檔。

---

## 注意事項

* 各錢包要求不同，請確認是否需事前申請開通 App 支付權限（如微信 SDK、支付寶商戶 App ID 等）
* 請務必測試跳轉與返回邏輯，確保付款完成後能正確返回 App 並顯示付款結果
* 建議搭配交易查詢 API 確認最終支付狀態，以避免用戶關閉 App 導致回調失敗

---

## 延伸閱讀

* [交易查詢 API](/docs/common-api/transaction-enquiry)
* [支付狀態碼對照表](/docs/api-reference/status-codes)
* [開發者常見問題 FAQ](/docs/faq)
