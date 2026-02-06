---
id: api-usage-guidelines
title: API Usage Guidelines
sidebar_label: API Usage
---

## Overview

To ensure fair usage and maintain optimal performance of our platform, we have implemented an API rate limit policy. This policy outlines the limits on API requests and the appropriate handling of any rate limit violations.

## Rate Limit

- **Limit**: Each merchant is allowed a maximum of **100 requests per second (RPS)** and **400 requests per minute**.
- **Response on Exceeding Limit**: If a merchant exceeds this limit, the API will respond with a **HTTP 429 Too Many Requests** status code.

## Usage Guidelines

To help you make the most of our API and to avoid hitting the rate limit, please consider the following best practices:

1. **Batch Requests**: Where applicable, use batch processing to minimize the number of individual requests.
2. **Efficient Data Retrieval**: Optimize your queries to reduce the frequency of requests. Utilize filtering and pagination to retrieve only the necessary data.
3. **Caching Responses**: Implement caching strategies to store responses temporarily and avoid repeated requests for the same data.
4. **Monitor Usage**: Keep track of your API usage to ensure you stay within the set limits. Implement logging to analyze your request patterns.

## Error Handling

In the event of a rate limit violation, the following practices should be followed:

1. **Handle HTTP 429 Response**:
   - Implement logic in your application to gracefully handle the HTTP 429 response by:
     - Pausing further requests for a specified duration (e.g., retry after a few seconds).
     - Logging the error for monitoring and alerting purposes.

2. **Exponential Backoff**:
   - When retrying after receiving a 429 response, implement an exponential backoff strategy to gradually increase the wait time between retries.

## Spike Traffic Management

If you anticipate a spike in traffic, such as during a promotional event, please contact our technical support team:

- **Technical Support Email**: [technical.support@qfpay.com](mailto:technical.support@qfpay.com)

By reaching out in advance, we can assist you in managing your traffic and ensure uninterrupted service during peak times.