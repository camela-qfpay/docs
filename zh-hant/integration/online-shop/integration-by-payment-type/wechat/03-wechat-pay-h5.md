---
id: wechat-pay-h5
title: 微信 H5 支付（第三方瀏覽器）
description: 商戶可透過本指引整合微信 H5 支付（非微信瀏覽器），支援設定場景資訊與 redirect_url。
sidebar_label: 微信 H5 支付
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# 微信 H5 支付（第三方瀏覽器）

<Link href="/img/wechat-h5.png" target="_blank">![WeChat H5 process-flow](@site/static/img/wechat-h5.png)</Link>

微信 H5 支付適用於從 **微信以外的瀏覽器**（例如 Safari、Chrome 等）中啟動的支付流程，例如商戶 App 內嵌瀏覽器或移動網頁頁面。

---

## HTTP 請求

`POST /trade/v1/payment`
**支付類型:** `800212`

---

## 請求參數

| 參數名稱   | 參數編碼          | 是否必填 | 類型     | 描述                                             |
| ------ | ------------- | ---- | ------ | ---------------------------------------------- |
| 公共支付參數 | —             | 是    | —      | 請參閱[公共支付參數](/docs/api-reference/request-format) |
| 擴展資訊   | `extend_info` | 是    | Object | 含場景資訊與用戶 IP 地址                                 |

```js
extend_info: {
  "scene_info": {
    "h5_info": {
      "type": "Wap",
      "wap_url": "https://qfpay.global/h5/pay",
      "wap_name": "qfpay"
    }
  },
  "spbill_create_ip": "192.168.1.10"
}
```

> IP 地址獲取請參考[微信官方指引](https://pay.weixin.qq.com/wiki/doc/api/H5.php?chapter=15_5)

### `extend_info` 詳細結構

| 主參數                | 次參數       | 子參數        | 是否必填 | 類型     | 描述                 |
| ------------------ | --------- | ---------- | ---- | ------ | ------------------ |
| `scene_info`       | —         | —          | 是    | Object | 場景資訊容器             |
| —                  | `h5_info` | —          | 是    | Object | 固定傳入               |
| —                  | —         | `type`     | 是    | String | 場景類型，固定為 **"Wap"** |
| —                  | —         | `wap_url`  | 是    | String | 商戶網站網址，需為手機版頁面 URL |
| —                  | —         | `wap_name` | 是    | String | 商戶網站名稱，會顯示給用戶      |
| `spbill_create_ip` | —         | —          | 是    | String | 用戶真實 IP 地址         |

---

## 回應參數

| 參數名稱   | 子參數       | 類型     | 是否必填 | 描述                                             |
| ------ | --------- | ------ | ---- | ---------------------------------------------- |
| 公共回應參數 | —         | —      | 是    | 請參閱[公共回應參數](/docs/api-reference/response-format) |
| 支付網址   | `pay_url` | String | 是    | 用戶需跳轉至該網址進行支付                                  |

---

## 使用 redirect_url 自訂支付完成後導向頁面

若希望使用者支付完成後導回特定網址，可在返回的 `pay_url` 中追加參數 `redirect_url`。

### 原始 pay_url 範例：

```
https://wx.tenpay.com/cgi-bin/mmpayweb-bin/checkmweb?prepay_id=wx20161110163838f231619da20804912345&package=1037687096
```

### 插入 redirect_url 後：

```
https://wx.tenpay.com/cgi-bin/mmpayweb-bin/checkmweb?prepay_id=wx20161110163838f231619da20804912345&package=1037687096&redirect_url=https%3A%2F%2Fwww.wechatpay.com.cn
```

請確保：

* `redirect_url` 已完成 URL 編碼
* 測試時實際打開網址驗證跳轉流程無誤

---

## 補充說明

* 適用於 App 內嵌瀏覽器、手機網站等場景，**不適用於微信內部瀏覽器**。
* 商戶須正確傳入 `wap_url` 與 `wap_name`，否則微信可能拒絕支付請求。
* 請確保 IP 地址為用戶設備的真實來源 IP，以通過微信風控。