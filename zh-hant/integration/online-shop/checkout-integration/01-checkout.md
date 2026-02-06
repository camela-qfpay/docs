---
id: checkout
title: 託管結帳頁面（收銀台）
sidebar_label: 託管結帳頁面（收銀台）
description: 透過 QFPay 託管的結帳頁面整合多種支付方式，快速啟用線上付款能力，無需自行開發前端 UI。
---

import Link from '@docusaurus/Link';

# 託管結帳頁面（收銀台）

## 簡介

本頁介紹 **QFPay 託管結帳頁面**，讓開發者可輕鬆整合多種支付方式。以下提供各種語言的程式碼範例。

## 頁面設計

<Link href="/img/shouyintai.png" target="_blank"> ![shouyintai](@site/static/img/shouyintai.png) </Link>

這個結帳頁面具有良好的響應式設計，能夠自動適應不同裝置與螢幕尺寸。介面目前支援英文、簡體中文及繁體中文。可依照商戶需求進行頁面客製化，例如電子錢包選項、樣式設計及付款說明文字等。如有具體需求，請聯絡 `technical.support@qfpay.com` 尋求協助。
## API 環境

:::warning
測試交易完成後，請立即透過商戶後台、QFPay App 或 API 進行退款。
:::

API 環境相關請參考[環境設定](/docs/preparation/environment)。

## 結帳流程

<Link href="/img/flowchart.png" target="_blank"> ![flowchart](@site/static/img/flowchart.png) </Link>

使用者於商戶網站選擇付款，點擊「付款」後將跳轉至 QFPay 託管結帳頁面，選擇錢包並完成付款流程。完成後將跳轉回商戶網站顯示付款結果。

## API 請求參數

**端點** : `https://<API 基礎端點>/checkstand/#/?`

**請求方法** : `GET`

以下為結帳頁面請求所需參數：

| 欄位 | 類型 | 必填 | 描述 |
| ---- | ---- | ---- | ---- |
| `appcode` | String(64) | 是 | QFPay 分配的憑證 |
| `sign_type` | String(256) | 是 | 簽名類型，建議使用 `sha256` |
| `sign` | String(128) | 是 | 驗證用簽名值 |
| `paysource` | String(12) | 是 | 請使用以 `_checkout` 結尾的來源標識，例如 `remotepay_checkout` |
| `txamt` | Int(11) | 是 | 交易金額（單位為分），建議超過 200 |
| `txcurrcd` | String(3) | 是 | 貨幣代碼，例如 `HKD` |
| `out_trade_no` | String(128) | 是 | 商戶自訂交易單號 |
| `txdtm` | String(32) | 是 | 交易時間，格式為 `YYYY-MM-DD hh:mm:ss` |
| `return_url` | String(256) | 是 | 成功付款後跳轉連結 |
| `failed_url` | String(256) | 是 | 付款失敗後跳轉連結 |
| `notify_url` | String(256) | 是 | 非同步通知連結 |
| `mchntid` | String(16) | 否 | 商戶代碼，代理商需填 |
| `goods_name` | String(64) | 否 | 商品名稱，不可含特殊字元，建議不超過 20 字 |
| `udid` | String(40) | 否 | 裝置代碼 |
| `expired_time` | String(3) | 否 | 二維碼有效時間（單位為分鐘，範圍 5–120） |
| `checkout_expired_time` | String(13) | 否 | 客戶端結帳頁逾時時間。支援格式：（最多3位，如 120）或 Unix timestamp 毫秒（13位，如 1715686118000) |
| `limit_pay` | String(3) | 否 | 禁用信用卡，僅 WeChat Pay 支援 |
| `lang` | String(5) | 否 | 語言代碼：zh-hk、zh-cn、en |
| `cancel_url` | String(256) | 否 | 結帳頁「返回商店」按鈕的跳轉連結 |

## 建立結帳訂單

:::info
每筆訂單請使用唯一的 `out_trade_no`。
:::

請以 `GET` 方法發送帶簽名的請求，將上述參數按順序拼接後加上 API Key 計算簽名。程式碼範例與簽名算法請參見下方。

如需完整範例可下載 [QFPay Online Checkout Boilerplate](@site/static/files/qfpay_online_checkout.html)。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>checkout</title>
    <style>

a{
  font-size: 20px;
}
    </style>
</head>
<body>
 <a id="standard">QFPay Online Checkout</a>

</body>
<script src="https://cdn.bootcss.com/blueimp-md5/2.10.0/js/md5.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/js-sha256/0.9.0/sha256.min.js"></script>

<script> 
window.onload = function(){
  let standard = document.getElementById('standard')

  let origin = 'https://test-openapi-hk.qfapi.com/checkstand/#/?'
   let obj = {
    appcode: "CC6FB660837E49F7A675D2**********",
    goods_name: "remotfpay_checkout_names",
    out_trade_no: "13322916216626239614",
    paysource: "remotepay_checkout",
    return_url: "https://www.baidu.com",
    failed_url: "https://www.baidu.com",
    notify_url: "https://www.baidu.com",
    sign_type: "sha256",
    txamt: "1",
    txcurrcd: "HKD",
    txdtm: "2020-06-28 18:33:20"
   }

   let api_key = "B3D4CCFD4AB049DCA82C25**********";
   let params = paramStringify(obj) 
   let sign = sha256(`${params}${api_key}`)
    standard.setAttribute('href', `${origin}${paramStringify(obj,true)}&sign=${sign}`)

}   

function paramStringify(json,flag) {
      let str = "";
      let keysArr = Object.keys(json);
      keysArr.sort().forEach(val => {
        if (!json[val]) return;
        str += `${val}=${flag ? encodeURIComponent(json[val]) : json[val]}&`;
      });
      return str.slice(0, -1);
    }

</script>
</html>
```
