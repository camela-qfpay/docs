import Link from '@docusaurus/Link';

# Apple Pay 支援

## 簡介

QFPay Checkout 與 Payment Element 均支援 Apple Pay 作為嵌入式支付選項。商戶需先於 QFPay 系統啟用 Apple Pay，消費者才能在支援裝置上選用此方式付款。Apple Pay 通常可提升授權成功率，並提供流暢的結帳體驗。

---

## 支援裝置

Apple Pay 可於下列平台安全便捷地使用：
- iPhone、iPad、Mac（Safari 瀏覽器）
- Windows 等平台之第三方瀏覽器（透過 QRCode 掃描）

:::info
Android 裝置不支援 Apple Pay，若用戶於 Android 裝置選擇 Apple Pay，按鈕將自動隱藏。
:::

---

## QR Code 支付模式

若裝置不支援原生 Apple Pay（例如 Windows 桌機），系統會自動顯示一個 QR Code。用戶可透過支援 Apple Pay 的 iPhone 或 iPad 掃碼完成付款（需 iOS 版本 ≥ 18）。

:::warning
若使用者將其 Apple Store 區域或系統語言設定為中國大陸，此功能將無法使用。
:::

<Link href="https://sdk.qfapi.com/images/applepay_qrcode.png" target="_blank">![Apple Pay QRCode 範例](@site/static/img/applepay_qrcode.png)</Link>