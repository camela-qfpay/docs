---
id: "fps-ecert-setup"
title: "FPS e-Cert Certificate Application and Setup Guide"
description: "Guide for applying and configuring SSL certificates for FPS App-to-App integration, particularly for HSBC merchants."
sidebar_label: "FPS e-Cert Setup Guide"
---

:::info This guide is required if you are an FPS merchant using App-to-App payment, especially for HSBC merchants. :::

---

## When Do You Need an e-Cert?

If you are integrating FPS App-to-App with a merchant-specific Universal Link, an e-Cert (Server certificate) is required to secure the domain.

This applies to banks like HSBC, which operate in direct integration mode and require domain validation and organisation name matching.

---

## Important Notes for FPS App-to-App Certificate

:::warning The **merchant organisation name** (subject field in the X.509 certificate) **must exactly match** the payee name registered in the FPS Addressing Service.

This requirement comes from FPS Technical Specification 6.9.2.

> Payment App MUST validate the organization name inside the subject field of the merchant’s certificate (in x509 format) matches the payee name returned from the FPS addressing service.

The **certificate domain name** is assigned and configured by QFPay based on the integration context. Merchants do not select this domain themselves.

Each **distinct domain** requires a **separate e-Cert application** (e.g. `fps.payment.example-shop.com`), which incurs additional application cost and processing time.

Merchants must also add the following **CNAME record** in their DNS configuration:

```
Host: fps.merchant.com
Type: CNAME
Value: hk.qfapi.com
```

:::

---

## FPS e-Cert Application Overview

| Step | Description                                                                                         |
| ---- | --------------------------------------------------------------------------------------------------- |
| 1    | Fill the application form CPos 798F                                                                 |
| 2    | Submit the form in person at any Hongkong Post office                                               |
| 3    | Present authorised ID and pay the subscription fee                                                  |
| 4    | Receive PIN envelope used to submit the CSR                                                         |
| 5    | Generate and submit CSR via designated e-Cert portal                                                |
| 6    | Wait ~10 working days for approval and domain/email validation                                      |
| 7    | Upon approval, submit CSR to issue certificate                                                      |
| 8    | Download and install the e-Cert to your HTTPS server                                                |
| 9    | Send **certificate and private key files** to QFPay technical support for FPS pay URL configuration |

---

## CSR Generation Requirements

Before submitting the certificate application, you must generate a **Certificate Signing Request (CSR)**. An example of the OpenSSL command:

```
openssl req -new -SHA256 -newkey rsa:2048 -nodes \
-keyout <key_name>.key \
-out <cert_name>.csr \
-subj "/C=HK/ST=HongKong/L=HongKong/O=<Your_Organisation_Name>/OU=/CN=<your_domain>"
```

#### Parameter Breakdown:

| Field              | Description                                  |
| ------------------ | -------------------------------------------- |
| `-newkey rsa:2048` | Generate a new RSA key pair (2048-bit)       |
| `-nodes`           | Skip password encryption for the private key |
| `-keyout`          | File path to save the private key            |
| `-out`             | File path to save the generated CSR          |
| `-subj`            | Subject fields included in the certificate   |

> **Notes**:
>
> - The `O=` (organisation name) **must exactly match** the FPS payee name registered in the FPS Addressing Service.
> - The `CN=` (common name) is the domain name to be configured by QFPay on a case-by-case basis.
> - Leave `OU=` empty if there is no department specified.

---

## Documents Required

- Completed CPos Form 798F
- Business Registration (BR) Copy
- Company Incorporation (CI) Copy
- Domain Ownership Proof (e.g. invoice, DNS panel screenshot, domain email confirmation)

---

## Post-Issuance Responsibilities

:::info Hongkong Post will send **expiration reminders** to the merchant’s registered email **30 days and 14 days before the certificate expires**.\
Merchants are responsible for timely renewal and communication with QFPay. :::

Once the certificate is approved and issued:

- Merchant must **send the certificate (.cer/.crt) and private key (.key)** to QFPay Technical Support.
- QFPay will complete backend setup for the FPS pay URL endpoint.

---

## FPS Specification Reference

**Section 6.9.2 – Certificate Validation Logic**\
The Payment App (e.g. HSBC app) will validate that the **Organisation Name (O)** in the X.509 certificate **matches the FPS payee name**.

This comparison is:

- **Case-insensitive**
- **Whitespace-insensitive**

Failure to match will result in payment rejection.

---

## Resources

- [e-Cert Form 798F (Apply for e-Cert Server)](https://www.ecert.gov.hk/product/ecert/apply/certapply.html#t4)
- [e-Cert Application Flowchart (PDF)](https://www.ecert.gov.hk/product/ecert/apply/img/e-Cert_\(S\)_Flow.pdf)

---