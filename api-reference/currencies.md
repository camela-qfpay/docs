---
id: currencies
title: Currencies
sidebar_label: Currencies
---

# Supported Currencies

This section lists the currency codes accepted by the QFPay API. The values passed in the `txcurrcd` parameter should match one of the supported currency codes listed below.

All codes follow the standard ISO 4217 format (3-letter uppercase).

Currency Code | Description
--------- | -------
AED | Arab Emirates Dirham
CNY | Chinese Yuan
EUR | Euro
HKD | Hong Kong Dollar
IDR | Indonesian Rupiah
JPY | Japanese Yen
MMK | Myanmar Kyat
MYR | Malaysian Ringgit
SGD | Singapore Dollar
THB | Thai Baht
USD | United States Dollar
CAD | Canadian Dollar
AUD | Australian Dollar

:::note
Some payment methods may only support HKD. Always verify with your integration manager before initiating non-HKD transactions.
:::
