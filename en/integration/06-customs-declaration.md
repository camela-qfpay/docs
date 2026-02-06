---
id: "customs-declaration"
title: "Customs Declaration API"
description: "Submit cross-border transaction info to customs for Alipay and WeChat transactions."
sidebar_label: "Customs Declaration"
---

Customs Declaration APIs allow merchants to automatically report cross-border eCommerce transactions to the relevant customs authorities. This supports compliance, streamlines clearance procedures, and helps reduce delays in product delivery.

---

## 1. Push Customs Declaration

Use this endpoint to submit customs information to Alipay or WeChat after a successful payment.

### HTTP Request

```http
POST /custom/v1/declare
```

### Request Parameters

| Parameter          | Required | Type        | Description                                                          |
| ------------------ | -------- | ----------- | -------------------------------------------------------------------- |
| `trade_type`       | Yes      | String(8)   | Payment platform: `weixin` or `alipay`                               |
| `syssn`            | Yes      | String(32)  | QFPay transaction number                                             |
| `customs`          | Yes      | String(20)  | Customs bureau code, e.g. `SHANGHAI_ZS`                              |
| `mch_customs_no`   | Yes      | String(20)  | Merchant’s customs registration number                               |
| `action_type`      | No       | String(256) | Declaration type (WeChat only): `"ADD"` (new) or `"MODIFY"` (update) |
| `mch_customs_name` | No       | String(256) | Alipay merchant record name (e.g. `jwyhanguo_card`)                  |
| `out_request_no`   | No       | String(32)  | Unique request ID for Alipay declaration                             |
| `amount`           | No       | String(20)  | Declaration amount (Alipay only), e.g. `2.00`                        |

### Sub-Order Fields (for split or modified declarations)

| Parameter       | Conditional | Type       | Description                                                         |
| --------------- | ----------- | ---------- | ------------------------------------------------------------------- |
| `sub_order_no`  | C           | String(64) | Required if the order is split into sub-orders                      |
| `fee_type`      | C           | String(8)  | Currency (WeChat only). Must be `CNY`                               |
| `order_fee`     | C           | String(8)  | Total sub-order amount in CNY cents = `transport_fee + product_fee` |
| `product_fee`   | C           | String(8)  | Product portion of the sub-order                                    |
| `transport_fee` | C           | String(8)  | Shipping portion of the sub-order                                   |

### Response

| Parameter                    | Type        | Description                                                       |
| ---------------------------- | ----------- | ----------------------------------------------------------------- |
| `syssn`                      | String(40)  | QFPay transaction number                                          |
| `respcd`                     | String(4)   | `0000` = Success<br />`1143/1145` = Pending<br />Others = Failure |
| `resperr`                    | String(128) | Error reason                                                      |
| `respmsg`                    | String(128) | Additional information                                            |
| `verify_department`          | String      | Customs department that processed the declaration                 |
| `verify_department_trade_id` | String      | Trade ID assigned by customs system                               |

---

## 2. Query Customs Declaration

Use this API to check the status of a customs declaration.

### HTTP Request

```
POST /custom/v1/query
GET /custom/v1/query
```

### Request Parameters

| Parameter      | Required | Type       | Description                                |
| -------------- | -------- | ---------- | ------------------------------------------ |
| `trade_type`   | Yes      | String(8)  | Payment platform: `weixin` or `alipay`     |
| `customs`      | Yes      | String(20) | Customs bureau code                        |
| `syssn`        | Yes      | String(32) | QFPay transaction number                   |
| `sub_order_no` | No       | String(40) | Required only for split-order declarations |

### Response

| Parameter | Type        | Description                                                                                                                                                 |
| --------- | ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `syssn`   | String(40)  | QFPay transaction number                                                                                                                                    |
| `respcd`  | String(4)   | Response code                                                                                                                                               |
| `resperr` | String(128) | Error reason                                                                                                                                                |
| `respmsg` | String(128) | Additional info                                                                                                                                             |
| `data`    | Array       | List of sub-declaration records with:<br />- `resperr`<br />- `errmsg`<br />- `sub_order_no`<br />- `verify_department`<br />- `verify_department_trade_id` |

---

## 3. Repush Customs Declaration

Use this when a previous declaration was lost or not received by customs.

### HTTP Request

```
POST /custom/v1/redeclare
```

### Request Parameters

| Parameter        | Required | Type       | Description                                |
| ---------------- | -------- | ---------- | ------------------------------------------ |
| `trade_type`     | Yes      | String(8)  | `weixin` or `alipay`                       |
| `customs`        | Yes      | String(20) | Customs bureau code                        |
| `syssn`          | Yes      | String(32) | QFPay transaction number                   |
| `mch_customs_no` | Yes      | String(20) | Merchant’s customs registration number     |
| `sub_order_no`   | No       | String(40) | Required only for split-order declarations |

### Response

Same format as [Push Customs Declaration](#1-push-customs-declaration)

---

## Additional Notes

:::tip

- All customs declarations must be made **after** a successful transaction (`respcd=0000`).
- For Alipay declarations, ensure you use the correct registered customs name (`mch_customs_name`) and number.
- WeChat declarations may require sub-order splitting and specific currency settings.
- Declarations are mandatory for certain jurisdictions to ensure product release and compliance. :::

For a full list of codes, see [Transaction Status Codes](/docs/api-reference/status-codes).