import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# Merchant Present Mode (MPM)

<Link href="/img/mpm_flow_en.jpeg" target="_blank">![MPM process-flow](@site/static/img/mpm_flow_en.jpeg)</Link>

## MPM API Request

**Endpoint** : `/trade/v1/payment`  
**Method** : `POST`  

### Supported PayTypes

| PayType | Description |
| ------- | ----------- |
| 800101  | Alipay MPM (Overseas Merchants) |
| 801501  | Alipay MPM (HK Merchants) |
| 800201  | WeChat MPM (Overseas & HK Merchants) |
| 805801  | PayMe MPM (HK Merchants) |
| 800701  | UNIONPAY Quick Pass MPM |
| 802001  | FPS MPM (HK Merchants) |
| 803701  | Octopus Dynamic QRC MPM (HK Merchants) |

**Scenario**: The merchant generates a dynamic QR code based on Alipay or WeChat Pay protocol and displays it for the customer to scan. Customers complete the payment by scanning the QRC using their wallets. This applies to both offline POS and online browser-initiated flows.

:::tip Signature Note
Refer to [Signature Generation](/docs/api-reference/signature-generation) for signing instructions.
:::

### Example Request

```http
POST /trade/v1/payment HTTP/1.1
Content-Type: application/x-www-form-urlencoded
X-QF-APPCODE: A6A49A66B4C********94EA95032
X-QF-SIGN: 3b020a6349646684ebeeb0ec2cd3d1fb

expired_time=10&goods_name=qfpay&limit_pay=no_credit&mchid=R1zQrTdJnn&out_trade_no=Native20190722145741431794b8d1&pay_type=800201&txamt=20&txcurrcd=HKD&txdtm=2019-07-22 14:57:42&udid=AA
```

### Example Response

```json
{
  "surcharge_fee": 0,
  "qrcode": "https://qr.alipay.com/bax03190uxd47wbekffy6033",
  "pay_type": "800101",
  "surcharge_rate": 0,
  "resperr": "success",
  "txdtm": "2020-04-23 11:09:24",
  "out_trade_no": "364ZK6BAJGYHMU3TUX0X7MGIGQL4O8KI",
  "syssn": "20200423066200020000976054",
  "sysdtm": "2020-04-23 11:09:27",
  "txcurrcd": "EUR",
  "respmsg": "",
  "chnlsn2": "",
  "cardcd": "",
  "udid": "qiantai2",
  "txamt": "1",
  "respcd": "0000",
  "chnlsn": ""
}
```

## Request Parameters

| Field | Required | Type | Description |
| ----- | -------- | ---- | ----------- |
| [Public Parameters](/docs/api-reference/request-format#public-payment-request-parameters) | — | — | Common fields like `mchid`, `txamt`, etc. |
| `pay_tag` | No | String(16) | Wallet specifier. Default: `ALIPAYHK`, Mainland Alipay: `ALIPAYCN` |
| `expired_time` | No | String(3) | QR code expiry in minutes (5–120). Default is 30. Only for WeChat/Alipay |
| `limit_pay` | No | String | Use `no_credit` to disable credit cards. Applicable to CN wallets only. |

## Response Parameters

| Field | Type | Description |
| ----- | ---- | ----------- |
| `qrcode` | String(512) | Generated QRC URL |
| [Public Parameters](/docs/api-reference/response-format#field-description) | — | Common fields like `respcd`, `syssn`, etc. |