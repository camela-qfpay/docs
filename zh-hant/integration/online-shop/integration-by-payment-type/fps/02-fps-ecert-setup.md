---
id: fps-ecert-setup
title: FPS 電子憑證申請與設定指南
description: FPS App-to-App 互聯付款中 SSL 憑證申請與設定指南，特別適用於 HSBC 商戶。
sidebar_label: FPS 電子憑證設定指南
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

:::info
本指南適用於使用 App-to-App 付款的 FPS 商戶，特別是與滙豐銀行（HSBC）進行對接的商戶。
:::

---

## 何時需要申請電子證書？

如您於 FPS App-to-App 隨付功能中使用專屬域名和 Universal Link，則必須按規格使用電子憑證來保護交易安全。

銀行如 HSBC 針對 FPS 商戶會要求域名立證與組織名稱配對。

---

## FPS App-to-App 憑證重要說明

:::warning
依據 FPS 技術規範第 6.9.2 節，憑證的 **組織名稱 (Organisation Name)** 必須與 FPS 地址服務中登記的 **收款人名稱 (Payee Name)** 完全一致（不分大小寫與空格）。

> Payment App MUST validate the organization name inside the subject field of the merchant’s certificate (in x509 format) matches the payee name returned from the FPS addressing service.


此名稱會出現在 X.509 憑證的 subject 欄位中。

雖然 **憑證的網域名稱（CN 欄位）無需與組織名稱一致**，但該網域會由 QFPay 根據整合情境指派與設定，商戶無法自行決定。

若使用多個子網域（如 `fps.payment.example-shop.com`），每個網域都需單獨申請 e-Cert，將產生額外費用與等待時間。

商戶必須在其 DNS 設定中新增以下 **CNAME 記錄**：
```
主機名稱（Host）：fps.merchant.com
類型（Type）：CNAME
指向（Value）：hk.qfapi.com
```
:::
---

## FPS 電子憑證申請流程

| 步驟 | 說明 |
|------|------|
| 1 | 填寫申請表格 CPos 798F |
| 2 | 攜帶文件親身前往任何香港郵政局遞交申請 |
| 3 | 出示授權代表身份證明，並繳交年費 |
| 4 | 當場或郵寄取得 PIN 信封，用以提交 CSR |
| 5 | 登入電子憑證網站提交 CSR |
| 6 | 等待約 10 個工作天審核，包含網域與電郵驗證 |
| 7 | 審核通過後憑 CSR 核發憑證 |
| 8 | 下載電子憑證並安裝於您的 HTTPS 伺服器上 |
| 9 | 將**憑證檔案與私鑰檔案**提交給 QFPay 技術支援團隊，以進行 FPS 支付網址（Pay URL）設定 |

---

## CSR 產生指令

在提交申請前，需先用 OpenSSL 產生 CSR。例子：

```
openssl req -new -SHA256 -newkey rsa:2048 -nodes \
-keyout <密鑰名稱>.key \
-out <憑證名稱>.csr \
-subj "/C=HK/ST=HongKong/L=HongKong/O=<您的組織名稱>/OU=/CN=<您的域名>"
```

#### 參數說明

| 項目                 | 說明                       |
| ------------------ | ------------------------ |
| `-newkey rsa:2048` | 產生新的 2048-bit RSA 金鑰     |
| `-nodes`           | 移除秘銷，不加密 private key     |
| `-keyout`          | 設定 private key 儲存檔名      |
| `-out`             | 指定產生的 CSR 檔名             |
| `-subj`            | 憑證 subject 內容，包括縣市、組織名稱等 |

> **備註：**
> * `O=` 項目必須與 FPS 給予的收款商戶名稱精確配對
> * `CN=` 域名由 QFPay 根據實際場景設置
> * `OU=` 可留空，如無特殊部門要求

---

## 必備文件

* 完成的申請表 CPos 798F
* 商業發展證 (BR) 複本
* 公司設立證 (CI) 複本
* 域名擁有證明 (e.g. 訂單發票，DNS 畫面截圖)

---

## 憑證簽發後的責任

:::info
此外，香港郵政會在憑證過期前 **30 天與 14 天** 發送電郵提醒至商戶註冊的電郵地址，請商戶留意郵件並準時辦理續期。

憑證核發完成後，商戶需將 **私鑰 (.key) 與憑證檔案 (.crt 或 .cer)** 提供給 QFPay 技術支援團隊，以便完成 FPS 支付網址的後端配置。
:::
---

## FPS 技術標準據點

【第 6.9.2 章：憑證驗證機制】
銀行支付應用程式將檢查 X.509 憑證中的 Organisation Name (O) 是否與 FPS 收款商戶名稱配對

* 配對方式為不分大小寫
* 空格也不算一種差異

如未配對將無法執行付款

---

## 參考資料

* [e-Cert 申請表格 798F](https://www.ecert.gov.hk/product/ecert/apply/certapply.html#t4)
* [申請流程圖 PDF](https://www.ecert.gov.hk/product/ecert/apply/img/e-Cert_%28S%29_Flow.pdf)
