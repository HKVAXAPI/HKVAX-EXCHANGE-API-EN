# Access to API

## Manual

To use the API, you need to complete the API Key creation and permission configuration first. Click here to create an API Key, and then develop and trade according to this document.

Each user can create 10 groups of API Keys, and each API Key can set three permissions for market quotation, trade, and user information query. Please remember the following information after successful creation:

- `Access Key` API access key
- `Secret Key` The key used for signature authentication and encryption (only visible at the time of creation completed)

## Access URLs

Trade: [https://api.trade.hkvax.com/v1](https://api.trade.hkvax.com/v1)

Sim-trade: [https://api.simtrade.hkvax.com/v1 ](https://api.simtrade.hkvax.com/v1 )

## Request Format

All requests are based on the https protocol.

### Request Parameter

A legitimate request consists of a header part and a requestbody part.

**Header**


| **Parameters** | Type   | Required | Description                                                  |
| -------------- | ------ | -------- | ------------------------------------------------------------ |
| APIKEY         | string | true     | Public key                                                   |
| SIGNATURE      | string | true     | **Request data signature string**                            |
| CEXTOKEN       | string | false    | Required to get the apikey interface, and the token needs to be logged in to get |
| DEVICEID       | string | true     | Random string (can be fixed to a value)                      |
| DEVICESOURCE   | string | true     | fixed value: web                                             |
| Lang           | string | true     | zh-CN or en-US                                               |
| Client         | string | true     | Fixed value, assigned by cex                                 |

**Requestbody**

| **Parameters** | Type     | Required | Description                                                  |
| -------------- | -------- | -------- | ------------------------------------------------------------ |
| lang           | string   | true     | Consistent with header                                       |
| data           | any type | false    | String/int/array/jsonObject/jsonArray<br>and other types are available, the actual type is subject to the interface description |

**The example of data as string is as follows**

```json
{
    "lang" : "zh-CN",
    "data" : "123"
}
```

**The example of data as json is as follows**

```json
{
    "lang" : "zh-CN",
    "data" : {
        "orderNo" : 123,
        "currency" : "USD"
    }
}
```

### Response Parameters

All interface returns are in JSON format. At the top level of JSON, there are two fields representing request status and attributes: "code", "msg", and the actual  return content is in the "data" field.

**ResponseBody**

| **Parameters** | Type     | Description                                                  |
| -------------- | -------- | ------------------------------------------------------------ |
| code           | string   | 000000 means success, others means failure                   |
| msg            | string   | returned messages                                            |
| data           | any type | string/int/array/jsonObject/jsonArray<br>and other types are available, the actual type is subject to the interface description |

## Authentication

To protect API communication from unauthorised change, all API calls are required to be signed. Each API Key requires permissions to access the corresponding interface, so please make sure that your API Key has the corresponding permissions before using the interface.

### Signing Steps

1. Sort the API Key, CEXTOKEN, and data parameters according to the order of the request parameters given in the document;

2. Perform an RSA operation on the above string to get the signature.


**Signature Example**

The following example uses SHA256withRSA signature:

   Public key：

```java
   MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCScP9BF63DY5z/WgCqAXpUSbWnY7wYLGT+5pn2NWjRuKmKF4aMxbSKmiGny3Sno+2yvqFqKWv/lz56yaylPKucN2WGdH0HKYnxhXWbaWLK23gBVfHu6YZvh8sBzJ7lOCXvLwMCRGNfDqFNj2TPmA0wMnop+0FXQOVe/q1USbFXPwIDAQAB
```

   Private key：

```text
   MIICeAIBADANBgkqhkiG9w0BAQEFAASCAmIwggJeAgEAAoGBAJJw/0EXrcNjnP9aAKoBelRJtadjvBgsZP7mmfY1aNG4qYoXhozFtIqaIafLdKej7bK+oWopa/+XPnrJrKU8q5w3ZYZ0fQcpifGFdZtpYsrbeAFV8e7phm+HywHMnuU4Je8vAwJEY18OoU2PZM+YDTAyein7QVdA5V7+rVRJsVc/AgMBAAECgYB33wMytz1HqWzEIVpVzyvhfwyxXpSDfSOW/BCfV4zbzzsIjMVYyiVFJ3HRNlvhNfDG1gCvNATxjU5ZmGg4Qfd+hNqB2X4BeeK3pUNbesz5SNXPGhu/0oIWVN5oKZJjwzhB8aaSSxNyU73g36NyvkfSJMOMvQMOqtlnAPAwL3biIQJBAM1NtEwtn2IcogYsF315qgqfefLPtFr/szbczVSkOng/+uVNmcxDrfA3bwu0+IqV5Yw/+57rDjkuBuU/yyO6aekCQQC2mlA3GqkNIFIo/C+W7aRhpJS2DF+J6EznZf82hgD0C/3D/npC0k/Diy89gglNpaKuSx/IxJjPLUpJl0uNe9bnAkEAnT7q3X4EGY18u+WBiGVrS/+h08wqg5hdl6O+0RmIfxnh/UdWiRE9ZEPRFdJimyL8UlOfUbUPi9QpC+W0nYTmIQJBAIy218/O/Kz/1jB9PhMZqE4SbQLpAAqe9/xtrkEO/NcUEochqIer2AnBTTMh7Rdn57hWbfTiAzvME+4n5/Hsl8sCQQC4/K4Q9r2frQcMRUNEu42tMkA1w1XgC8fRC7/9K4Y4+XLJIx7YN9hoKkyvxbmM35yRiwYUoUJrRuLNaQ32trRu
```

   requestbody：

```json
{"data":{"currency":"USD"},"lang":"zh-CN"}
```

  Signature information：

```text
KsxKzaDdcoufq8Nz3xHV0rz1t4ewswZNKrY3Utv3ltR/xcMztm0LUDcGo9B1YTz8lilzLIlknj/VcFsYzBEtonhA0nxcuRVkfcvkY5gHCcaGBvrSzjkDNSmr1tsPcoaDShy5MxEJeiLNn8qi6pFxtYWT8LHMvM+N1/jUSRVVpWI=
```

## Pagination

After paging, the data parameter consists of a paging object and a business array object.

**Paging Object**

| Parameter | Meaning                 | Type | Description              |
| --------- | ----------------------- | ---- | ------------------------ |
| pageSize  | Total pages             | int  | &nbsp;                   |
| currPage  | Actual page             | int  | from page 1              |
| totalRows | Total number of rows    | int  | &nbsp;                   |
| pageRows  | Number of rows per page | int  | Page size (maximum 1000) |

**Successful example one**

```json
{
    "code" : "000000",
    "msg"  : "success",
    "data" : 15
}
```

**Successful example two (paging example)**

```json
{
    "code" : "000000",
    "msg"  : "success",
    "data" : {
        "pagination" : {
            "pageSize" : 22,
            "currPage" : 1,
            "totalRows" : 212,
            "pageRows" : 10
        },
        "list" : [
            {
                "orderNo" : 123,
                "currency" : "USD",
                "amount" : 10,
                "userNo" : 11
            }, {
                "orderNo" : 222,
                "currency" : "USD",
                "amount" : 3,
                "userNo" : 12
            }
        ]
    }
}
```

**Failure Example**

```json
{
     "code": "999006",
     "msg": "Order number does not exist",
     "data": null
}
```

## Rules

### Timestamp

Unless otherwise specified, all timestamps are Unix timestamps.

### Precision

It is recommended that you convert the number to a string when making a request to avoid truncation and precision errors.

| Pair       | Order Quantity Precision | Order Price Precision | Order Amount Precision |
| ---------- | ------------------------ | --------------------- | ---------------------- |
| BTC/USD    | 4                        | 2                     | 2                      |
| ETH/USD    | 4                        | 2                     | 2                      |
| BCHABC/USD | 4                        | 2                     | 2                      |
| LTC/USD    | 4                        | 2                     | 2                      |
| XRP/USD    | 4                        | 6                     | 6                      |

## Response parameter error code

| Error code | Meaning                                     | Description                                                  |
| ---------- | ------------------------------------------- | ------------------------------------------------------------ |
| 100006     | Bad request                                 | Invalid request format: invalid request parameter or header parameter |
| 100037     | Orders not from users                       | The order number the user is requesting to cancel is not his own order number |
| 100038     | Order does not exist                        |                                                              |
| 100039     | Order has been cancelled                    |                                                              |
| 100040     | Order has been completed                    |                                                              |
| 100041     | Order cancellation failed                   |                                                              |
| 100043     | Unsupported trading pair                    | The trading pair is offline or the trading pair does not exist |
| 100044     | Order action is not supported               | An action other than buying and selling is passed            |
| 100046     | Trading pair is not allowed to place orders | This trading pair is not allowed to place orders             |
| 100047     | Trading prohibited                          | User group permission restrictions                           |
| 100048     | Invalid order price or quantity             | Excessive order quantity or price                            |
| 100049     | Invalid order price or quantity             | Order quantity or price is less than the minimum value       |
| 100050     | Invalid order price or quantity             | Order quantity or price is greater than the maximum value    |
| 100051     | Price exceeds limit                         |                                                              |
| 100052     | Order type is not supported                 | An unknown order type was provided                           |
| 100054     | Insufficient balance                        |                                                              |
| 100055     | Order placing failed                        |                                                              |
| 100112     | The account does not exist                  | Funds account not created                                    |
| 100212     | Security authentication failed              | Failure of a safety check for a parameter or result          |
| 100152     | Frequent orders                             | Exceeding order flow limits                                  |
| 500100     | Signature error                             |                                                              |
| 500202     | ApiKey has no access permission             |                                                              |

## Interface Groups

| Interface grouping | Containing interface                                         |
| ------------------ | ------------------------------------------------------------ |
| Quotation          | 24hr Ticker  (POST /quotation/quotation/query/symbolQuotationGroupBy)<br />Market Depth (POST /quotation/quotation/query/depthData) <br />Candles (POST /quotation/kline/get/quotationHistory)<br />The Latest Trade (POST /quotation/quotation/query/latestTradeQuotation)<br />Order Book (POST /quotation/order/query/orderBook)<br />Latest Trades (POST /quotation/order/query/dealList)<br />Best Bid& Best Ask (POST /quotation/order/query/marketBestPrice) |
| Trade              | Place an order (POST /order/cex/user/order/create)<br />Cancel an order (POST /order/cex/user/order/cancel) |
| User Data Query    | Account Balance (POST /user/cex/user/asset/query/currencyAsset)<br/>Order List (POST /user/cex/user/order/query/orderList)<br/ >Order Details (POST /user/cex/user/mm/query/orderDetail)<br/>Trades (POST /user/cex/user/order/query/dealOrderList)<br/>Order Trades (POST /user/cex/user/order/query/dealDetail) |
