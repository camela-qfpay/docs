---
id: payment-element
title: 支付組件 (Element) SDK
sidebar_label: 支付組件 (Element) SDK
description: 使用 QFPay 的 Element SDK，在您的網站中嵌入卡支付或多錢包 UI，保留靈活度的同時確保安全性。
---
# 支付組件 (Element) SDK

QFPay 的 **支付組件 SDK** 讓您能夠使用 QFPay 提供的預建 UI 元件，自行建立結帳流程。這是一種靈活的前端整合方式，適合希望在保留安全託管邏輯的同時，擁有更多自訂體驗的商戶。

本指南將說明如何將 QFPay 的託管支付組件 SDK 整合至您的網站或應用程式中。

---

## 整合流程總覽

![Element Sequence diagram](https://www.plantuml.com/plantuml/png/VLF1ZXCn3BtdAwozmuAuzO1s0Qs4458H7r0bgQTZTPBCEiukvUjn9bEbhO1BbTX-p--zPXwoM9OI9cEz90PVigI033OlPpDhcppDDWhSVKVsOpqzSOg2SG-VH_J7L0Isze1t5HM6Vs0-MNzKI1jorqC_dhRs13XXGBt-_F9jcNeUjFAtmKvLXvpUZ36sI8ebE6HJbSERZwfb0_wiK0rIYYOCIyTTT1YV2whNu6gh4MgRqGh2R4-BAAg6nSIaDQR3AEPn-tK3zsj_jug_Vtb_tv2xSsT5rhWgsZ1AuVWVukiElD8qWKF0NpCnxhKC7zv1e5W4SwSDxcnvNT2okYPhzbko6wsHa9teDuAtl8SXST1rCjwYMg8TE5H9CgumYXLeQxpNqK_aZv2B2oJWYaYApUQ4WvWARqK82geEASmjHNNfJX3MfzCztgX_IKVKcufzwrCSYCDsrJsKw8MkzhLKTeMdwXCqIMBq0dFXEMNiInRw_X3MBFepQVKB8NqWbqaQlcNGUpvLRsgiJiqfwi83fp833N2XZ39alA4uA-qVfwHBp2kwJB8OC2lfOpv5FtAAgUHgYWRoxV_fueFhwdBn7lFDQELxq9yIfZy0)

1.	使用 QFPay 的 API 建立 Payment Intent（支付意圖）。
2.	使用 `QFpay.config()` 初始化 SDK。
3.	選擇支付模式：信用卡表單（`elements.createEnhance`）或多錢包介面（`elements.createWallet`）。
4.	向顧客收集支付資料。
5.	使用 QFPay 後端 API 進行確認（`confirmPayment()` 或 `confirmWalletPayment()`）。

---

## 支援的支付方式

支付組件 SDK 支援以下支付方式：
- 支付寶（中國大陸、香港）
- 微信支付
- 銀聯 / 雲閃付
- 轉數快（FPS）
- PayMe
- Visa / MasterCard
- Apple Pay
- Visa / MasterCard 預授權

---

## 整合步驟

### 步驟一：引入 SDK JavaScript 檔案

請在您的 HTML 中加入以下 `<script>` 標籤以載入 SDK：

```html
<!-- 測試環境（Sandbox） -->
<script src="https://cdn-int.qfapi.com/qfpay_element/qfpay.js"></script>

<!-- 線上測試環境（Live Test） -->
<script src="https://test-cdn-hk.qfapi.com/qfpay_element/qfpay.js"></script>

<!-- 正式環境（Production） -->
<script src="https://cdn-hk.qfapi.com/qfpay_element/qfpay.js"></script>
```


### 步驟二：初始化 SDK

初始化 SDK 並設定所需的區域與環境參數。
```js
const qfpay = QFpay.config({
  region: 'hk',
  env: 'prod'
});
```

| 參數       | 是否必填 | 可選值                         | 說明                                                                 |
|------------|----------|--------------------------------|----------------------------------------------------------------------|
| `region`   | 否       | `hk` \| `qa` \| `hkt`          | 區域代碼：`hk` 表示香港正式環境，`hkt` 表示線上測試環境，`qa` 表示沙盒環境。 |
| `env`      | 否       | `prod` \| `test` \| `qa`       | 環境設定：`prod` 為正式環境，`test` 為測試環境，`qa` 為沙盒環境。         |

回傳全域的 `qfpay` 物件，用於後續操作。

---

### 步驟三：建立 Payment Intent（後端 API）

**端點**：`/payment_element/v1/create_payment_intent`  
**方法**：POST  

**標頭**（Headers）：

| 標頭名稱       | 是否必填 | 說明             |
|----------------|----------|------------------|
| X-QF-APPCODE   | 是       | 商戶代碼         |
| X-QF-SIGN      | 是       | 商戶金鑰         |

**請求參數（Request Parameters）**：

| 參數名稱       | 是否必填 | 說明                                  |
|----------------|----------|---------------------------------------|
| txamt          | 是       | 支付金額（以分為單位），建議大於 200   |
| txcurrcd       | 否       | 幣別，例如 HKD                        |
| pay_type       | 是       | 支付類型代碼                          |
| out_trade_no   | 是       | 訂單號 / 商戶平台訂單編號             |
| mchid          | 否       | 商戶 ID（代理商適用）                 |
| return_url     | 否       | 支付成功跳轉網址                      |
| failed_url     | 否       | 支付失敗跳轉網址                      |
| notify_url     | 否       | 支付結果通知 webhook                  |

**回應範例（Response）**：
```json
{
  "respcd": "0000",
  "payment_intent": "38aec7ce...",
  "intent_expiry": "2026-01-01 12:00:00"
}
```


### 步驟四：驗證 Payment Intent（前端）

在後端透過 API 建立 payment intent 後，請在前端呼叫下列方法進行驗證：

```js
const qfpay = QFpay.config();
const result = qfpay.retrievePaymentIntent();

if (result.code === '0000') {
  console.log('Payment intent is valid');
} else {
  console.error('Invalid or expired payment intent');
}
```
此方法將驗證該 payment intent 是否有效且可用。

---

### 步驟五：設定外觀樣式（可選）

初始化 UI 元件處理器，用於渲染卡片表單或錢包介面。你可以傳入 `appearance` 設定物件來自訂外觀風格。

```js
const appearance = {
  // 預設主題（可選）
  theme: 'night',

  // 自訂樣式變數
  variables: {
    fontFamily: 'cursive', // 字體
    fontWeight: '400', // 字體粗細
    colorText: 'black', // 標題文字顏色
    sizeFontSubTitle: 'inherit', // 副標文字尺寸
    colourBackground: '#fff', // 背景顏色
    colourPrimary: '#ced4da', // 主要按鈕或邊框顏色
    colourComponentText: 'purple', // 表單文字顏色
    sizeComponentText: '15px', // 表單文字大小
    colourErrorMessage: '#da5d4a', // 錯誤提示訊息顏色
    sizeErrorMessage: 'inherit', // 錯誤訊息文字大小
    colorPaymentButton: '#000000', // 支付按鈕背景色
    colorPaymentButtonText: '#FFFFFF', // 支付按鈕文字顏色
    colorQRCodeTopPromptContent: '#000000', // 二維碼上方提示文字顏色
    colorQRCodeBottomPromptContent: '#000000', // 二維碼下方提示文字顏色
    fontWeightQRCodeTopPrompt: '900', // 上方提示字體粗細
    fontWeightQRCodeBottomPrompt: '300' // 下方提示字體粗細
  },

  // 帳單地址欄位是否顯示（可選）
  billingAddressDisplay: {
    city: true, // 顯示城市與郵遞區號欄位
    address1: true, // 顯示地址欄位一
    address2: true // 顯示地址欄位二
  }
};

const elements = qfpay.element(appearance);
```

此 `elements` 物件可用於建立錢包清單介面或信用卡輸入表單。

---

### 渲染付款介面

#### 使用 `elements.createEnhance()` 建立錢包 + 信用卡介面（進階選單式）

建立一個可切換的多支付方式錢包介面。

```js
const elements = qfpay.element(appearance);
elements.createEnhance({
  selector: '#container',
  email: true,
  tab: true,
  element: 'payment',
  lang: 'en'});
```

> **回傳值**：此函式 不會回傳任何值。元件將直接掛載至指定的 DOM 元素。

#### 使用 `elements.create()` 僅建立信用卡表單

用於 Visa / Mastercard 卡支付。

```js
elements.create("#container")
```

> **回傳值**：此函式 不會回傳任何值。元件將直接掛載至指定的 DOM 元素。

### 步驟七：執行付款

#### 使用 `payment.pay()` 收集信用卡支付資料
```js
payment.pay({
  goods_name: 'Premium Product',
  paysource: 'payment_element',
  customer_id: 'CUST123456',      // 可選
  token_expiry: '2026-01-01',     // 可選
  token_reason: 'Recurring Setup',// 可選
  token_reference: 'INV-0987'     // 可選
}, intentParams.payment_intent)
```
> **回傳值**：此函式不會回傳任何結果。僅用於初始化表單參數，無返回值。


#### 使用 `payment.walletPay()` 收集錢包 + 信用卡支付資料

此方法可支援多種錢包支付方式（如支付寶、微信、FPS、銀聯）與信用卡同時出現在介面中。商戶可以自定義要顯示的支付方式，並透過 `elements.createWallet()` 來渲染 UI。

```js
payment.walletPay({
  lang: 'en', // 可選：'en'、'zh-cn'、'zh-hk'，預設為瀏覽器語言
  goods_info: 'Premium membership', // 可選：產品詳細說明
  goods_name: 'VIP Access',         // 可選：商品名稱
  paysource: 'payment_element_checkout', // 必填：固定值
  out_trade_no: intentParams.out_trade_no, // 必填：商戶訂單號（來自後端）
  txamt: intentParams.txamt,               // 必填：交易金額（單位：分）
  txcurrcd: intentParams.txcurrcd,         // 必填：幣種代碼（例如 'HKD'）
  support_pay_type: [ // 可選：顯示指定支付方式
    'Alipay',                  // 支付寶（中國大陸）
    'AlipayHK',                // 支付寶香港
    'WeChat',                  // 微信支付
    'FPS',                     // 轉數快
    'UnionPay',                // 銀聯 / 快捷通
    'PayMe',                   // PayMe（香港）
    'VisaMasterCardPayment',   // 信用卡
    'ApplePay',                // Apple Pay
    'VisaMasterCardPreAuth'    // 信用卡預授權
  ],
  // 可選：卡片代儲參數
  customer_id: 'CUST123456',
  token_expiry: '2026-01-01',
  token_reason: 'Subscription',
  token_reference: 'SUB20241101'
}, intentParams.payment_intent);
```

:::note
- `txamt`、`txcurrcd` 與 `out_trade_no` 應由後端在建立 Payment Intent 時一併生成。
- 如未指定 `support_pay_type`，將自動顯示所有已開通支付方式。
- 此方法不會回傳任何結果；需配合 `qfpay.confirmWalletPayment()` 完成整個付款流程。
:::

呼叫完 `walletPay()` 後，使用 `elements.createWallet()` 把支付 UI 插入到畫面容器中。

> **回傳值**：此函式不會回傳任何結果。僅用於初始化參數。

---

### 步驟八：確認付款
#### 確認信用卡 / Apple Pay 付款

在渲染信用卡表單後，使用 `confirmPayment()` 來完成交易流程。你可以選擇性地傳入 `return_url` 作為跳轉頁面。

```js
const paymentResponse = qfpay.confirmPayment({
  return_url: 'https://example.com'
})
```

**回應碼說明：**

| 代碼   | 含義       | 備註                      |
|--------|--------------------|-------------------------------|
| 0000   | 付款成功    |                               |
| 1111   | 用戶取消（Apple Pay） | 僅適用於 Apple Pay |
| 其他  | 付款失敗   | 詳見 `description` 欄位      |


#### 確認多錢包付款

此方法用於確認由 Element SDK 發起的錢包支付（如支付寶、轉數快、微信等）。
- 如果提供 `return_url`，用戶付款完成後將會跳轉到該頁面。
- 若無提供，則會停留在當前頁面。

成功交易將返回以下欄位：
- `code`: 若為 0000 則表示成功，其他代碼請根據 `description` 欄位判斷
- `description`: 描述交易結果的訊息
- `out_trade_no`: 商戶訂單號
- `syssn`: QFPay交易 ID

```js
const paymentResponse = qfpay.confirmWalletPayment({
  return_url: 'https://example.com'
})
```

---

### 步驟九：查詢交易結果
用於查詢過往交易的狀態與結果。

```js
const inquiryResponse = payment.inquiry({
  syssn: '20251201...',
  out_trade_no: intentParams.out_trade_no,
  pay_type: 'VisaMasterCardPayment',
  respcd: '0000',
  start_time: '2025-01-01 00:00:00',
  end_time: '2025-01-31 23:59:59'
}, intentParams.payment_intent)
```
此方法將返回符合條件的歷史交易記錄。

## 範例應用場景

### 範例 1：信用卡表單整合（Credit Card Form Integration）

```js
const qfpay = QFpay.config();
const payment = qfpay.payment();

payment.pay({
  goods_name: 'Socks',
  paysource: 'payment_element',
  // Token 模式
  customer_id: 'CUST123456',
  token_expiry: '2026-01-01',
  token_reason: 'membership_renewal',
  token_reference: 'user_456'
}, intentParams.payment_intent);

const elements = qfpay.element();
elements.createEnhance({
  selector: '#container'
});

const response = qfpay.confirmPayment({
  return_url: 'https://example.com/success'
});

// 檢查結果
if (response.code === '0000') {
  console.log('Payment successful');
} else {
  console.error('Payment failed', response.description);
}
```

### 範例 2：錢包 + 信用卡整合介面（多支付方式）
```js
qfpay.retrievePaymentIntent();

payment.walletPay({
  lang: 'en', // 可選值：'zh-cn'、'zh-hk' 或 'en'。預設為瀏覽器語言。
  goods_info: 'Gift Card',
  goods_name: 'Gift Card',
  paysource: 'payment_element_checkout',
  out_trade_no: intentParams.out_trade_no,
  txamt: intentParams.txamt,
  txcurrcd: intentParams.txcurrcd,
  support_pay_type: ['Alipay', 'WeChat', 'UnionPay', 'FPS', 'VisaMasterCardPayment', 'ApplePay'],
  // 適用於Token
  customer_id: 'CUST123456', 
  token_expiry: '2026-01-01',
  token_reason: 'signup_bonus',
  token_reference: 'token001'
}, intentParams.payment_intent);

const elements = qfpay.element({
  variables: {
    colourComponentText: 'black',
    colorQRCodeTopPromptContent: '#000000',
    fontWeightQRCodeTopPrompt: '900',
    colorPaymentButton: '#000000',
    colorPaymentButtonText: '#FFFFFF'
  },
  billingAddressDisplay: {
    city: true,
    address1: true,
    address2: true
  }
});

elements.createWallet({ selector: '#container' });

qfpay.confirmWalletPayment({ return_url: 'https://example.com/success' });
```

---

## 注意事項

:::tip
若未傳入 `lang` 參數，系統將自動採用瀏覽器預設語言。:::

:::warning
Element 容器不可嵌套於 `<form>` 標籤內，否則元件將無法正確渲染。:::

:::info
付款意圖的有效期限：正式環境為 2 年，沙盒環境為 7 天。:::
