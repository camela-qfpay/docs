---
id: "app-call-app"
title: "App Call App Android SDK"
sidebar_label: "App Call App Android SDK"
description: "This document explains how to integrate the QFPay HaoJin Android SDK to enable App-to-App payment features, including payment, refund, transaction query, pre-authorisation, and card settlement."
---

## Latest Version Changelog

**version 2.3.4.jar**

1. SDK supports setting scan type:
   - `CollectionReq.SCAN_TYPE_SCAN`: Scan customer QR code
   - `CollectionReq.SCAN_TYPE_QRCODE`: Display QR code for customer to scan

Example:

```java
CollectionReq req = new CollectionReq(Long.parseLong(money));
req.setScan_type(scan_type); // Set scan type: CollectionReq.SCAN_TYPE_SCAN / CollectionReq.SCAN_TYPE_QRCODE
```

## :::note If the merchant wishes to force QR code display mode (e.g. showing a static QR code for the customer to scan), it is recommended to use `SCAN_TYPE_QRCODE`. The default camera used is the rear camera. :::

## Introduction

HaoJin is a mobile app offering integrated payment collection for merchants. This document describes the SDK integration interfaces for third-party apps.

HaoJin supports:

1. Initiating payment, refund, querying multiple transactions and transaction detail.
2. Retrieving transaction summary and channel configuration.
3. Handling card transaction query/cancel/adjustment.

<Link href="/img/android/architecture__diagram.png" target="_blank">
  ![Introduction](@site/static/img/android/architecture__diagram.png)
</Link>

---

## Installation & Setup

### Permission Setup

Add the following to `AndroidManifest.xml` (requires HaoJin App installed):

```xml
<uses-permission android:name="com.qfpay.haojin.permission.OPEN_API"/>
```

### Integrate JAR

Place `qfpay_haojin_api_x.x.x.jar` ([Download Latest](@site/static/files/qfpay_haojin_api_2.3.6.zip)) into `/libs`, and import in `build.gradle`.

### Set Target App ID

```java
Config.setTargetAppId("in.haojin.nearbymerchant.oversea");
```

### Proguard Rules

Add to `proguard-rules.pro`:

```proguard
-dontnote com.qfpay.haojin.model.**
-keep class com.qfpay.haojin.model.** {*;}
```

### API Usage Example

#### Collection

Send a collection request:

```java
ITradeAPI mTradeApi = TradeApiFactory.createTradeApi(XXXActivity, this);

// Create CollectionReq instance
CollectionReq collectionReq = new CollectionReq(100);

// Set parameters
collectionReq.setScan_type(CollectionReq.SCAN_TYPE_SCAN);  // scan mode
collectionReq.setOut_trade_no("EXT202312345");            // external order ID
collectionReq.setWait_card_timeout(120);                  // wait timeout
collectionReq.setPay_method("card_payment");              // payment method
collectionReq.setCamera_id(0);                            // 0 = back camera

// Send request
int ret = mTradeApi.doTrade(collectionReq);

@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    CollectionResp collectionResp =
        (CollectionResp) mTradeApi.parseResponse(requestCode, resultCode, data);

    if (collectionResp == null) {
        return;
    }

    if (collectionResp.isSuccess()) {
        Transaction transaction = collectionResp.getPayResult();
    } else {
        Log.e(TAG, "collection error: " + collectionResp.getErrorMsg());
    }
}
```

:::warning The returned result may be `null`. Always perform a null check before accessing any data fields. :::

#### Refund

Send refund request:

```java
RefundReq refundReq = new RefundReq(qfOrderId); // QF order ID
int ret = mTradeApi.doTrade(refundReq);
```

Parse response:

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    RefundResp refundResp = (RefundResp) mTradeApi.parseResponse(requestCode, resultCode, data);

    if (refundResp == null) {
        return;
    }

    if (refundResp.isSuccess()) {
        Transaction transaction = refundResp.getRefundResult();
    }
}
```

#### Query Multiple Transactions

```java
GetTransListReq getTransListReq = new GetTransListReq();
getTransListReq.setChannels(selectedChannel);
getTransListReq.setTypes(selectedType);
getTransListReq.setMonth(month);
getTransListReq.setStartTime(startTime);
getTransListReq.setEndTime(endTime);
getTransListReq.setPageSize(pageSize);
getTransListReq.setPageNum(pageNum);
int ret = mTradeApi.doTrade(getTransListReq);
```

:::note

1. Please ensure the selected payment channel is supported.
2. Only two transaction types are supported: payment and refund.
3. Custom time range takes precedence over monthly query.
4. Time format must be “yyyy-MM-dd HH:mm:ss”.
5. Month format must be “yyyyMM”.
6. Pagination starts from page 1.\
   :::

:::note If both time range and month are provided, the SDK will prioritise the time range.\
Use monthly queries for full-month reports and time range queries for precise statistics.\
:::

Parse:

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    GetTransListResp getTransListResp =
        (GetTransListResp) mTradeApi.parseResponse(requestCode, resultCode, data);
    if (getTransListResp != null && getTransListResp.isSuccess()) {
        List<Transaction> transactions = getTransListResp.getTransList();
    }
}
```

#### Query Transaction Detail

```java
GetTransReq getTransReq = new GetTransReq(qfOrderId);
getTransReq.setOut_trade_no("EXT20230123");
int ret = mTradeApi.doTrade(getTransReq);
```

Parse:

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    GetTransResp getTransResp =
        (GetTransResp) mTradeApi.parseResponse(requestCode, resultCode, data);
    if (getTransResp != null && getTransResp.isSuccess()) {
        Transaction transaction = getTransResp.getTrans();
    }
}
```

#### View Summary

```java
CheckTradeSumReq checkTradeSumReq = new CheckTradeSumReq();
int ret = mTradeApi.doTrade(checkTradeSumReq);
```

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    CheckTradeSumResp resp = (CheckTradeSumResp)
        mTradeApi.parseResponse(requestCode, resultCode, data);
}
```

#### Deprecated: Get Channel Config

:::danger This API has been deprecated. Please use the `GetUserConfigReq` API to achieve the same functionality. :::

```java
GetChannelConfigReq req = new GetChannelConfigReq();
int ret = getTradeApi().doTrade(req);
```

Parse the return value:

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    GetChannelConfigResp resp = (GetChannelConfigResp)
        mTradeApi.parseResponse(requestCode, resultCode, data);
    if (resp == null) return;
    if (resp.isSuccess()) {
        List<Channel> channels = resp.getChannels();
    }
}
```

---

## Query User Configuration

```java
GetUserConfigReq getUserConfigReq = new GetUserConfigReq();
int ret = getTradeApi().doTrade(getUserConfigReq);
```

Parse response:

```java
UserConfig userConfig = getUserConfigResp.getUserConfig();
if (userConfig == null) {
    Log.e(TAG, "Failed to get user config info.");
    return;
}
List<Channel> channels = userConfig.getTransChannels();
int currencyCode = userConfig.getCurrency();
```

:::warning If `userConfig` cannot be retrieved properly, it usually indicates an authorisation issue or incomplete account configuration. Please contact technical support. :::

---

## Pre-authorization: Deduct

:::note Pre-authorisation transactions are suitable for scenarios such as hotel check-ins or equipment rentals, where an amount can be frozen first and later charged or cancelled after the service is completed. :::

```java
PreAuthTransDeductReq req = new PreAuthTransDeductReq(transId);
int ret = mTradeApi.doTrade(req);
```

Parse response:

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    PreAuthTransDeductResp resp = (PreAuthTransDeductResp)
        mTradeApi.parseResponse(requestCode, resultCode, data);
    if (resp.isSuccess()) {
        Log.i(TAG, "Success");
    }
}
```

## Pre-authorization: Cancel

```java
PreAuthTransCancelReq cancelReq = new PreAuthTransCancelReq(transId);
int ret = mTradeApi.doTrade(cancelReq);
```

Parse response:

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    PreAuthTransCancelResp cancelResp = (PreAuthTransCancelResp)
        mTradeApi.parseResponse(requestCode, resultCode, data);
    if (cancelResp.isSuccess()) {
        Log.i(TAG, "Cancel success");
    }
}
```

## Pre-authorization: List

```java
PreAuthTransListReq listReq = new PreAuthTransListReq(10, 1);
int ret = mTradeApi.doTrade(listReq);
```

Parse response:

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    PreAuthTransListResp listResp = (PreAuthTransListResp)
        mTradeApi.parseResponse(requestCode, resultCode, data);
    List<PreAuthTransactions> transactions = listResp.getTransList();
}
```

## Pre-authorization: Detail

```java
PreAuthTransDetailReq detailReq = new PreAuthTransDetailReq(transId);
int ret = getTradeApi().doTrade(detailReq);
```

Parse response:

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    PreAuthTransDetailResp detailResp = (PreAuthTransDetailResp)
        mTradeApi.parseResponse(requestCode, resultCode, data);
    PreAuthTransaction transaction = detailResp.getTrans();
}
```

---

## Card Refund

```java
CardRefundReq refundReq = new CardRefundReq(qfOrderId);
int ret = mTradeApi.doTrade(refundReq);
```

Parse response:

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    RefundResp refundResp = (RefundResp) mTradeApi.parseResponse(requestCode, resultCode, data);
    if (refundResp != null && refundResp.isSuccess()) {
        Transaction transaction = refundResp.getRefundResult();
    }
}
```

## Card Transactions: List

```java
GetCardTransListReq req = new GetCardTransListReq();
int ret = mTradeApi.doTrade(req);
```

Parse response:

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    GetTransListResp resp = (GetTransListResp)
        mTradeApi.parseResponse(requestCode, resultCode, data);
    List<Transaction> transactions = resp.getTransList();
}
```

## Card Transaction: Detail

```java
GetCardTransReq detailReq = new GetCardTransReq(orderId);
int ret = mTradeApi.doTrade(detailReq);
```

Parse response:

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    GetTransResp getTransResp = (GetTransResp)
        mTradeApi.parseResponse(requestCode, resultCode, data);
    if (getTransResp != null && getTransResp.isSuccess()) {
        Transaction transaction = getTransResp.getTrans();
    }
}
```

## Card Adjustment

```java
CardAdjustReq req = new CardAdjustReq(orderId);
int ret = mTradeApi.doTrade(req);
```

Parse response:

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    CardAdjustResp resp = (CardAdjustResp)
        mTradeApi.parseResponse(requestCode, resultCode, data);
    Transaction transaction = resp.getCardTrans();
}
```

## Card Settlement

```java
CardSettleReq req = new CardSettleReq();
int ret = mTradeApi.doTrade(req);
```

Parse response:

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    CardSettleResp resp = (CardSettleResp)
        mTradeApi.parseResponse(requestCode, resultCode, data);
```

## Reference

### Transaction Information Field Description

| Attribute         | Type    | Mandatory | Description                                                                                               |
| ----------------- | ------- | --------- | --------------------------------------------------------------------------------------------------------- |
| `id`              | String  | Yes       | Transaction idendity number                                                                               |
| `amt`             | Long    | Yes       | Transaction Amount                                                                                        |
| `time`            | String  | Yes       | Transaction time yyy-MM-dd HH:mm:ss                                                                       |
| `channel`         | String  | Yes       | Transaction channel like weixin, alipay                                                                   |
| `status`          | Integer | Yes       | Transaction status                                                                                        |
| `type`            | String  | Yes       | Transaction type, payment or refund                                                                       |
| `originId`        | String  | No        | Original transaction id, mandatory if the transaction is refund                                           |
| `mchntName`       | String  | Yes       | Sore name                                                                                                 |
| `remarks`         | String  | No        | Transaction remarks                                                                                       |
| `confirmCode`     | String  | No        | Transaction confirmation code                                                                             |
| `operatorAccount` | String  | Yes       | Operator name                                                                                             |
| `appCode`         | String  | No        | Application code (swipe card)                                                                             |
| `customerId`      | String  | No        | Idendity of customer wallet (pre-authorization)                                                           |
| `customerAccount` | String  | No        | Account of customer wallet (pre-authorization)                                                            |
| `completeTransId` | String  | No        | Newly generated transaction id when the pre-authorization transaction is completed (pre-authorization)    |
| `completeTime`    | String  | No        | Complete time when the pre-authorization transaction is completed yyyy-MM-dd HH:mm:ss (pre-authorization) |

## Channel Information Field Description

| Attribute | Type   | Mandatory | Description         |
| --------- | ------ | --------- | ------------------- |
| `name`    | String | Yes       | Channel name        |
| `desc`    | String | Yes       | Channel description |

### Transaction Status Field Description

| Transaction Status | Description                 |
| ------------------ | --------------------------- |
| 0                  | Normal transaction          |
| -1/-2              | Waiting for payment         |
| -3                 | Failed                      |
| 1                  | Reversal                    |
| 2                  | Void                        |
| 3                  | Refund                      |
| 4                  | Partial Refund              |
| 5                  | Pre-authorization Frozen    |
| 6                  | Pre-authorization Unfrozen  |
| 7                  | Pre-authorization Completed |

### Result Code Description

| Result Code | Description                          |
| ----------- | ------------------------------------ |
| -1          | Unknown error                        |
| 0           | Success                              |
| 100         | Client error                         |
| 101         | Amount error                         |
| 102         | AppId is empty                       |
| 103         | Order id is empty                    |
| 104         | Other parameter is empty             |
| 105         | User cancel                          |
| 106         | Network error                        |
| 107         | User not logged in                   |
| 108         | Application not installed            |
| 109         | Launch App failed                    |
| 110         | Non-support API invoke               |
| 111         | Time period error                    |
| 112         | Cross-month query not allowed        |
| 113         | Failed to get config info            |
| 114         | Card adjust failed                   |
| 115         | Device does not support card swiping |
| 116         | Password input error                 |
| 200         | Server error                         |
| 201         | Order id does not exist              |
| 202         | Transaction Failed                   |
| 203         | Insufficient account balance         |
| 204         | Transaction is confirming            |
| 205         | Login status expired                 |
| 206         | Refund is confirming                 |
| 207         | Refund Failed                        |

## Version History

### version 2.3.3.jar

1. The `Transaction` class added two new methods: `getOut_trade_no` and `getCardscheme`.

---

### version 2.3.2.jar

#### 1. Transaction Features:

1. Transactions now support passing an external order number (`out_trade_no`).
2. Card swipe timeout (`wait_card_timeout`) can be configured (default is 120 seconds, must be greater than 0).

Example usage:

```java
CollectionReq req = new CollectionReq(Long.parseLong(money));
req.setWait_card_timeout(wait_card_timeout);
req.setOut_trade_no(out_trade_no);
```

#### 2. Query Features:

1. Supports querying transaction data using out_trade_no. Example usage:

```java
GetTransReq req = new GetTransReq(order_id);
req.setOut_trade_no(out_trade_no);
```

:::warning The timeout value must be a positive integer. If not set or set incorrectly (e.g. ≤ 0), it will default to 120 seconds. :::

---

### version 2.3.1.jar

1. SDK now supports specifying payment methods: Set via CollectionReq.setPay_method.

Available `pay_method` values:

- `card_payment`: Credit Card
- `wx`: WeChat Pay
- `alipay`: Alipay
- `payme`: PayMe
- `union`: UnionPay
- `fps`: FPS
- `octopus`: Octopus
- `unionpay_card`: UnionPay Card
- `amex_card`: American Express

:::danger If the account is not enabled for the specified payment method, the SDK will fall back to the payment method selection screen. :::

2. Support for camera selection: Use `CollectionReq.setCamera_id` to specify front or back camera (default is back camera).

Available camera_id values:

- `0`: CAMERA_PARAM_BACK (Back Camera)
- `1`: CAMERA_PARAM_FRONT (Front Camera)

:::danger Only values `0` and `1` are supported. Any other values will be ignored and default to the back camera. :::

Example usage:

```java
CollectionReq req = new CollectionReq(Long.parseLong(money));
req.setPay_method(current_paymethod);
req.setCamera_id(current_camera);
```