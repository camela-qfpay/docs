---
id: "status-codes"
title: "Transaction Status Codes"
sidebar_label: "Transaction Status Codes"
---

The table below lists all standard `respcd` values returned by QFPay API responses. These codes help you determine the result and next action for each transaction.

| Return Code | Description                                                                                                                                              |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0000        | Transaction successful                                                                                                                                   |
| 1100        | System under maintenance (1100)                                                                                                                          |
| 1101        | Reversal error (1101)                                                                                                                                    |
| 1102        | Duplicate request (1102)                                                                                                                                 |
| 1103        | Request format error (1103)                                                                                                                              |
| 1104        | Request parameter error (1104)                                                                                                                           |
| 1105        | Device not activated (1105)                                                                                                                              |
| 1106        | Invalid device (1106)                                                                                                                                    |
| 1107        | Device not allowed (1107)                                                                                                                                |
| 1108        | Signature error (1108)                                                                                                                                   |
| 1125        | Transaction has been refunded already (1125)                                                                                                             |
| 1136        | The transaction does not exist or is not operational (1136)                                                                                              |
| 1142        | Order already closed (1142)                                                                                                                              |
| 1143        | The order has not been paid for, the password is currently being entered (1143)                                                                          |
| 1145        | Please wait while processing (1145)                                                                                                                      |
| 1147        | WeChat Pay transaction error (1147)                                                                                                                      |
| 1150        | Your billing method is T0 and does not support canceling transactions. (1150)                                                                            |
| 1155        | Refund request denied (1155)                                                                                                                             |
| 1181        | Order expired (1181)                                                                                                                                     |
| 1201        | Insufficient balance, please use a different payment method (1201)                                                                                       |
| 1202        | Incorrect or expired payment code, please show the correct payment code or refresh the payment code and retry (1202)                                     |
| 1203        | Merchant account error, confirm that the payment account is configured correctly (1203)                                                                  |
| 1204        | Bank error, confirm that the payment wallet is functional (1204)                                                                                         |
| 1205        | The transaction failed. Please try again later (1205)                                                                                                    |
| 1212        | Please use the UnionPay overseas payment code (1212)                                                                                                     |
| 1241        | The store does not exist or the status is incorrect. Do not conduct payments (1241)                                                                      |
| 1242        | The store has not been configured correctly, unable to conduct payments (1242)                                                                           |
| 1243        | The store has been disabled. Do not conduct payments, contact the owner to confirm (1243)                                                                |
| 1250        | The transaction is forbidden. For more information please contact QFPay Customer Service Team (1250)                                                     |
| 1251        | The store has not been configured correctly, we are currently working to fix this problem (1251)                                                         |
| 1252        | System error when making the order request (1252)                                                                                                        |
| 1254        | A problem occurred. We are currently resolving the issue (1254)                                                                                          |
| 1260        | The order has already been paid for, please confirm the transaction result before conducting more transactions (1260)                                    |
| 1261        | The order has not been paid for, please confirm the transaction result before conducting more transactions (1261)                                        |
| 1262        | The order has been refunded, please confirm the order status before conducting more transactions (1262)                                                  |
| 1263        | The order has been cancelled, please confirm the order status before conducting more transactions (1263)                                                 |
| 1264        | The order has been closed, please confirm the order status before conducting more transactions (1264)                                                    |
| 1265        | The transaction cannot be refunded. Refunds for transactions between 11:30pm to 0:30am and special promotions cannot be processed. (1265)                |
| 1266        | The transaction amount is wrong, please confirm the order status (1266)                                                                                  |
| 1267        | The order information does not match, please confirm the order status (1267)                                                                             |
| 1268        | The order does not exist, please confirm the order status (1268)                                                                                         |
| 1269        | Today's unsettled transaction amount is insufficient. Refunds cannot be processed. Please confirm that the balance is sufficient (1269)                  |
| 1270        | This currency does not support partial refunds (1270)                                                                                                    |
| 1271        | The selected transaction does not support partial refunds (1271)                                                                                         |
| 1272        | The refund amount is greater than the maximum amount that can be refunded for the original transaction (1272)                                            |
| 1294        | The transaction may be non-compliant and has been prohibited by the bank (1294)                                                                          |
| 1295        | The connection is slow, waiting for a network response (1295)                                                                                            |
| 1296        | The connection is slow, waiting for a network response. Please try again later or use other payment methods (1296)                                       |
| 1297        | The banking system is busy. Please try again later or use other payment methods (1297)                                                                   |
| 1298        | The connection is slow, waiting for a network response. In case you have already paid, do not repeat the payment. Please confirm the result later (1298) |
| 2005        | The customer payment code is incorrect or has expired, please refresh and restart the transaction process (2005)                                         |
| 2011        | Transaction serial number repeats (2011)                                                                                                                 |