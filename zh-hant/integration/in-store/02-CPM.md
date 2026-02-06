import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# 消費者出示碼模式（反掃）（CPM）

<Link href="/img/cpm_flow_chin.jpeg" target="_blank">![CPM process-flow](@site/static/img/cpm_flow_chin.jpeg)</Link>

## CPM API 請求

**端點** : `/trade/v1/payment`  
**請求方法** : `POST`  

## 支援的支付類型

| 編碼 | 說明 |
| ------------------ | ---- |
| 800008  | 微信、支付寶、銀聯雲閃付、PayMe 反掃支付 |
| 800108  | 支付寶跨境反掃支付 |
| 800208  | 微信反掃支付 |
| 801008  | 微信香港反掃支付（適用於向微信香港申請的商戶） |
| 805808  | PayMe 反掃支付 |
| 800708  | 銀聯雲閃付反掃支付 |

**說明**：顧客打開其電子錢包（例如 WeChat、Alipay），展示動態二維碼，並出示給商戶進行掃描。此流程僅用於線下實體付款場景。若回傳代碼為 `1143/1145`，表示交易仍在處理中，或顧客正輸入密碼 — 商戶應[查詢交易狀態](/docs/common-api/transaction-enquiry)。

:::tip Signature Note
請參考 [簽名生成](/docs/api-reference/signature-generation) 了解簽名生成方式。
:::

### 請求示例

```http
POST /trade/v1/payment HTTP/1.1
Content-Type: application/x-www-form-urlencoded
X-QF-APPCODE: A6A49A66B4C********94EA95032
X-QF-SIGN: 3b020a6349646684ebeeb0ec2cd3d1fb

auth_code=13485790*******88557&goods_name=qfpay&mchid=R1zQrTdJnn&out_trade_no=Native201907221520536a25477909&pay_type=800208&txamt=10&txcurrcd=HKD&txdtm=2019-07-22 15:20:54&udid=AA
```

### 回應示例

```json
{
  "pay_type": "800108",
  "sysdtm": "2019-07-22 15:20:54",
  "paydtm": "2019-07-22 15:20:56",
  "txdtm": "2019-07-22 15:20:54",
  "udid": "AA",
  "txcurrcd": "EUR",
  "txamt": 10,
  "resperr": "交易成功",
  "respmsg": "OK",
  "out_trade_no": "201907221520536a25477909",
  "syssn": "20190722000300020081074842",
  "respcd": "0000",
  "chnlsn": "4200000384201907223585006133"
}
```

## 請求參數

| 欄位 | 必填 | 類型 | 描述 |
| ----- | ---- | ---- | ---- |
| [公共支付請求參數](/docs/api-reference/request-format#公共支付請求參數) | — | — | 通用欄位，例如 `mchid`、`txamt` 等。 |
| `auth_code` | 是（僅限 CPM） | String(128) | 從顧客的錢包條碼/二維碼掃描得到的授權碼。每筆交易唯一。Alipay/WeChat 可在條碼下方找到。 |

## 回應參數

| 欄位 | 類型 | 描述 |
| ----- | ---- | ---- |
| [通用回應格式](/docs/api-reference/response-format#欄位說明) | — | 包含欄位如 `respcd`、`syssn` 等。 |