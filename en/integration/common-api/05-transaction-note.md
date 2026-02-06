---
id: transaction-note
title: Transaction Notes API
description: Add custom remarks to a completed transaction. Remarks will show in the Merchant Portal.
sidebar_label: Transaction Notes
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Transaction Notes

<Note>
This endpoint is only available to merchants who **do not** pass in the `mchid`.
</Note>

Merchants can use this API to attach remarks to a completed transaction. These remarks will be visible in the Merchant Portal and transaction reports.

---

## API Endpoint

**URL**: `/trade/v1/add_note`  
**Method**: `POST`  
**Content-Type**: `application/x-www-form-urlencoded`

---

## Request Parameters

| Attribute | Mandatory | Type        | Description |
|-----------|-----------|-------------|-------------|
| `code`    | Yes       | String(32)  | Merchant app code provided by QFPay |
| `syssn`   | Yes       | String(40)  | QFPay transaction number (from original transaction) |
| `note`    | Yes       | String(200) | Remark content to be added (max 200 characters) |

---

## Request Sample

```http
POST /trade/v1/add_note
Content-Type: application/x-www-form-urlencoded
X-QF-APPCODE: D5589D2A1F2E42A9A60C37**********
X-QF-SIGN: 6FB43AC29175B4602FF95F8332028F19

code=A6A49A6******DFE94EA95032&note=add_note&syssn=20190722000200020081075691
```


> The above command returns JSON structured like this:

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

### HTTP Request

**Endpoint** : `/trade/v1/add_note`  
**Method** : `POST`


### Request Parameters

| Attribute | Mandatory | Type        | Description                                                                 |
|-----------|-----------|-------------|-----------------------------------------------------------------------------|
| `code`    | Yes       | String(32)  | Merchant app code, provided by QFPay                                        |
| `syssn`   | Yes       | String(40)  | QFPay transaction number (generated upon successful transaction)            |
| `note`    | Yes       | String(200) | Remarks or notes. Will be shown on Merchant Portal and transaction reports  |


### Response Parameters

| Attribute   | Type        | Description                                      |
|-------------|-------------|--------------------------------------------------|
| `resperr`   | String(128) | Transaction result message                       |
| `respmsg`   | String(128) | Error message                                    |
| `respcd`    | String(4)   | Return code (`0000` = Interface call successful) |
| `syssn`     | String(40)  | QFPay transaction number                         |
