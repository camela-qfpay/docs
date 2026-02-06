---
id: ecr
title: ECR 整合技術規格
description: 用於連接 POS 與 QFPay ECR 系統的整合通訊協定與加密規格說明。
sidebar_label: ECR 整合
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# ECR 整合技術規格

<Note>
**支援的終端機型號**
- **Landi A8 / A8S**：支援所有支付方式  
- **PAX A920**：僅支援 QR Code 支付
</Note>

---

## 1. POS-KEY

POS-KEY 為用於加密交易資料的金鑰，透過 Haojin App 產生。

系統預設為 **啟用加密**。商戶可於商戶平台（MMS）開啟 / 關閉加密或重新產生 POS-KEY，並需於 Haojin App 進行裝置刷新後才會生效。

**重新產生 POS-KEY：**

- Haojin App → 我的 → 設定 → POS-Key → 產生

**查詢 POS-KEY：**

- 商戶平台 → 設定 → 裝置設定 → POS Key 管理

## 2. 加密方式（Encryption）

- 所有交易資料皆使用 **AES 加密**
- 加密金鑰：POS-KEY
- IV：`qfpay202306_hjsh`
- 加密後結果再進行 Base64 編碼

## 3. 請求資料格式（Request Payload）

| 參數 | 必填 | 型態 | 說明 |
|------|------|------|------|
| `amt` | 是 | Double | 交易金額（如 10.1 = HKD $10.10） |
| `func_type` | 是 | String | 功能指令代碼 |
| `channel` | 是 | String | 錢包 / 支付方式 |
| `out_trade_no` | 否 | String | 商戶交易參考號 |
| `camera_id` | 否 | Integer | 0 = 後鏡頭（預設），1 = 前鏡頭 |
| `payment_timeout` | 否 | Integer | 支付逾時時間（秒） |
| `wait_card_timeout` | 否 | Integer | 等待刷卡時間（預設 120 秒，不建議修改） |


### 3.1 付款（Payment）

:::note
- **QR Code 模式**：MPM / CPM 依最近一次使用自動切換  
- **鏡頭選擇**：`camera_id` 可選前/後鏡頭  
- **付款逾時設定**：
  - 刷卡交易：等待感應卡最大時間
  - 錢包交易：整筆交易最大等候時間
  - **PayMe 最大僅支援 120 秒**
- **掃碼類型 (`scan_type`)**：
  - `QRCODE_PAY`
  - `SCAN_PAY`
- **完成後返回首頁 (`moveToBack`)**
  - `0`：不返回（預設）
  - `1`：交易完成後自動返回首頁
:::

```json
{
  "content": {
    "amt": 100,
    "camera_id": 0,
    "channel": "card_payment",
    "func_type": 1001,
    "moveToBack": 1,
    "out_trade_no": "456799999999",
    "wait_card_timeout": 120,
    "scan_type": "SCAN_PAY"
  },
  "digest": "76b9186077cdc2bc5d78ae921309811d"
}
```

### 3.2 退款（Refund / Void）

| 欄位 | 必填 | 型別 | 說明 |
|------|------|------|------|
| `orderId` | 是 | String | QFPay 交易編號（syssn） |
| `refund_amount` | 否 | String | 退款金額（預設為最大可退金額，支援部分退款） |
| `allow_modify_flag` | 否 | Integer | 0 = 不可修改（預設），1 = 可修改 |

:::note
Visa / Mastercard / 銀聯卡 / 美國運通卡  
**當日退款必須全額退款，不支援當日部分退款。**
:::

```json
{
  "content": {
    "allow_modify_flag": 1,
    "func_type": 1002,
    "moveToBack": 1,
    "orderId": "order_id",
    "refund_amount": "0.05"
  },
  "digest": "9C8E9FB05C7C24B6CA04EBFA1263EF41"
}

```

### 3.3 列印收據（Print Receipt）

```json
{
    "content": {"orderId": "12345678","func_type": 3001},
    "digest":"79fd145311d54d03e4e685d50f15dd7f"
}
```

### 3.4 3.4 列印交易摘要（Print Summary）

```json
{
    "content": {"func_type": 3002},
    "digest":"79fd145311d54d03e4e685d50f15dd7f"
}
```

### 3.5 3.4 列印交易摘要（Print Summary）
支援參數 `out_trade_no`

```json
{
     "content": {"orderId": "12345678","func_type": 4001},
     "digest":"99CE8BF9C7304AC964522D10F51660B4"
}
```

### 3.6 取消付款 / 退款請求
{/* md:version 4.31.3 */}

```json
{
     "content": {"func_type": 5001},
     "digest":"99CE8BF9C7304AC964522D10F51660B4"
}
```

## 4. 簽名產生機制（Signature Generation）

**簽名計算流程範例**

```js
// original payload
content={"amt":100,"channel": "card_payment","func_type":1001,"out_trade_no":"456799999999"} 

// sorted keys in alphabetical ascending order
format_content={amt=100,channel='card_payment',func_type=1001,out_trade_no='456799999999'} 这种格式

// encryption
// !! if the value is empty, pass '' (empty string) instead
digest=md5(format_content + pos_key)
digest=(
  md5(
    {amt=100,channel='card_payment',func_type=1001,out_trade_no='456799999999'}f46b1f****aacd
  ) 
)

```

如啟用加密，需**先將 content 進行 AES 加密，再以加密結果計算 digest**。

**加密後請求範例**

```json
{
    "content": "{func_type: 3002}",
    "digest":"79fd145311d54d03e4e685d50f15dd7f"
}
```

## 5. 欄位定義（Field Definitions）

### 5.1 `func_type`

| 數值 | 說明 |
|------|------|
| 1001 | 交易 |
| 1002 | 退款 |
| 3001 | 列印收據 |
| 3002 | 列印交易摘要 |
| 4001 | 交易查詢 |
| 5001 | 取消請求 |

---

### 5.2 `channel`（付款方式）

> CPM = 消費者出示碼  
> MPM = 商戶出示碼  

| 值 | 說明 | PayType |
|----|------|---------|
| card_payment | Visa / Mastercard | 802808 |
| wx | 微信支付 | 800208 / 800201 |
| alipay | 支付寶 | 800108 / 800101 |
| payme | PayMe | 805808 / 805801 |
| union | 銀聯閃付 | 800708 / 800701 |
| fps | FPS | 802001 |
| octopus | 八達通 | 803708 |
| unionpay_card | 銀聯卡 | 806708 |
| amex_card | 美國運通 | 806808 |


## 6. 回應格式（Response Format）


```json
{\"respcd\": \"6000\",\"data\": \"{"aaaaaa"}\",\"respmsg\": \"xxxxxxxxxx\",\"resperr\":\"xxxxxxxxxx\"}
```

### 回應代碼（Response Codes）

| 代碼 | 說明 |
|------|------|
| 4003 | 請求被拒絕 |
| 5001 | 解密失敗 |
| 4004 | 請求方法錯誤（請使用 POST） |
| 4005 | 其他錯誤 |
| 4006 | 參數錯誤 |
| 6000 | 請求成功 |
| 6001 | 使用者取消 |
| 6002 | 請求失敗 |

---

### 通用回應欄位（Common Fields）

| 欄位 | 說明 |
|------|------|
| `respcd` | 回應代碼 |
| `respmsg` | 回應訊息 |
| `resperr` | 錯誤訊息 |
| `data` | 業務資料區塊 |

---

### 交易回應資料（Trade Response Data）

| 欄位 | 說明 |
|------|------|
| `mchntnm` | 商戶名稱 |
| `sysdtm` | QF 系統時間 |
| `userid` | 門店 ID |
| `busicd` | 交易業務代碼（PayType） |
| `txamt` | 交易金額 |
| `txcurrcd` | 交易幣別 |
| `chnlsn` | 通道訂單號 |
| `paydtm` | 錢包支付時間 |
| `udid` | 裝置 / 使用者 ID |
| `syssn` | QF 訂單號 |
| `clisn` | 客戶端序號 |
| `out_trade_no` | 商戶訂單號 |
| `cardscheme` | 卡組織（VISA / MASTERCARD / UNIONPAY / AMEX） |

---

### 退款回應資料（Refund Response Data）

| 欄位 | 說明 |
|------|------|
| `orig_syssn` | 原始交易單號 |
| `syssn` | 退款交易單號 |
| `txamt` | 退款金額 |
| `originTxamt` | 原始交易金額 |
| `sysdtm` | 系統時間 |
| `paydtm` | 退款完成時間 |

---

### 交易查詢回應資料（Transaction Inquiry Data）

| 欄位 | 說明 |
|------|------|
| `server_time` | 伺服器時間 |
| `cancel` | 取消狀態 |
| `clisn` | 客戶端序號 |
| `opuid` | 操作員 ID |
| `syssn` | QF 訂單號 |
| `tradetp` | 交易類型（payment / refund） |
| `sysdtm` | 系統時間 |
| `txcurrcd` | 交易幣別 |
| `origssn` | 原始交易單號 |
| `opuser` | 操作員名稱 |
| `nickname` | 門店名稱 |
| `allow_refund_amt` | 可退款金額 |
| `desc` | 交易描述 |
| `txamt` | 交易金額 |
| `busicd` | 目前交易 PayType |
| `origbusicd` | 原始交易 PayType |
| `chnlsn` | 錢包訂單號 |
| `cardscheme` | 卡組織（VISA / MASTERCARD / UNIONPAY / AMEX） |
| `cardno` | 遮罩卡號（例：520000******1096） |
| `cardtype` | 卡種類（CREDIT / DEBIT） |
| `batchno` | 批次號 |
| `refno` | 參考號 |

## 7. USB 資料傳輸方式

1. 以 USB 連接 POS 與收銀機
2. 依 USB 通訊協議封裝資料
3. 回傳資料需依協議解析後再進行 AES 解密

## 8. HTTP 通訊協議（HTTP Protocol）

1. **HTTP 傳輸需指定 POS 主機 IP 與 Port**，預設 Port 為 `9001`。
2. **資料格式**：
   - 使用 AES 演算法對請求資料進行加密。
   - 加密後的 Payload 以 `HTTP POST` 方法傳送至指定端點。

3. **API 路徑對應表**：

| 操作項目             | API 路徑                       |
|----------------------|-------------------------------|
| 發起交易             | `/api/pos/trade`              |
| 發起退款             | `/api/pos/cancel`             |
| 列印收據             | `/api/pos/print_receipt`      |
| 列印交易摘要         | `/api/pos/transaction_info`   |
| 查詢交易結果         | `/api/pos/query_transaction`  |
| 取消交易／退款請求   | `/api/pos/cancel_request`     |

4. **HTTP 請求標頭（Header）**：

| 欄位           | 值                     |
|----------------|------------------------|
| `Content-Type` | `application/json`     |

5. **HTTP 回應處理**：

回傳結果亦為 AES 加密內容，接收端需進行解密才能取得交易結果資料。

## 9. TCP 協議

- 預設通訊埠：9002  
- 以 Socket 方式連線  
- 內容皆為 AES 加密資料  

## 10. 收銀機與 POS 通訊協議（USB）

### 10.1 使用場景
透過 Haojin App 進行付款與取消交易。

### 10.2 傳輸方式
- 使用 Micro USB 或底座轉換 USB Host
- USB 較 WiFi 穩定、安全

### 10.3 封包結構

| 欄位 | 內容 | 說明 | 長度 |
|------|------|------|------|
| Start indicator | 0x2f6e | 封包起始 | 2 Bytes |
| version | 0x01 | 版本 | 1 Byte |
| payload type | 0x10 / 0x20 / 0x30 | 請求 / 回應 / 錯誤 | 1 Byte |
| ref number | 0x01~0x7f | 封包對應碼 | 1 Byte |
| payload length | — | 總長度 | 2 Bytes |
| data length | — | 資料長度 | 2 Bytes |
| data segment | — | 資料主體 | 不定 |
| End indicator | 0x2f6e | 封包結尾 | 2 Bytes |

<Warning>
`0x2f6e` 為 ASCII 編碼中字串 `/n` 的十六進制表示，**不是換行字元**。
</Warning>

### 10.4 錯誤碼

| 代碼 | 說明 |
|------|------|
| 0x30 | 不明錯誤 |
| 0x31 | 格式錯誤 |
| 0x32 | 驗證錯誤 |
| 0x33 | 解密錯誤 |
| 0x34 | 資料格式錯誤 |
| 0x35 | 封包拼接錯誤 |

### 10.5 Timeout
回應逾時為 **1000ms**

### 10.6 AES 加密規格
- AES-128 / CBC / PKCS5Padding

### 10.7 串列埠設定

| 設定 | 值 |
|------|----|
| Baud rate | 9600 |
| Stop bit | 1 |
| Parity | 0 |
| Data bits | 8 |
| Flow control | Off |

##### USB-to-Serial 晶片支援

| 晶片 | 是否支援 |
|------|----------|
| PL2303 HXD | ✅ |
| CH340 | ❌ |
| FT232 | ❌ |

:::note
此支援表僅適用於 **本 USB 模式整合**。  
若商戶使用 **HTTP / TCP 整合方式**，則晶片限制不適用。  
一般穩定性與相容性排序為：**FT232 > CH340 > PL2303**。
:::

### 10.8 封包範例

**明文封包範例：**

`{\"content\":\"{\\\"amt\\\":100,\\\"channel\\\":\\\"wx\\\",\\\"funcType\\\":1,\\\"mode\\\":1}\",\"digest\":\"2f0c4683e25a7b9407265033070e9034\"}`

**Hex 封包範例：**
`2f6e011001007f00747b22636f6e74656e74223a227b5c22616d745c223a3130302c5c226368616e6e656c5c223a5c2277785c222c5c2266756e63547970655c223a312c5c226d6f64655c223a317d222c22646967657374223a223266306334363833653235613762393430373236353033333037306539303334227d2f6e`
