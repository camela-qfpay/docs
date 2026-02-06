---
title: "FAQs"
---

**Q1. In the provided test account credentials, which field identifies the partner and which identifies the merchant?**

> A1. If you are an agent providing payment services to merchants, `X-QF-APPCODE` and `ClientKey` identify the **partner**, while `MCHID` identifies the **merchant**.\
> If you are a direct merchant, then `X-QF-APPCODE` and `ClientKey` identify the **merchant**, and `MCHID` is typically not provided.

**Q2. Can I use the given test or production account in a different country?**

> A2. No. Both test and production accounts are **country-specific** and cannot be used outside their designated region.

**Q3. How do I test a transaction?**

> A3. You can use a real wallet (e.g., Alipay or WeChat Pay) to test transactions in the test environment, which simulates the production setup. Contact our technical support if you need assistance.

**`Q4. I received response code 1143 or 1145. What should I do?`**

> A4. Continue polling the transaction status using the enquiry API.\
> If your business process requires binary logic (success/failure), you may treat the transaction as **failed**, and later issue a refund if a successful asynchronous notification is received.

**Q5. The payment method I want to integrate is not listed in the documentation. What should I do?**

> A5. Use the [Public Payment Parameters](/docs/api-reference/request-format) and refer to the **Notes** section at the end of the [Payment Types](/docs/api-reference/paycode#payment-codes) page for any special instructions.

**Q6. Can I refund a transaction made a few days ago?**

> A6. Refunds are only allowed if the **same-day unsettled transaction amount** is **equal to or greater** than the refund amount.

**Q7. Will funds be transferred to my bank account when testing in the sandbox environment?**

> A7. No. Transactions in the sandbox environment are **not settled**. Please remember to **immediately refund** test transactions after they are created.

**Q8. Can I use my overseas Alipay wallet to make a payment?**

> A8. No. Only Alipay wallets that are **real-name verified Mainland China accounts** can be used for cross-border transactions.