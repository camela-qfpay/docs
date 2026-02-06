---
id: response-format
title: 回應格式
sidebar_label: 回應格式
---

### API 回應格式

所有 QFPay 的 API 均以 JSON 格式返回資料。標準成功的回應結構如下：

```json
{
  "respcd": "0000",
  "respmsg": "success",
  "data": {
    "txamt": "100",
    "out_trade_no": "20231101000001",
    "txcurrcd": "HKD",
    "txstatus": "SUCCESS",
    "qf_trade_no": "9000020231101000001",
    "pay_type": "800101",
    "txdtm": "2023-11-01 10:00:00"
  }
}
```

### 欄位說明

| 欄位| 類型 | 說明                                                                 |
| ------------- | ----------- | --------------------------------------------------------------------------- |
| `respcd`      | String(4)   | 回應代碼。"0000" 代表成功，其他代碼代表失敗。|
| `respmsg`     | String(64)  | 對 respcd 對應的文字說明。|
| `data`        | Object      | 包含交易資料的物件，詳見下方說明。|

#### `data` 物件欄位說明

| 欄位             | 類型        | 說明                                                |
|------------------|-------------|-----------------------------------------------------|
| `txamt`          | String      | 交易金額（以分為單位）                             |
| `out_trade_no`   | String      | 商戶原始訂單編號                                   |
| `txcurrcd`       | String      | 貨幣代碼（例如：HKD）                              |
| `txstatus`       | String      | 支付狀態：`SUCCESS`、`FAILED`、`PENDING`            |
| `qf_trade_no`    | String      | QFPay 指派的唯一交易編號                           |
| `pay_type`       | String      | 支付方式代碼                                       |
| `txdtm`          | String      | 支付時間（格式：`YYYY-MM-DD HH:mm:ss`）            |

### 回應簽名驗證

:::note
在關鍵系統整合中，建議開發者驗證回應 Header 中的簽名（若存在），以確保資料完整性。
:::

回應中可能包含 X-QF-SIGN 和 X-QF-SIGNTYPE Header，可依以下方式進行驗證：

1.	依照欄位名稱升冪排序取出 data 物件中的所有欄位。
2.	將欄位組合為：key1=value1&key2=value2&... 的格式。
3.	尾端附加商戶的 client_key。
4.	使用 MD5 進行雜湊處理並與回應中的簽名進行比對。

完整的簽名產生與驗證邏輯，請參考 [簽名生成方式](./signature-generation.md)。