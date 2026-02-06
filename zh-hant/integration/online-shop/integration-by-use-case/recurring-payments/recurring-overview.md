---
id: recurring-overview
title: Recurring Payment Overview
description: Overview of QFPay's recurring payment system, including use cases, system architecture, and key components involved in subscription billing.
sidebar_label: Overview
---


# Recurring Payments Overview

Recurring payments (also known as subscription payments) allow merchants to automatically charge customers on a regular schedule using a previously saved payment token. This flow is ideal for memberships, SaaS billing, utility payments, and any services billed over time.

---

## What is a Subscription?

A subscription is an agreement to charge a customer at fixed intervals (e.g. every month) for a fixed or variable amount. In QFPay, this is represented by a `subscription` object that references:

* A `customer_id`: the end user being charged
* A `token_id`: the saved card or payment method
* A `product_id`: defining the billing frequency and amount

---

## Key Concepts & Objects

| Object            | Description                                                                                                                            |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `customer_id`     | A user profile saved for recurring billing (e.g. email, phone)                                                                         |
| `token_id`        | A stored payment credential obtained from the cardholder via [Payment Element](/docs/online-shop/checkout-integration/payment-element) |
| `product_id`      | Defines billing amount, interval (monthly, yearly), and currency                                                                       |
| `subscription_id` | The subscription record combining the above, controlling the billing lifecycle                                                         |

---

## End-to-End Subscription Lifecycle

### Step 1: Create a Customer

Use `/customer/v1/create` to register customer details.

### Step 2: Tokenise the Payment Method

Use the [Payment Element](/docs/online-shop/checkout-integration/payment-element) to securely collect card details and get a `token_id`.

### Step 3: Create a Product

Use `/product/v1/create` to define billing frequency and amount (e.g. HKD 100 every month).

### Step 4: Create the Subscription

Use `/subscription/v1/create` with `customer_id`, `product_id`, and `token_id`.

### Step 5: Automatic or Manual Billing

QFPay will auto-charge the customer on the schedule. Failed payments can be retried manually via `/subscription/v1/charge`.

---

## Subscription States

| State        | Description                                       |
| ------------ | ------------------------------------------------- |
| `ACTIVE`     | Subscription is live and billing as scheduled     |
| `INCOMPLETE` | Initial setup failed or token is invalid          |
| `PAST_DUE`   | One or more billing attempts failed               |
| `CANCELLED`  | Subscription was cancelled by merchant or expired |
| `COMPLETED`  | Subscription ended after completing all cycles    |

To receive updates, implement [webhook notifications](/docs/online-shop/integration-by-use-case/recurring-payments/recurring-webhook).

---

## Example Use Cases

* **Gym Memberships**: Charge HKD 300 monthly
* **SaaS Subscriptions**: Charge HKD 98 monthly or HKD 999 yearly
* **Equipment Rentals**: Time-limited recurring payments with retries

---

## FAQ

### Can I update the payment token?

Yes. You can update the `token_id` on an active subscription via `/subscription/v1/update`.

### Can I pause a subscription?

Pausing is not currently supported. Cancel the subscription and create a new one when resuming.

### Can I retry failed charges?

Yes. Use `/subscription/v1/charge` with the failed `subscription_order_id`.

---

## Next Steps

* [API Reference](/docs/online-shop/integration-by-use-case/recurring-payments/recurring-api-reference)
* [Webhook Guide](/docs/online-shop/integration-by-use-case/recurring-payments/recurring-webhook)
