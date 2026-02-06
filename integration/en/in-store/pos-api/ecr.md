---
id: ecr
title: ECR Integration Technical Specification
description: Integration protocol and encryption details for connecting POS with QFPay ECR system.
sidebar_label: ECR Integration
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';


# ECR Integration Technical Specification

:::note
**Supported Terminal Models**
- **Landi A8 / A8S**: All payment methods supported
- **PAX A920**: QR code payment only
:::

## 1. POS-KEY

POS-KEY is a secret key used to encrypt transaction data. It is generated via the Haojin App.

The default encryption is **enabled**. Merchants may toggle this on/off or regenerate their POS-KEY from the Merchant Portal (MMS). For changes to take effect, a POS device refresh via the Haojin App is required.

**Steps to refresh POS-KEY:**

* Haojin App → My → Settings → POS-Key → Generate

**Steps to check POS-KEY:**

* Merchant Portal → Settings → Devices Settings → POS Key Mgmt tab

## 2. Encryption

* All transaction data is encrypted using **AES**.
* Key = POS-KEY
* IV = `qfpay202306_hjsh`
* Encrypted result is Base64 encoded

## 3. Request Payload Format

| Attribute           | Mandatory | Type    | Description                                                          |
| ------------------- | --------- | ------- | -------------------------------------------------------------------- |
| `amt`               | Yes       | Double  | Transaction amount (e.g., 10.1 = HKD $10.10)                         |
| `func_type`         | Yes       | String  | Instruction code                                                     |
| `channel`           | Yes       | String  | Wallet/payment method (see table below)                              |
| `out_trade_no`      | No        | String  | Merchant reference ID                                                |
| `camera_id`         | No        | Integer | 0 = back camera (default), 1 = front camera                          |
| `payment_timeout`   | No        | Integer | Payment timeout (in seconds)                                         |
| `wait_card_timeout` | No        | Integer | Wait time for card input (default: 120s). Not recommended to modify. |

### 3.1 Payment

:::note

- **QR Code Mode Selection**: MPM / CPM mode is auto-selected based on last usage.
- **Camera Selection**: `camera_id` allows use of front or back camera in CPM mode.
  - `0`: back camera (default)
  - `1`: front camera
- **Payment Timeout Settings**:
  - `payment_timeout` in card payment defines max waiting time to tap/swipe the card.
  - For QR/wallet payments, it defines max time for transaction confirmation.
  - PayMe supports max `payment_timeout` of **120 seconds** only.
- **Scan Type** (`scan_type`):
  - `QRCODE_PAY`
  - `SCAN_PAY`
- **Return to Home (`moveToBack`)**:
  - `0`: Do not return to home screen (default)
  - `1`: Automatically return to home screen after transaction

:::

```json
{
    "content": {
        "amt": 100, 
        "camera_id":0, 
        "channel": "card_payment",
        "func_type": 1001, 
        "moveToBack": 1,
        "out_trade_no": "456799999999",
        "wait_card_timeout":120,
        "scan_type": "SCAN_PAY"
    },
    "digest":"76b9186077cdc2bc5d78ae921309811d"
}
```

### 3.2 Refund / Void

| Attribute           | Mandatory | Type    | Description                                               |
| ------------------- | --------- | ------- | --------------------------------------------------------- |
| `orderId`           | Yes       | String  | QFPay Transaction ID (syssn)                              |
| `refund_amount`     | No        | String  | Supports partial refund. Default = max refundable amount. |
| `allow_modify_flag` | No        | Integer | 0 = fixed refund amount (default), 1 = allow modification |

:::note
For Visa / Mastercard, UnionPay Card and American Express Card, the amount of same-day refund must be the **full amount**. Partial same-day refunds are not supported.
:::

```json
{
     "content": {
        "allow_modify_flag":1, 
        "func_type": 1002,
        "orderId": "order_id",
        "refund_amount": "0.05",
        "moveToBack": 1
    },
     "digest": "9C8E9FB05C7C24B6CA04EBFA1263EF41"
}

```

### 3.3 Print Receipt

```json
{
    "content": {"orderId": "12345678","func_type": 3001},
    "digest":"79fd145311d54d03e4e685d50f15dd7f"
}
```

### 3.4 Print Transaction Summary

```json
{
    "content": {"func_type": 3002},
    "digest":"79fd145311d54d03e4e685d50f15dd7f"
}
```

### 3.5 Transaction Inquiry by Order ID

Paramter `out_trade_no` is supported

```json
{
     "content": {"orderId": "1234567890","func_type": 4001},
     "digest":"99CE8BF9C7304AC964522D10F51660B4"
}
```

### 3.6 Cancel Trade/Refund Request

```json
{
    "content": {"func_type": 5001},
    "digest": "99CE8BF9C7304AC964522D10F51660B4"
}
```

## 4. Signature Generation

```javascript
// original payload
content={"amt":100,"channel": "card_payment","func_type":1001,"out_trade_no":"456799999999"} 

// sorted keys in alphabetical ascending order
format_content={amt=100,channel='card_payment',func_type=1001,out_trade_no='456799999999'} 

// encryption
// !! if the value is empty, pass '' (empty string) instead
digest=md5(format_content + pos_key)
digest=md5({amt=100,channel='card_payment',func_type=1001,out_trade_no='456799999999'}f46b1f****aacd) 

```

If encryption is enabled, encrypt `content` with AES and calculate `digest` based on encrypted payload.

```json
{
    "content": "{func_type: 3002}",
    "digest":"79fd145311d54d03e4e685d50f15dd7f"
}
```

## 5. Field Definitions

### 5.1 `func_type`

| Value  | Description                 |
| ------ | --------------------------- |
| `1001` | Payment                     |
| `1002` | Refund                      |
| `3001` | Print receipt               |
| `3002` | Print transaction summary   |
| `4001` | Transaction inquiry         |
| `5001` | Cancel trade/refund request |

### 5.2 `channel` (Payment Method)

> **CPM** = Consumer-Presented QR Code<br/>**MPM** = Merchant-Presented QR Code

| Value           | Description           | PayType Mapping                 |
| --------------- | --------------------- | ------------------------------- |
| `card_payment`  | Visa / Mastercard     | `802808`                        |
| `wx`            | WeChat Pay            | `800208` (CPM) / `800201` (MPM) |
| `alipay`        | Alipay                | `800108` (CPM) / `800101` (MPM) |
| `payme`         | PayMe                 | `805808` (CPM) / `805801` (MPM) |
| `union`         | UnionPay QuickPass    | `800708` (CPM) / `800701` (MPM) |
| `fps`           | FPS                   | `802001` (MPM)                  |
| `octopus`       | Octopus               | `803708`                        |
| `unionpay_card` | UnionPay Card         | `806708`                        |
| `amex_card`     | American Express Card | `806808`                        |

### 5.3 `amt`

Transaction amount (in dollars)

### 5.4 `orderId`

QFPay Transaction ID (same as `syssn` or `out_trade_no`)

## 6. Response Format

```json
{\"respcd\": \"6000\",\"data\": \"{"aaaaaa"}\",\"respmsg\": \"xxxxxxxxxx\",\"resperr\":\"xxxxxxxxxx\"}
```

### Response Codes

| Code | Meaning                     |
| ---- | --------------------------- |
| 4003 | Request Denied              |
| 5001 | Decryption Failed           |
| 4004 | Incorrect Method (use POST) |
| 4005 | Other Errors                |
| 4006 | Incorrect Parameters        |
| 6000 | Request Successful          |
| 6001 | Request Cancelled           |
| 6002 | Request Error               |

### Common Fields

| Field   | Description         |
| ------- | ------------------- |
| respcd  | Return code         |
| respmsg | Response message    |
| resperr | Error message       |
| data    | Business data block |

### Trade Response Data

| Field        | Description             |
| ------------ | ----------------------- |
| mchntnm      | Merchant name           |
| sysdtm       | QF system time          |
| userid       | Store ID                |
| busicd       | Business code (PayType) |
| txamt        | Amount                  |
| txcurrcd     | Currency                |
| chnlsn       | Channel order number    |
| paydtm       | Payment time            |
| udid         | Device/user ID          |
| syssn        | QF Order ID             |
| clisn        | Client serial number    |
| out_trade_no | Merchant Order ID       |
| cardscheme   | Card scheme used        |

### Refund Response Data

| Field       | Description           |
| ----------- | --------------------- |
| orig_syssn  | Original Order ID     |
| syssn       | Refund Order ID       |
| txamt       | Refund amount         |
| originTxamt | Original amount       |
| sysdtm      | System time           |
| paydtm      | Refund processed time |

### Transaction Inquiry Data

| Field            | Description             |
| ---------------- | ----------------------- |
| server_time      | Server time             |
| cancel           | Cancel status           |
| clisn            | Client SN               |
| opuid            | Operator ID             |
| syssn            | QF Order ID             |
| tradetp          | Type: payment/refund    |
| sysdtm           | System time             |
| txcurrcd         | Currency                |
| origssn          | Original Order ID       |
| opuser           | Operator name           |
| nickname         | Store name              |
| allow_refund_amt | Refundable amount       |
| desc             | Description             |
| txamt            | Transaction amount      |
| busicd           | Current PayType         |
| origbusicd       | Original PayType        |
| chnlsn           | Wallet order ID         |
| cardscheme       | VISA / MASTERCARD / etc |
| cardno           | Masked card no.         |
| cardtype         | CREDIT / DEBIT          |
| batchno          | Batch number            |
| refno            | Reference number        |

---

## 7. USB Data Transmission Method

1. Connect the POS to the cash register via USB cable.
2. Follow the USB communication protocol to construct the data. See [Section 10: Communication Protocol](#10-cash-register--pos-communication-protocol-usb).
3. Data response: the received payload must be parsed according to the protocol and decrypted via AES.

## 8. HTTP Protocol

1. HTTP transmission requires POS host IP and port. Default port: `9001`.
2. Data Format:
   - Encrypt the payload via AES.
   - Send request via HTTP POST.

3. API Paths:
   | Operation              | Endpoint Path              |
   |------------------------|----------------------------|
   | Trade                  | `/api/pos/trade`           |
   | Refund                 | `/api/pos/cancel`          |
   | Print Receipt          | `/api/pos/print_receipt`   |
   | Print Summary          | `/api/pos/transaction_info`|
   | Transaction Inquiry    | `/api/pos/query_transaction`|
   | Cancel Trade/Refund    | `/api/pos/cancel_request`  |

4. Headers:  
   - `Content-Type: application/json`

5. Response:  
   - Response must be decrypted using AES to obtain result content.

## 9. TCP Protocol

1. TCP transmission requires POS host and port. Default port: `9002`.
2. The cash register connects to POS via **socket**.
3. Data is sent over socket as AES-encrypted payload.
4. Response must also be AES-decrypted to read content.

## 10. Cash Register & POS Communication Protocol (USB)

### 10.1 Use Scenario

Cash register communicates with Smart POS via serial port (USB or Bluetooth) using Haojin Merchant App to perform transactions or cancellations.

### 10.2 Communication Method

- Connection via **Micro USB** or **USB Host mode using a base**.
- USB preferred for stability, security, and easier deployment.

### 10.3 Payload Format

| Field Name                | Content            | Description                                               | Length    |
|--------------------------|--------------------|-----------------------------------------------------------|-----------|
| `Start indicator`        | `0x2f6e`            | Marks start of payload                                    | 2 Bytes   |
| `version`                | `0x01`              | Static version                                            | 1 Byte    |
| `payload type`           | `0x10` / `0x20` / `0x30` | Request / Response / Error                        | 1 Byte    |
| `response ref number`    | `0x01 ~ 0x7f`        | Used for request-response pairing                         | 1 Byte    |
| `payload length`         | —                   | Length from `Start` to `End` indicator                    | 2 Bytes   |
| `payload length (data)`  | —                   | Length of `data segment`                                  | 2 Bytes   |
| `data segment`           | —                   | UTF-8 encoded business data                               | Variable  |
| `End indicator`          | `0x2f6e`            | Marks end of payload                                      | 2 Bytes   |

:::warning
> `0x2f6e` is the hexadecimal of `/n` (string literal, not newline) in ASCII encoding.
:::

### 10.4 Error Types

| Code  | Description                        |
|-------|------------------------------------|
| `0x30` | Unknown error                     |
| `0x31` | Format error                      |
| `0x32` | Validation error                  |
| `0x33` | Decryption error                  |
| `0x34` | Data segment format error         |
| `0x35` | Packet concatenation error        |

### 10.5 Request and Response

- If `payload type` is `0x10`, receiver must send:
  - `0x20` on **success**
  - `0x32` on **validation failure**
- Response packet number must match request packet number.

### 10.6 Timeout

- **Response timeout**: `1000ms`
- If timeout occurs, request is marked failed and device disconnects.

### 10.7 Payload Length

1. Total packet length (from start to end)
2. Data segment length
3. Max payload: **65536 bytes** (2 bytes length)
4. Recommended: ≤ **1024 bytes**. If exceeded, split into packets.

##### Data Splitting / Concatenation

- Same `ref number`, `total length` specified in each packet.
- Receiver waits for full set (within **500ms**); else discards and returns `0x35` error.

### 10.8 AES Encryption

- **Algorithm**: AES-128 (CBC mode, PKCS5 padding)
- **Key**: Provided by service provider (16 bytes)
- Both request & response must be AES encrypted.

### 10.9 Serial Port Settings

| Setting        | Value     |
|----------------|-----------|
| Baud rate      | 9600      |
| Stop bit       | 1         |
| Parity bit     | 0         |
| Data bits      | 8         |
| Flow control   | Off       |

##### Supported USB-to-Serial Chips

| Chip    | Supported |
|---------|-----------|
| PL2303 HXD | ✅ Yes |
| CH340      | ❌ No  |
| FT232      | ❌ No  |

:::note
The table above applies to this POS integration’s USB communication mode only. 

If merchants are **not using USB**, and instead integrating via **HTTP or TCP**, chip support is not relevant. In terms of general driver compatibility and stability across platforms, the typical ranking is: FT232 > CH340 > PL2303.
:::

### 10.10 Sample Payload

**Plaintext Example**:

```json
{"content":"{\"amt\":100,\"channel\":\"wx\",\"funcType\":1,\"mode\":1}","digest":"2f0c4683e25a7b9407265033070e9034"}
```

**Hexadecimal Packet (complete)**:
```plaintext
2f6e011001007f00747b22636f6e74656e74223a227b5c22616d745c223a3130302c5c226368616e6e656c5c223a5c2277785c222c5c2266756e63547970655c223a312c5c226d6f64655c223a317d222c22646967657374223a223266306334363833653235613762393430373236353033333037306539303334227d2f6e
```

