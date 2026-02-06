---
title: "Element SDK"
---

# Element SDK

QFPay's **Element SDK** allows you to build your own checkout flows using prebuilt UI components provided by QFPay. It offers a flexible, front-end integration method suitable for merchants who want a more customisable experience while still relying on secure, hosted logic.

This guide explains how to integrate QFPay's hosted Element SDK into your website or app.

---

## Overview of Integration Flow

![Element Sequence diagram](https://www.plantuml.com/plantuml/png/VLF1ZXCn3BtdAwozmuAuzO1s0Qs4458H7r0bgQTZTPBCEiukvUjn9bEbhO1BbTX-p--zPXwoM9OI9cEz90PVigI033OlPpDhcppDDWhSVKVsOpqzSOg2SG-VH_J7L0Isze1t5HM6Vs0-MNzKI1jorqC_dhRs13XXGBt-_F9jcNeUjFAtmKvLXvpUZ36sI8ebE6HJbSERZwfb0_wiK0rIYYOCIyTTT1YV2whNu6gh4MgRqGh2R4-BAAg6nSIaDQR3AEPn-tK3zsj_jug_Vtb_tv2xSsT5rhWgsZ1AuVWVukiElD8qWKF0NpCnxhKC7zv1e5W4SwSDxcnvNT2okYPhzbko6wsHa9teDuAtl8SXST1rCjwYMg8TE5H9CgumYXLeQxpNqK_aZv2B2oJWYaYApUQ4WvWARqK82geEASmjHNNfJX3MfzCztgX_IKVKcufzwrCSYCDsrJsKw8MkzhLKTeMdwXCqIMBq0dFXEMNiInRw_X3MBFepQVKB8NqWbqaQlcNGUpvLRsgiJiqfwi83fp833N2XZ39alA4uA-qVfwHBp2kwJB8OC2lfOpv5FtAAgUHgYWRoxV_fueFhwdBn7lFDQELxq9yIfZy0)

1. Create a payment intent using QFPay's API.
2. Initialise the SDK with `QFpay.config()`.
3. Select the payment mode: credit card form (`elements.createEnhance`) or multi-wallet interface (`elements.createWallet`).
4. Collect payment details from the customer.
5. Confirm payment with QFPay backend using `confirmPayment()` or `confirmWalletPayment()`.

---

## Supported Payment Methods

The following payment types are supported through Element SDK:

- Alipay (Mainland China, Hong Kong)
- WeChat Pay
- UnionPay / QuickPass
- FPS
- PayMe
- Visa / MasterCard
- Apple Pay
- Visa / MasterCard Pre-Authorization

---

## Integration Steps

### Step 1: Import SDK JavaScript Library

Add the following `<script>` tag in your HTML to load the SDK:

```html
<!-- Sandbox environment -->
<script src="https://cdn-int.qfapi.com/qfpay_element/qfpay.js"></script>

<!-- Live test environment -->
<script src="https://test-cdn-hk.qfapi.com/qfpay_element/qfpay.js"></script>

<!-- Production environment -->
<script src="https://cdn-hk.qfapi.com/qfpay_element/qfpay.js"></script>
```

### Step 2: Initialise SDK

Initialises the SDK with the selected region and environment.

```js
const qfpay = QFpay.config({
  region: 'hk',
  env: 'prod'
});
```

| Parameter | Required | Values                   | Description                                                                             |
| --------- | -------- | ------------------------ | --------------------------------------------------------------------------------------- |
| `region`  | No       | `hk` \| `qa` \| `hkt`    | Region code. `hk` for Hong Kong (production), `hkt` for live test, or `qa` for sandbox. |
| `env`     | No       | `prod` \| `test` \| `qa` | Environment setting: production, live test (`test`), or sandbox (`hkt`/`qa`).           |

Returns the global `qfpay` object for further operations.

---

### Step 3: Create Payment Intent (Backend API)

**Endpoint**: `/payment_element/v1/create_payment_intent`\
**Method**: POST

**Headers**:

| Header       | Required | Description   |
| ------------ | -------- | ------------- |
| X-QF-APPCODE | Yes      | Merchant code |
| X-QF-SIGN    | Yes      | Merchant key  |

**Request Parameters**:

| Parameter    | Required | Description                                               |
| ------------ | -------- | --------------------------------------------------------- |
| txamt        | Yes      | Amount in cents. Recommended \> 200                       |
| txcurrcd     | No       | Currency, e.g. HKD                                        |
| pay_type     | Yes      | Payment type code                                         |
| out_trade_no | Yes      | transaction number / Merchant platform transaction number |
| mchid        | No       | Merchant ID (for agents only)                             |
| return_url   | No       | Success redirect URL                                      |
| failed_url   | No       | Failure redirect URL                                      |
| notify_url   | No       | Notification webhook                                      |

**Response**:

```json
{
  "respcd": "0000",
  "payment_intent": "38aec7ce...",
  "intent_expiry": "2026-01-01 12:00:00"
}
```

### Step 4: Retrieve Payment Intent (Frontend)

After creating the payment intent via API, call the following method on the frontend to validate it:

```js
const qfpay = QFpay.config();
const result = qfpay.retrievePaymentIntent();

if (result.code === '0000') {
  console.log('Payment intent is valid');
} else {
  console.error('Invalid or expired payment intent');
}
```

---

### Step 5: Configure Appearance (Optional)

Initialise the UI component handler for rendering card form or wallet interface. You can pass an optional `appearance` configuration object to customise the look and feel.

```js
const appearance = {
  // Optional theme preset
  theme: 'night',

  // Custom styling variables
  variables: {
    fontFamily: 'cursive',
    fontWeight: '400',
    colorText: 'black',
    sizeFontSubTitle: 'inherit',
    colourBackground: '#fff',
    colourPrimary: '#ced4da',
    colourComponentText: 'purple',
    sizeComponentText: '15px',
    colourErrorMessage: '#da5d4a',
    sizeErrorMessage: 'inherit',
    colorPaymentButton: '#000000',
    colorPaymentButtonText: '#FFFFFF',
    colorQRCodeTopPromptContent: '#000000',
    colorQRCodeBottomPromptContent: '#000000',
    fontWeightQRCodeTopPrompt: '900',
    fontWeightQRCodeBottomPrompt: '300'
  },

  // Optional billing address field visibility
  billingAddressDisplay: {
    city: true,
    address1: true,
    address2: true
  }
};

const elements = qfpay.element(appearance);
```

This `elements` object can then be used to create either a wallet list or card form UI on your checkout page.

---

### Step 6: Render Payment UI

#### Render Wallet + Card UI (Enhanced Tabbed Interface) via `elements.createEnhance()`

Renders an enhanced multi-wallet interface.

```js
const elements = qfpay.element(appearance);
elements.createEnhance({
  selector: '#container',
  email: true,
  tab: true,
  element: 'payment',
  lang: 'en'});
```

> **Return value**: This function returns **void**. The UI component is mounted directly to the specified DOM element.

#### Render Credit Card Form Only via `elements.create()`

Use for Visa / Mastercard card form.

```js
elements.create("#container")
```

> **Return value**: This function returns **void**. The UI component is mounted directly to the specified DOM element.

### Step 7: Execute Payment

#### Collect Card Payment via `payment.pay()`

```js
payment.pay({
  goods_name: 'Premium Product',
  paysource: 'payment_element',
  customer_id: 'CUST123456',      // optional
  token_expiry: '2026-01-01',     // optional
  token_reason: 'Recurring Setup',// optional
  token_reference: 'INV-0987'     // optional
}, intentParams.payment_intent)
```

> **Return value**: This function returns **void**. It is used only to initialise form parameters; no response object will be returned.

#### Collect Multi-Wallet + Card Payments via `payment.walletPay()`

This method supports initiating payments using a variety of wallet options (e.g. Alipay, WeChat Pay, FPS, UnionPay) along with credit card support in a combined interface. Merchants can control which payment methods to show and style the resulting UI using `elements.createWallet()`.

```js
payment.walletPay({
  lang: 'en', // Optional: 'en', 'zh-cn', 'zh-hk'; defaults to browser language
  goods_info: 'Premium membership', // Optional: product details
  goods_name: 'VIP Access',         // Optional: short product name
  paysource: 'payment_element_checkout', // Required: fixed value
  out_trade_no: intentParams.out_trade_no, // Required: merchant order ID (from backend)
  txamt: intentParams.txamt,               // Required: amount in cents
  txcurrcd: intentParams.txcurrcd,         // Required: currency code (e.g. 'HKD')
  support_pay_type: [ // Optional: list of supported methods
    'Alipay',                  // Alipay Mainland
    'AlipayHK',                // Alipay Hong Kong
    'WeChat',                  // WeChat Pay
    'FPS',                     // Faster Payment System
    'UnionPay',                // UnionPay/QuickPass
    'PayMe',                   // PayMe (HK)
    'VisaMasterCardPayment',   // Credit Card
    'ApplePay',                // Apple Pay
    'VisaMasterCardPreAuth'    // Pre-Authorisation
  ],
  // Optional Tokenisation Fields:
  customer_id: 'CUST123456',           // Customer ID for saving card
  token_expiry: '2026-01-01',      // Token expiration date
  token_reason: 'Subscription',    // Token usage reason
  token_reference: 'SUB20241101'   // External reference
}, intentParams.payment_intent);
```

:::note

- `txamt`, `txcurrcd`, and `out_trade_no` should be sourced from your backend when generating the Payment Intent.
- If no `support_pay_type` is provided, all eligible payment methods will be shown.
- This method does **not** return a value; use `qfpay.confirmWalletPayment()` to complete the payment flow. :::

After invoking this method, use `elements.createWallet()` to render the UI into a DOM container.

> **Return value**: This function returns **void**. It is used only to initialise form parameters; no response object will be returned.

---

### Step 8: Confirm Payment

#### Confirm Credit Card / Apple Pay Payment

Use `confirmPayment()` after rendering the form to finalise the transaction. You may pass an optional return URL.

```js
const paymentResponse = qfpay.confirmPayment({
  return_url: 'https://example.com'
})
```

**Response Code Interpretation:**

| Code | Meaning | Note |
| --- | --- | --- |
| 0000 | Payment Success | |
| 1111 | Cancelled (Apple Pay) | Applicable to Apple Pay only |
| other | Payment Failed | See `description` field |


#### Confirm Multi-Wallet Payment

This method confirms wallet-based payments initiated via the Element SDK, including Alipay, FPS, WeChat, etc.

- If `return_url` is passed, the user is redirected to it after completing payment.
- If not provided, the page remains on the current screen.

Successful transactions return:

- `code`: `0000` indicates success; other values should be checked via description
- `description`: Message explaining the result
- `out_trade_no`: Merchant order number
- `syssn`: QFPay transaction ID

```js
const paymentResponse = qfpay.confirmWalletPayment({
  return_url: 'https://example.com'
})
```

---

### Step 9: Query Transaction Result

Query past transaction status.

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

## Example Use Cases

### Example 1: Credit Card Form Integration

```js
const qfpay = QFpay.config();
const payment = qfpay.payment();

payment.pay({
  goods_name: 'Socks',
  paysource: 'payment_element',
  // Applicable for tokenisation:
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

// Check result
if (response.code === '0000') {
  console.log('Payment successful');
} else {
  console.error('Payment failed', response.description);
}
```

### Example 2: Wallet + Card UI (Multi-Payment)

```js
qfpay.retrievePaymentIntent();

payment.walletPay({
  lang: 'en', // 'zh-cn', 'zh-hk', or 'en'. Defaults to browser language.
  goods_info: 'Gift Card',
  goods_name: 'Gift Card',
  paysource: 'payment_element_checkout',
  out_trade_no: intentParams.out_trade_no,
  txamt: intentParams.txamt,
  txcurrcd: intentParams.txcurrcd,
  support_pay_type: ['Alipay', 'WeChat', 'UnionPay', 'FPS', 'VisaMasterCardPayment', 'ApplePay'],
  // Applicable for tokenisation
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

## Notes

:::tip If `lang` is not passed, Element will use the browser’s default language. :::

:::warning The Element container must not be placed inside a `<form>` element or rendering will break. :::

:::info In production, the payment intent expires in 2 years. In sandbox: 7 days. :::