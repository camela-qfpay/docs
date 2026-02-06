---
id: app-call-app
sidebar_label: App 呼叫 App Android SDK
title: App 呼叫 App Android SDK
description: 本文件說明如何整合 QFPay HaoJin Android SDK，以實現 App 呼叫 App 的聚合收款功能，支援付款、退款、查詢、預授權、卡清算等常見場景。
---

import Link from '@docusaurus/Link';

# App call App Android SDK

## 最新版本更新紀錄

**version 2.3.4.jar**

1、SDK 支援指定正掃或者反掃支付：
   - CollectionReq.SCAN_TYPE_SCAN：掃碼支付
   - CollectionReq.SCAN_TYPE_QRCODE：QR_CODE 支付

設定方法：
```java
CollectionReq req = new CollectionReq(Long.parseLong(money));
req.setScan_type(scan_type); // 設定掃碼方式：CollectionReq.SCAN_TYPE_SCAN / CollectionReq.SCAN_TYPE_QRCODE
```

:::note
若商戶希望強制使用 QR 碼主掃模式（例如展示靜態碼供客戶掃描），建議使用 `SCAN_TYPE_QRCODE`。預設為後置鏡頭掃描。
::: 

---

## 簡介

HaoJin 是一款為商戶提供聚合收款服務的手機應用程式。本文件說明 HaoJin 對外開放的介面，協助第三方應用整合收單功能。

HaoJin 支援以下功能：

1. 發起付款、退款、查詢多筆交易記錄、交易明細。
2. 查詢交易摘要與通道配置。
3. 處理卡類交易的查詢／取消／調整等流程。

<Link href="/img/android/architecture__diagram.png" target="_blank">
  ![Introduction](@site/static/img/android/architecture__diagram.png)
</Link>

---

## 安裝與配置

### 權限設定
請於 `AndroidManifest.xml` 添加以下權限（需安裝 HaoJin App 才能取得授權）：

```xml
<uses-permission android:name="com.qfpay.haojin.permission.OPEN_API"/>
```

### 整合 JAR
將 `qfpay_haojin_api_x.x.x.jar` （[在此下載最新版本](@site/static/files/qfpay_haojin_api_2.3.6.zip)） 放入 `/libs` 目錄，並在 `build.gradle` 引入。

### 設定目標應用 ID
```java
Config.setTargetAppId("in.haojin.nearbymerchant.oversea");
```

### 4. Proguard 觀察規則

請在 `proguard-rules.pro` 添加：

```proguard
-dontnote com.qfpay.haojin.model.**
-keep class com.qfpay.haojin.model.** {*;}
```

### 第三方接口調用範例

#### 收單

調用收單請求：

```java
ITradeAPI mTradeApi = TradeApiFactory.createTradeApi(XXXActivity, this);

// 建立收單請求：CollectionReq 為 SDK 類別
CollectionReq collectionReq = new CollectionReq(100);

// 設定 API 請求參數：如掃碼方式、外部訂單號、卡超時時間等
collectionReq.setScan_type(CollectionReq.SCAN_TYPE_SCAN);  // API 參數：掃碼方式
collectionReq.setOut_trade_no("EXT202312345");            // API 參數：外部訂單號
collectionReq.setWait_card_timeout(120);                  // API 參數：刷卡等待秒數
collectionReq.setPay_method("card_payment");              // API 參數：支付方式
collectionReq.setCamera_id(0);                            // API 參數：鏡頭選擇（0: 後鏡頭）

// 發送請求：SDK 方法
int ret = mTradeApi.doTrade(collectionReq);

@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data)
{
    super.onActivityResult(requestCode, resultCode, data);

    CollectionResp collectionResp =
            (CollectionResp) mTradeApi.parseResponse(requestCode, resultCode, data);

    if (collectionResp == null) {
        return;
    }

    if (collectionResp.isSuccess()) {
        Transaction transaction = collectionResp.getPayResult();
    } else {
        // 處理錯誤
        Log.e(TAG, "onActivityResult: collection error message is " +
                collectionResp.getErrorMsg());
    }
}
```

:::warning
回傳結果可能為 `null`，請務必進行非空判斷後再存取資料欄位。
:::

#### 退款

調用退款請求：

```java
ITradeAPI mTradeApi = TradeApiFactory.createTradeApi(XXXActivity, this);

// RefundReq 為 SDK 類別，構造時帶入 QF 訂單號
RefundReq refundReq = new RefundReq(qfOrderId); // HaoJin 返回的訂單 ID

// 發送退款請求（SDK 方法）
int ret = mTradeApi.doTrade(refundReq);
```

解析返回值：

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data)
{
    super.onActivityResult(requestCode, resultCode, data);

    RefundResp refundResp = (RefundResp) mTradeApi.parseResponse(requestCode,
            resultCode, data);

    if (refundResp == null) {
        return;
    }

    if (refundResp.isSuccess()) {
        Transaction transaction = refundResp.getRefundResult();
    } else {
    }
}
```

#### 查詢多筆交易

調用查詢請求：

```java
// 建立請求類別：GetTransListReq 為 SDK 類別
GetTransListReq getTransListReq = new GetTransListReq();

// 以下為 API 參數設定
getTransListReq.setChannels(selectedChannel); // 支付通道，如 weixin/alipay
getTransListReq.setTypes(selectedType);       // 支付類型，如 payment/refund
getTransListReq.setMonth(month);              // 按月查詢
getTransListReq.setStartTime(startTime);      // 自定義開始時間
getTransListReq.setEndTime(endTime);          // 自定義結束時間
getTransListReq.setPageSize(pageSize);        // 分頁大小
getTransListReq.setPageNum(pageNum);          // 分頁頁碼

// 發送請求（SDK 方法）
int ret = mTradeApi.doTrade(getTransListReq);
```

:::note
1. 請確認通道是否支援。
2. 僅支援兩種交易類型（付款、退款）。
3. 自定義時間區間查詢優先於按月查詢。
4. 時間格式為 “yyyy-MM-dd HH:mm:ss”。
5. 月份格式為 “yyyyMM”。
6. 分頁起始為第 1 頁。
:::

:::note
若傳入時間區間與月份查詢條件，SDK 將優先使用時間區間。月份查詢適合整月報表，時間區間適合精確統計。
:::

解析返回值：

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data)
{
    super.onActivityResult(requestCode, resultCode, data);

    GetTransListResp getTransListResp =
            (GetTransListResp) mTradeApi.parseResponse(requestCode, resultCode, data);

    if (getTransListResp == null) {
        return;
    }

    if (getTransListResp.isSuccess()) {
        List<Transaction> transactions = getTransListResp.getTransList();
    } else {
    }
}
```

#### 查詢交易明細

調用查詢請求：

```java
// 建立查詢請求：GetTransReq（SDK 類別）
GetTransReq getTransReq = new GetTransReq(qfOrderId); // API 參數：訂單號

// 可選：使用外部訂單號查詢（API 參數）
getTransReq.setOut_trade_no("EXT20230123");

// 發送請求（SDK 方法）
int ret = mTradeApi.doTrade(getTransReq);
```

解析返回值：

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    GetTransResp getTransResp =
            (GetTransResp) mTradeApi.parseResponse(requestCode, resultCode, data);

    if (getTransResp == null) {
        return;
    }

    if (getTransResp.isSuccess()) {
        Transaction transaction = getTransResp.getTrans();
    } else {
        // 錯誤處理
    }
}
```

#### 查看交易摘要

調用查看交易摘要請求：

```java
CheckTradeSumReq checkTradeSumReq = new CheckTradeSumReq();

int ret = mTradeApi.doTrade(checkTradeSumReq);
```

解析返回值：

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    CheckTradeSumResp checkTradeSumResp = (CheckTradeSumResp)
            mTradeApi.parseResponse(requestCode, resultCode, data);
}
```

#### 查詢交易通道配置（已棄用）

:::danger
此接口已被標記為棄用，請改用 `GetUserConfigReq` 接口以獲得相同功能。
:::

調用查詢請求：

```java
GetChannelConfigReq channelConfigReq = new GetChannelConfigReq();

int ret = getTradeApi().doTrade(channelConfigReq);
```

解析返回值：

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    GetChannelConfigResp getChannelConfigResp = (GetChannelConfigResp)
            mTradeApi.parseResponse(requestCode, resultCode, data);

    if (getChannelConfigResp == null) {
        return;
    }

    if (getChannelConfigResp.isSuccess()) {
        List<Channel> channels = getChannelConfigResp.getChannels();
    }
}
```

#### 查詢客戶配置信息

調用查詢請求：

```java
GetUserConfigReq getUserConfigReq = new GetUserConfigReq();

int ret = getTradeApi().doTrade(getUserConfigReq);
```

解析返回值：

```java
UserConfig userConfig = getUserConfigResp.getUserConfig();

if (userConfig == null) {
    Log.e(TAG, "handleChannelsResp: get user config info failed.");
    return;
}

List<Channel> channels = userConfig.getTransChannels();
int currencyCode = userConfig.getCurrency();
```

:::warning
若未能正確取得 `userConfig`，通常表示授權異常或帳號配置不完整，請聯絡技術支持。
:::

#### 預授權交易扣款

:::note
預授權交易適用於飯店入住、租借設備等場景，可先凍結金額，完成服務後再扣款或取消。
:::

呼叫預授權交易扣款請求：

```java
// 建立請求類別：PreAuthTransDeductReq（SDK 類別）
PreAuthTransDeductReq req = new PreAuthTransDeductReq(transId); // API 參數：原交易 ID

// 發送請求（SDK 方法）
int ret = mTradeApi.doTrade(req);

if (ret != Config.ResponseCode.SUCCESS) {
    // 錯誤處理
}
```

解析返回值：

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    PreAuthTransDeductResp deductResp = (PreAuthTransDeductResp)
            mTradeApi.parseResponse(requestCode, resultCode, data);

    if (deductResp.isSuccess()) {
        Log.i(TAG, "onActivityResult: success");
    }
}
```

#### 預授權交易取消

呼叫預授權交易取消請求：

```java
// 建立取消請求：PreAuthTransCancelReq（SDK 類別）
PreAuthTransCancelReq preAuthTransCancelReq = new PreAuthTransCancelReq(transId);

// 發送請求（SDK 方法）
int ret = mTradeApi.doTrade(preAuthTransCancelReq);

if (ret != Config.ResponseCode.SUCCESS) {
    // 錯誤處理
}
```

解析返回值：

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    PreAuthTransCancelResp cancelResp = (PreAuthTransCancelResp)
            mTradeApi.parseResponse(requestCode, resultCode, data);

    if (cancelResp.isSuccess()) {
        Log.i(TAG, "onActivityResult: success");
    }
}
```

#### 預授權交易列表

呼叫預授權交易列表請求：

```java
int pageSize = 10;
int pageNum = 1;

// 建立列表請求：PreAuthTransListReq（SDK 類別）
PreAuthTransListReq preAuthTransListReq = new PreAuthTransListReq(pageSize, pageNum);

// 發送請求（SDK 方法）
int ret = mTradeApi.doTrade(preAuthTransListReq);

if (ret != Config.ResponseCode.SUCCESS) {
    // 錯誤處理
}
```

解析返回值：

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    PreAuthTransListResp transListResp =
            (PreAuthTransListResp) mTradeApi.parseResponse(requestCode, resultCode, data);

    List<PreAuthTransactions> transactions = transListResp.getTransList();
}
```

#### 預授權交易詳情

呼叫預授權交易詳情請求：

```java
String transId = "123123123123";

PreAuthTransDetailReq preAuthTransDetailReq = new PreAuthTransDetailReq(transId);

int ret = getTradeApi().doTrade(preAuthTransDetailReq);

if (ret != Config.ResponseCode.SUCCESS) {
    // 錯誤處理
}
```

解析返回值：

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    PreAuthTransDetailResp transDetailResp =
            (PreAuthTransDetailResp) mTradeApi.parseResponse(requestCode, resultCode, data);

    PreAuthTransaction transaction = transDetailResp.getTrans();
}
```

#### 卡退款

發起卡退款請求：

```java
ITradeAPI mTradeApi = TradeApiFactory.createTradeApi(XXXActivity.this);

CardRefundReq cardRefundReq = new CardRefundReq(qfOrderId); // HaoJin 訂單 ID

int ret = mTradeApi.doTrade(cardRefundReq);
```

解析返回結果：

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
    } else {
        // 錯誤處理邏輯
    }
}
```

#### 查詢多筆卡交易

```java
GetCardTransListReq cardTransListReq = new GetCardTransListReq();
int ret = mTradeApi.doTrade(cardTransListReq);
```

解析返回結果：

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    GetTransListResp getTransListResp =
        (GetTransListResp) mTradeApi.parseResponse(requestCode, resultCode, data);

    if (getTransListResp == null) {
        return;
    }

    if (getTransListResp.isSuccess()) {
        List<Transaction> transactions = getTransListResp.getTransList();
    } else {
        // 錯誤處理邏輯
    }
}
```

#### 查詢卡交易詳情

```java
GetCardTransReq getCardTransReq = new GetCardTransReq(orderId);
int ret = mTradeApi.doTrade(getCardTransReq);
```

解析返回結果：

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    GetTransResp getTransResp =
        (GetTransResp) mTradeApi.parseResponse(requestCode, resultCode, data);

    if (getTransResp == null) {
        return;
    }

    if (getTransResp.isSuccess()) {
        Transaction transaction = getTransResp.getTrans();
    } else {
        // 錯誤處理邏輯
    }
}
```

#### 卡調整

```java
CardAdjustReq cardAdjustReq = new CardAdjustReq(orderId);
int ret = mTradeApi.doTrade(cardAdjustReq);
```

解析返回結果：

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    CardAdjustResp cardAdjustResp = (CardAdjustResp)
        mTradeApi.parseResponse(requestCode, resultCode, data);

    if (cardAdjustResp == null) {
        return;
    }

    if (cardAdjustResp.isSuccess()) {
        Transaction transaction = cardAdjustResp.getCardTrans();
    } else {
        // 錯誤處理邏輯
    }
}
```

#### 卡清算

```java
CardSettleReq cardSettleReq = new CardSettleReq();
int ret = mTradeApi.doTrade(cardSettleReq);
```

解析返回結果：

```java
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    CardSettleResp cardSettleResp = (CardSettleResp)
        getTradeApi().parseResponse(requestCode, resultCode, data);

    if (cardSettleResp == null) {
        return;
    }

    List<SettleData> settleDataList = cardSettleResp.getSettleDataList();
}
```

# 附錄

## 交易資訊欄位說明

| 欄位名稱 | 類型   | 是否必填 | 說明                                   |
|----------|--------|----------|----------------------------------------|
| `id` | String | 是 | 交易識別碼 |
| `amt` | Long | 是 | 交易金額 |
| `time` | String | 是 | 交易時間（格式：yyyy-MM-dd HH:mm:ss） |
| `channel` | String | 是 | 交易通道（如 weixin、alipay） |
| `status` | Integer | 是 | 交易狀態（詳見下方交易狀態表） |
| `type` | String | 是 | 交易類型（payment 為支付、refund 為退款） |
| `originId` | String | 否 | 原始交易識別碼（若為退款必填） |
| `mchntName` | String | 是 | 商戶名稱 |
| `remarks` | String | 否 | 備註 |
| `confirmCode` | String | 否 | 交易確認碼 |
| `operatorAccount` | String | 是 | 操作員帳號 |
| `appCode` | String | 否 | 刷卡交易申請碼 |
| `customerId` | String | 否 | 客戶錢包身份（預授權交易用） |
| `customerAccount` | String | 否 | 客戶錢包帳戶（預授權交易用） |
| `completeTransId` | String | 否 | 預授權完成後的新交易 ID |
| `completeTime` | String | 否 | 預授權完成的時間（格式：yyyy-MM-dd HH:mm:ss） |

## 通道資訊欄位說明

| 欄位名稱 | 類型   | 是否必填 | 說明     |
|----------|--------|----------|----------|
| `name` | String | 是 | 通道名稱 |
| `desc` | String | 是 | 通道描述 |

## 交易狀態代碼表

| 狀態碼 | 說明         |
|--------|--------------|
| `0`    | 正常交易     |
| `-1` / `-2` | 等待支付 |
| `-3`   | 交易失敗     |
| `1`    | 沖正         |
| `2`    | 撤銷         |
| `3`    | 退款         |
| `4`    | 部分退款     |
| `5`    | 預授權凍結   |
| `6`    | 預授權解凍   |
| `7`    | 預授權完成   |

## 結果代碼說明

| 結果碼 | 說明                     |
|--------|--------------------------|
| `-1`   | 未知錯誤                 |
| `0`    | 成功                     |
| `100`  | 客戶端錯誤               |
| `101`  | 金額錯誤                 |
| `102`  | AppId 為空               |
| `103`  | 訂單 Id 為空             |
| `104`  | 其他參數為空             |
| `105`  | 用戶取消                 |
| `106`  | 網路錯誤                 |
| `107`  | 用戶未登入               |
| `108`  | 應用未安裝               |
| `109`  | 啟動應用失敗             |
| `110`  | 不支援 API 調用         |
| `111`  | 時間段錯誤               |
| `112`  | 不允許跨月查詢           |
| `113`  | 取得配置資訊失敗         |
| `114`  | 卡交易調整失敗           |
| `115`  | 裝置不支援刷卡           |
| `116`  | 密碼輸入錯誤             |
| `200`  | 伺服器錯誤               |
| `201`  | 訂單 ID 不存在           |
| `202`  | 交易失敗                 |
| `203`  | 餘額不足                 |
| `204`  | 交易確認中               |
| `205`  | 登入狀態過期             |
| `206`  | 退款確認中               |
| `207`  | 退款失敗                 |

## 歷史版本紀錄

### version 2.3.3.jar
1、Transaction 類別新增 getOut_trade_no 與 getCardscheme 方法

---
### version 2.3.2.jar
#### 一、交易相關：
1. 交易支援傳入外部訂單號 out_trade_no。
2. 可設定刷卡等待超時時間 wait_card_timeout（預設為 120 秒，必須大於 0）。

設定方式：
```java
CollectionReq req = new CollectionReq(Long.parseLong(money));
req.setWait_card_timeout(wait_card_timeout);
req.setOut_trade_no(out_trade_no);
```

#### 二、查詢相關：
1. 支援使用 out_trade_no 查詢交易資料。
設定方式：
```java
GetTransReq req = new GetTransReq(order_id);
req.setOut_trade_no(out_trade_no);
```

:::warning
該欄位必須為正整數，若未設定或設定錯誤（如小於等於 0），將自動套用預設 120 秒。
:::

---
### version 2.3.1.jar

1、SDK 支援設定具體支付方式：
透過 CollectionReq.setPay_method 指定支付方式

`pay_method` 可設定值：
  - `card_payment`: 信用卡
  - `wx`: 微信
  - `alipay`: 支付寶
  - `payme`: PayMe
  - `union`: 銀聯
  - `fps`: FPS
  - `octopus`: 八達通
  - `unionpay_card`: 銀聯卡
  - `amex_card`: 美國運通

:::danger
若帳號未開通指定支付方式，即使設定了 `pay_method`，仍會彈出支付方式選擇視窗。
:::

2、支援設定相機：
透過 `CollectionReq.setCamera_id` 設定前／後鏡頭（預設後鏡頭）

camera_id 可用值：
  - `0`：CAMERA_PARAM_BACK（後鏡頭）
  - `1`：CAMERA_PARAM_FRONT（前鏡頭）

:::danger
僅支援 `0`（後鏡頭）與 `1`（前鏡頭），其他值將被忽略，預設為後鏡頭。
:::

設定方式範例：
```java
CollectionReq req = new CollectionReq(Long.parseLong(money));
req.setPay_method(current_paymethod);
req.setCamera_id(current_camera);
```