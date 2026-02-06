---
id: transaction-note
title: 交易備註 API 指南
description: 商戶可透過本接口為交易添加備註，便於後續查詢與報表顯示。
sidebar_label: 交易備註 API
---

# 交易備註 API 指南

:::note
此接口僅適用於未傳入 `mchid` 的商戶。
:::

商戶可使用本接口為交易添加備註文字，該備註內容將顯示於商戶後台及交易報表中，便於識別與對帳。


## API 請求範例

```http
POST /trade/v1/add_note
Content-Type: application/x-www-form-urlencoded
X-QF-APPCODE: D5589D2A1F2E42A9A60C37**********
X-QF-SIGN: 6FB43AC29175B4602FF95F8332028F19

code=A6A49A6******DFE94EA95032&note=add_note&syssn=20190722000200020081075691
```
---

> 上述請求將回傳如下 JSON 結構：

```json
{
  "resperr": "Success",
  "respcd": "0000",
  "respmsg": "",
  "data": {
    "syssn": "20190722000200020081084545"
  }
}
```

## HTTP 請求說明

**接口路徑**：`/trade/v1/add_note`  
**請求方法**：`POST`

## 請求參數

| 參數名稱 | 必填 | 類型        | 說明                                                                 |
|----------|------|-------------|----------------------------------------------------------------------|
| `code`   | 是   | String(32)  | 商戶應用代碼，由 QFPay 分配                                         |
| `syssn`  | 是   | String(40)  | QFPay 交易號（交易完成後由系統返回）                               |
| `note`   | 是   | String(200) | 備註內容，將顯示於商戶後台與交易報表中                             |


## 回應參數

| 參數名稱   | 類型        | 說明                                            |
|------------|-------------|-------------------------------------------------|
| `resperr`  | String(128) | 接口執行結果訊息                               |
| `respmsg`  | String(128) | 錯誤訊息（如有）                                |
| `respcd`   | String(4)   | 返回碼，`0000` 代表接口調用成功                |
| `syssn`    | String(40)  | 系統返回的交易號（與請求中相同）               |