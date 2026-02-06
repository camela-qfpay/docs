---
id: recurring-api-reference
title: Subscription API Reference
description: This page lists the API reference for creating, updating, and managing recurring (subscription) payment resources.
sidebar_label: Subscription API Reference
---

# Subscription API Reference

This page documents the available API endpoints for recurring payments. Make sure you have read the [Recurring Overview](/docs/online-shop/integration-by-use-case/recurring-payments/recurring-overview) before using these APIs.

---

## 1. Create Customer

**POST** `/customer/v1/create`

Creates a new customer profile.

### Request Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `name` | String | No | Customer name |
| `phone` | String | No | Contact number |
| `email` | String | No | Email address |
| `billing_address` | String (JSON) | No | JSON string of billing address |

### Response
Returns the `customer_id`.

---

## 2. Update Customer

**POST** `/customer/v1/update`

Updates an existing customer.

### Request Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `customer_id` | String | Yes | QFPay system customer ID |
| `name` | String | No | Customer name |
| `phone` | String | No | Contact number |
| `email` | String | No | Email address |
| `billing_address` | JSON | No | JSON object of billing address |

### Response
Returns number of affected rows.

---

## 3. Query Customers

**POST** `/customer/v1/query`

Search for customers.

### Request Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `customer_id` | String | No | Customer ID |
| `name` | String | No | Customer name |
| `phone` | String | No | Contact number |
| `email` | String | No | Email address |
| `page` | Int | No | Default = 1 |
| `page_size` | Int | No | Default = 10, max = 100 |

### Response
List of matching customers.

---

## 4. Delete Customer

**POST** `/customer/v1/delete`

Deletes a customer. Irreversible.

### Request Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `customer_id` | String | Yes | Customer ID to delete |

### Response
Returns deleted customer ID and row count.

---

## 5. Create Product

**POST** `/product/v1/create`

Creates a recurring billing product.

### Request Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `name` | String | Yes | Product name |
| `type` | String | Yes | `recurring` |
| `description` | String | No | Product description |
| `txamt` | Int | Yes | Amount in cents (100 = $1) |
| `txcurrcd` | String | Yes | Currency (e.g. HKD) |
| `interval` | String | Yes | Billing cycle: `monthly`, `yearly`, etc. |
| `interval_count` | Int | Yes | Interval count (e.g. 1 month = 1) |
| `usage_type` | String | Yes | `licensed` |

:::note
In sandbox environment, `minutes` and `hours` can be used in the `interval` field for testing
:::

### Response
Returns generated `product_id`.

---

## 6. Update Product

**POST** `/product/v1/update`

Updates product metadata.

### Request Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `product_id` | String | Yes | Product ID to update |
| `name` | String | No | New name |
| `description` | String | No | New description |

### Response
Returns number of affected rows.


---

## 7. Query Products

**POST** `/product/v1/query`

Search for products.

### Request Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `product_id` | String | No | Product ID |
| `name` | String | No | Product name |
| `description` | String | No | Description |
| `txcurrcd` | String | No | Currency |
| `interval` | String | No | `monthly` / `yearly` |
| `page` | Int | No | Default = 1 |
| `page_size` | Int | No | Default = 10, max = 100 |

### Response
List of matching products.

---

## 8. Delete Product

**POST** `/product/v1/delete`

Deletes a product (must not be linked to subscriptions).

### Request Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `product_id` | String | Yes | Product ID to delete |

### Response
Returns deleted product ID and row count.

---

## 9. Create Subscription

:::tip
You must create `customer_id`, `product_id`, and `token_id` before calling the subscription API.
:::

**POST** `/subscription/v1/create`

Creates a recurring subscription plan.

### Request Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `customer_id` | String | Yes | Linked customer ID |
| `token_id` | String | Yes | Linked payment token |
| `products` | List | Yes | Product ID and quantity |
| `total_billing_cycles` | Int | No | Total number of billing periods (null = indefinite) |
| `start_time` | String | No | ISO timestamp to start subscription |

### Parameters inside products
 Attribute    | Type   | Mandatory | Description                               |
| ------------ | ------ | --------- | ----------------------------------------- |
| `product_id` | String | Yes       | Unique product identifier in QFPay system |
| `quantity`   | Int    | No        | Default value=1                           |

### Example Request
```json
{
  "products": [
    {
      "product_id": "prod_54c3772d******9a54b236e09ec74f",
      "quantity": 1
    }
  ],
  "customer_id": "cust_aaf6aae94******982c54c9cae5ba32",
  "token_id": "tk_a99892fd*********d3417d168a18bb",
  "total_billing_cycles": 2,
  "start_time": "2020-05-14 12:32:56"
}
```

### Response Parameters (in `data` field)
Returns `subscription_id` and initial state.
| Attribute              | Type   | Description                                                                 |
| ---------------------- | ------ | --------------------------------------------------------------------------- |
| `subscription_id`          | String | Unique subscription identifier in QFPay system (e.g. sub_xxxxxxx)                                   |
| `state` | String | Initial state of the subscription. Possible values: ACTIVE, INCOMPLETE, COMPLETED |
---

## 10.Update Subscription

Update an existing subscription.

**Endpoint**: `/subscription/v1/update`  
**Method**: `POST`

### Request Parameters

| Attribute              | Type   | Mandatory | Description                                                                                   |
| ---------------------- | ------ | --------- | --------------------------------------------------------------------------------------------- |
| `subscription_id`      | String | Yes       | Unique subscription identifier in QFPay system                                                |
| `total_billing_cycles` | Int    | No        | Total billing cycles. If null, the subscription will run indefinitely                         |
| `start_time`           | String | No        | The time at which the subscription starts. Determines the first billing time                  |
| `token_id`             | String | No        | Unique payment token identifier in QFPay system                                               |
| `products`             | Object | No        | List of product objects. Each object includes `product_id` and optional `quantity`            |

### Response Parameters (in `data` field)

| Attribute         | Type   | Description                                  |
| ----------------- | ------ | -------------------------------------------- |
| `subscription_id` | String | Unique subscription ID                       |
| `rowAffected`     | Int    | Number of records updated                    |

---

## 11. Query Subscription {#query-subscription}

Query subscription objects.

**Endpoint**: `/subscription/v1/query`  
**Method**: `POST`

### Request Parameters

| Attribute         | Type   | Mandatory | Description                                      |
| ----------------- | ------ | --------- | ------------------------------------------------ |
| `page`            | Int    | No        | Page number. Default = 1                         |
| `page_size`       | Int    | No        | Page size. Default = 10. Max = 100               |
| `subscription_id` | String | No        | Unique subscription ID in QFPay system           |
| `customer_id`     | String | No        | Unique customer ID in QFPay system               |
| `state`           | String | No        | Subscription state (e.g. `INCOMPLETE`, `ACTIVE`, etc.) |

### Response Parameters (in `data` field)

An array of subscription objects containing the following attributes:

| Attribute                     | Type   | Description                                                                 |
| ----------------------------- | ------ | --------------------------------------------------------------------------- |
| `subscription_id`             | String | Unique subscription ID                                                      |
| `customer_id`                 | String | Unique customer ID                                                          |
| `token_id`                    | String | Unique payment token ID                                                     |
| `products`                    | Object | List of product items (`product_id` + optional `quantity`)                 |
| `total_billing_cycles`        | Int    | Total billing cycles (null if unlimited)                                   |
| `state`                       | String | Subscription state                                                          |
| `next_billing_time`           | String | Time of the next billing event                                              |
| `last_billing_time`           | String | Time of the previous billing event                                          |
| `completed_billing_iteration` | Int    | Number of completed billing cycles                                          |
| `start_time`                  | String | Subscription start time                                                     |

### Sample Response
```json
{
  "resperr": "success",
  "respcd": "0000",
  "respmsg": "success",
  "data": [
    {
      "total_billing_cycles": 3,
      "last_billing_time": "2024-11-21T11:12:06Z",
      "is_available": 1,
      "userid": 2510351,
      "last_billing_status": "SUCCESS",
      "state": "ACTIVE",
      "products": [
        { "product_id": "prod_8efecd0bd******b9aa1ec5ec01", "quantity": 1 }
      ],
      "retry_attempts": 0,
      "completed_iteration": 1,
      "token_id": "tk_9ac510017*******69b614e8f7ee",
      "subscription_id": "sub_e120378de*******da066f690da75",
      "customer_id": "cust_5ba1539f*******c9bda11d12c854e36",
      "next_billing_time": "2024-11-21T11:13:06Z"
    }
  ]
}
```

---

## 12. Cancel Subscription

Cancel a subscription immediately.

**Endpoint**: `/subscription/v1/cancel`  
**Method**: `POST`

### Request Parameters

| Attribute         | Type   | Mandatory | Description                            |
| ----------------- | ------ | --------- | -------------------------------------- |
| `subscription_id` | String | Yes       | Unique subscription ID in QFPay system |

---

## 13. Query Subscription Orders

Query all billing orders under a specific subscription.

**Endpoint**: `/subscription/billing_order/v1/list`  
**Method**: `POST`

### Request Parameters

| Attribute         | Type   | Mandatory | Description                            |
| ----------------- | ------ | --------- | -------------------------------------- |
| `subscription_id` | String | Yes       | Unique subscription ID                 |
| `page`            | Int    | No        | Page number. Default = 1               |
| `page_size`       | Int    | No        | Page size. Default = 10, Max = 100     |

### Response Parameters (in `data` field)

Array of subscription billing orders:

| Attribute               | Type   | Description                                                                                       |
| ----------------------- | ------ | ------------------------------------------------------------------------------------------------- |
| `subscription_order_id` | String | Unique order ID, format: `sub_ord_{subscription_id}_{0001}` for the N-th billing of subscription |
| `subscription_id`       | String | Subscription ID this billing belongs to                                                           |
| `trigger_by`            | String | Trigger type: `auto` (QF system) or `manual`                                                      |
| `sequence_no`           | Int    | Iteration number of this billing in the subscription                                              |

---

## 14. Manual Charge a Subscription

Trigger a manual charge on a failed subscription order.

**Endpoint**: `/subscription/v1/charge`  
**Method**: `POST`

### Request Parameters

| Attribute             | Type   | Mandatory | Description                                                    |
| --------------------- | ------ | --------- | -------------------------------------------------------------- |
| `subscription_id`     | String | Yes       | Unique subscription ID                                         |
| `subscription_order_id` | String | No      | Specific order ID to retry (optional, for fine control)        |

:::note
This API is only valid for subscriptions with a failed billing and a current state of `UNPAID`, `INCOMPLETE`, or `PAST_DUE`.

- If the manual charge is **before** the next billing time, the subscription remains `ACTIVE`.
- If the charge is **after** the next billing time, the subscription will be **cancelled**.
- If the manual charge fails, the subscription state remains **unchanged**.
:::

