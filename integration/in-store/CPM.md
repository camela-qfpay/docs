import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# Consumer Present Mode (CPM)

<Link href="/img/cpm_flow_en.jpeg" target="_blank">![CPM process-flow](@site/static/img/cpm_flow_en.jpeg)</Link>

## CPM API Request

**Endpoint** : `/trade/v1/payment`  
**Method** : `POST`  

### Supported PayTypes

| PayType | Description |
| ------- | ----------- |
| 800008  | WeChat, Alipay, UNIONPAY Quick Pass Consumer Presented QR Code (CPM) |
| 800108  | Alipay CPM (Overseas & HK Merchants) |
| 800208  | WeChat CPM (Overseas & HK Merchants) |
| 801008  | WeChat Pay HK CPM (Direct Settlement, HK Merchants) |
| 805808  | PayMe CPM (HK Merchants) |
| 800708  | UNIONPAY Quick Pass CPM |

**Scenario**: The customer opens their wallet app (e.g. WeChat, Alipay), displays a dynamic QR code, and presents it to the merchant for scanning. This flow is used exclusively for offline, in-person payments. If `1143/1145` is returned, the transaction is still processing, or the customer is entering their password — merchants should [query transaction status](/docs/common-api/transaction-enquiry).

:::tip Signature Note
Refer to [Signature Generation](/docs/api-reference/signature-generation) for signing instructions.
:::

### Example Request

```http
POST /trade/v1/payment HTTP/1.1
Content-Type: application/x-www-form-urlencoded
X-QF-APPCODE: A6A49A66B4C********94EA95032
X-QF-SIGN: 3b020a6349646684ebeeb0ec2cd3d1fb

auth_code=13485790*******88557&goods_name=qfpay&mchid=R1zQrTdJnn&out_trade_no=Native201907221520536a25477909&pay_type=800208&txamt=10&txcurrcd=HKD&txdtm=2019-07-22 15:20:54&udid=AA
```

### Example Response

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

## Request Parameters

| Field | Required | Type | Description |
| ----- | -------- | ---- | ----------- |
| [Public Parameters](/docs/api-reference/request-format#public-payment-request-parameters) | — | — | Common fields like `mchid`, `txamt`, etc. |
| `auth_code` | Yes (CPM only) | String(128) | Authorisation code scanned from consumer’s QR. For Alipay/WeChat it can be found below the barcode in wallet. Unique per transaction. |

## Response Parameters

| Field | Type | Description |
| ----- | ---- | ----------- |
| [Public Parameters](/docs/api-reference/response-format#field-description) | — | Includes fields like `respcd`, `syssn`, etc. |