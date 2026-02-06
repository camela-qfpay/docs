import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# 商戶出示碼模式（正掃）(MPM)

<Link href="/img/mpm_flow_chin.jpeg" target="_blank">![MPM process-flow](@site/static/img/mpm_flow_chin.jpeg)</Link>

## MPM API 請求

**端點** : `/trade/v1/payment`  
**請求方法** : `POST`  

### 支持的支付类型

| 編碼 | 說明 |
| ------- | ----------- |
| 800101  | 支付寶跨境線下掃碼支付|
| 801501  | 支付寶線上掃碼支付 (香港商戶)  |
| 800201  | 微信掃碼支付 |
| 805801  | PayMe 掃碼支付 |
| 800701  | 銀聯雲閃付掃碼支付 |
| 802001  | 香港轉數快掃碼支付 |
| 803701  | 八達通動態二維碼掃碼支付(香港商戶) |

**使用場景**: 商戶根據 Alipay 或 WeChat Pay 協議產生動態 QR 碼並展示給顧客掃描。顧客使用錢包掃描該 QR 碼以完成付款。此模式適用於線下實體店或線上網頁啟動的支付流程。

:::tip 簽名說明
請參考 [簽名生成](/docs/api-reference/signature-generation#%E7%B0%BD%E5%90%8D%E7%94%9F%E6%88%90) 以了解簽名生成的指引。
:::

### 請求範例

```http
POST /trade/v1/payment HTTP/1.1
Content-類型: application/x-www-form-urlencoded
X-QF-APPCODE: A6A49A66B4C********94EA95032
X-QF-SIGN: 3b020a6349646684ebeeb0ec2cd3d1fb

expired_time=10&goods_name=qfpay&limit_pay=no_credit&mchid=R1zQrTdJnn&out_trade_no=Native20190722145741431794b8d1&pay_type=800201&txamt=20&txcurrcd=HKD&txdtm=2019-07-22 14:57:42&udid=AA
```

### 回應範例

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

## 請求參數

| 欄位 | 必填 | 類型 | 描述 |
| ----- | -------- | ---- | ----------- |
| [公共支付請求參數](/docs/api-reference/request-format#公共支付請求參數) | — | — | 通用欄位，例如 `mchid`、`txamt` 等。 |
| `pay_tag` | No | String(16) | 錢包類型標示。預設為 `ALIPAYHK`，中國大陸使用者為 `ALIPAYCN` |
| `expired_time` | No | String(3) | QR 碼有效時間（單位為分鐘，範圍 5–120），預設為 30 分鐘。僅適用於 Alipay/WeChat |
| `limit_pay` | No | String | 使用 `no_credit` 可禁用信用卡付款。僅適用於中國大陸錢包。 |

## 回應參數

| 欄位 | 類型 | 描述 |
| ----- | ---- | ----------- |
| `qrcode` | String(512) | 生成的 QR 碼網址 |
| [通用回應格式](/docs/api-reference/response-format#%E6%AC%84%E4%BD%8D%E8%AA%AA%E6%98%8E) | — | 通用回應欄位，例如 `respcd`、`syssn` 等。 |