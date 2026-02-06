---
id: async-notifications
title: 非同步通知
description: 說明如何處理 QFPay 的支付與退款非同步通知。
sidebar_label: 非同步通知
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 非同步通知（Asynchronous Notifications）

目前支援以下類型的通知：

* 付款成功：**"notify_type": "payment"**
* 退款成功：**"notify_type": "refund"**

未來版本的非同步通知請求參數可能會包含更多欄位，開發者應確保系統能夠容納新欄位。建議前往本網站獲取最新開發文件。

## 描述

當支付或退款成功後，QFPay 將自動向商戶預設的 URL 發送非同步通知，商戶可透過實作接收端點以更新交易狀態。我們建議您搭配 API 的交易查詢功能使用，以確認交易結果。出於安全考量，非同步通知僅支援 80 或 443 端口。

## 非同步通知規則

1. 僅在交易成功（支付或退款）後才會發送通知。
2. 商戶需透過電郵提交通知端點的 URL 至 `technical.support@qfpay.com`，由技術支援人員設定。
3. 收到通知後，商戶需依以下簽名驗證流程確認資料完整性。若驗證成功，請回應 HTTP 狀態碼 200 並回傳字串 `SUCCESS`。
4. 若未收到成功響應，我們將依下列時間間隔重試：2 分、10 分、10 分、60 分、2 小時、6 小時、15 小時。
5. 一組 app code 和 key 只能綁定一個通知 URL。代理商應為所有子商戶使用相同通知 URL。
6. HTTP 方法：POST，content-type 為 `application/json`

## 簽名驗證方法

通知的簽名驗證與一般 POST 請求稍有不同，需將**原始請求內容**加上 `client_key` 後進行 MD5 雜湊比對。

**步驟如下：**

1. 從 HTTP header 的 `X-QF-SIGN` 欄位中取得簽名值。
2. 將接收到的原始 JSON 字串與 `client_key` 串接。
3. 對步驟 2 的字串進行 MD5 雜湊。
4. 將雜湊結果與 `X-QF-SIGN` 比對，若一致則回應 HTTP 200 並在回應體中輸出 `SUCCESS` 字串。

## 簽名範例

<Tabs>
<TabItem value="python" label="Python">

```python
import hashlib
import json

# Client Credentials
client_key = "3ABB1BFFE2E0497BB9270978B0BXXXXX"

# Raw Content Data
data = {"status": "1", "pay_type": "800101", "sysdtm": "2020-06-15 10:32:58", "paydtm": "2020-06-15 10:33:35", "goods_name": "", "txcurrcd": "HKD", "txdtm": "2020-06-15 10:32:58", "mchid": "O37MRh6Qq5", "txamt": "10", "exchange_rate": "", "chnlsn2": "", "out_trade_no": "9G3ZIWTG1R3IVSC2AH2O5EGKJQ7I72QO", "syssn": "20200615000200020000641807", "cash_fee_type": "", "cancel": "0", "respcd": "0000", "goods_info": "", "cash_fee": "0", "notify_type": "payment", "chnlsn": "2020061522001453561406303428", "cardcd": "2088032341453564"}

combine_str = (json.dumps(data)+client_key).encode()

signature = hashlib.md5(combine_str).hexdigest().upper()

print(signature)
```

</TabItem>
</Tabs>

> 範例簽名結果：

```json
"A4021A3B1EBBB0F05451EF94E9EXXXXX"
```

## 回傳欄位說明

> 非同步通知會以 JSON 格式發送內容如下：

```json
{
  "status": "1",
  "pay_type": "800101",
  "sysdtm": "2020-05-14 12:32:56",
  "paydtm": "2020-05-14 12:33:56",
  "goods_name": "",
  "txcurrcd": "HKD",
  "txdtm": "2020-05-14 12:32:56",
  "mchid": "lkbqahlRYj",
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
  "chnlsn": "2020051422001453561444935817",
  "cardcd": "2088032341453564"
}
```


| 欄位名稱                   | 必填 | 類型     | 說明                                                                                                 |
| ---------------------- | -- | ------ | -------------------------------------------------------------------------------------------------- |
| `status`               | 是  | String | 交易成功狀態，固定為 `1`                                                                                     |
| `pay_type`             | 是  | String | QFPay 支付代碼，請參考 [支付方式](/docs/api-reference/paytypes)                                                |
| `sysdtm`               | 是  | String | 系統建立交易時間，用作結算日切點                                                                                   |
| `paydtm`               | 是  | String | 用戶實際支付時間                                                                                           |
| `txcurrcd`             | 是  | String | 交易幣別，請參考 [幣別清單](/docs/api-reference/currencies)                                                    |
| `txdtm`                | 是  | String | 商戶請求中的交易建立時間                                                                                       |
| `txamt`                | 是  | String | 交易金額（單位：分），建議金額大於 200                                                                              |
| `out_trade_no`         | 是  | String | 商戶平台的訂單號                                                                                           |
| `syssn`                | 是  | String | QFPay 交易流水號                                                                                        |
| `cancel`               | 是  | String | 交易取消狀態：<br/>0 = 未取消<br/>1 = 已撤銷或退款成功（CPM）<br/>2 = 成功取消（MPM）<br/>3 = 已退款<br/>4 = 預授權完成<br/>5 = 部分退款 |
| `respcd`               | 是  | String | 固定為 `0000` 表示成功                                                                                    |
| `notify_type`          | 是  | String | 通知類型：`payment` 或 `refund`                                                                          |
| `mchid`                | 否  | String | 商戶號，僅代理商接收通知時會回傳                                                                                   |
| `goods_name`           | 否  | String | 商品名稱，若使用中文請使用 UTF-8 編碼                                                                             |
| `exchange_rate`        | 否  | String | 若有匯率轉換，此欄位表示適用匯率                                                                                   |
| `chnlsn2`              | 否  | String | 附加交易編號                                                                                             |
| `goods_info`           | 否  | String | 商品描述                                                                                               |
| `chnlsn`               | 否  | String | 支付渠道提供的交易編號                                                                                        |
| `cardcd`               | 否  | String | 卡號                                                                                                 |
| `cash_fee`             | 否  | String | 實際用戶支付金額（扣除優惠後）                                                                                    |
| `cash_fee_type`        | 否  | String | 實際支付幣種，例如：CNY                                                                                      |
| `cash_refund_fee`      | 否  | String | 實際退款金額                                                                                             |
| `cash_refund_fee_type` | 否  | String | 實際退款幣種，例如：CNY                                                                                      |

## 發送 IP 清單

非同步通知將會從以下 IP 發送，請確保您的伺服器允許來自這些 IP 的請求：

* 13.228.112.115
* 18.138.115.47
* 18.166.202.92