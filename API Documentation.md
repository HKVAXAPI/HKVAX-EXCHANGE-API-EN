# Introduction

## Basic Introduction

Welcome to the HKVAX developer documentation. This document outlines the API introduction, access method and interface description of HKVAX.

API is divided into three categories: quotation, trade, and user information query. You can create API Keys with different permissions according to your needs, and use the APIs for automated transactions.

The quotation APIs provides market quotation data, and all quotation interfaces are public. The trading and user query APIs requires identity verification and provides functions such as placing and canceling orders, and querying order and account information.

## Trading Rules

### Matching Engine

HKVAX matches according to the rule of "price priority-user priority-time priority". The rules are the same for all order types.

Price priority means that a buy order with a higher price has priority over a buy order with a lower price, and a sell order with a lower price has priority over a sell order with a higher price.

User priority means that if the price of some pending orders is the same, users’ orders will enjoy a higher priority than orders placed by HKVAX’s company accounts or employees’ accounts.

Time priority means that for pending orders with the same price and the same user type, the order sequence is determined by the time of placing orders, the earlier placed orders are prioritized than the later placed orders.

### Order Lifecycle

After the order enters the matching engine, it is "NOT FILLED" `MISMATCH` status;

Then the order starts to be filled, and the partially filled order will become ``PARTIAL `` status;

Until an order is all filled, it will become "deal" `DEAL` status.

If an unfilled order is successfully canceled, then it becomes `CANCEL_MISMATCH` status;

If a partially filled order is successfully canceled, then it becomes `CANCEL_PARTIAL` status;

In addition, if the system is processing the order cancellation and there is no result response, then the order is "being cancelled" `FIRSTTRIAL ` status.

## Self-Trade Prevention 

Self-trading is not allowed on HKVAX. Two orders from the same user will not fill one another. When a user's order is matching, if the counterparty is himself, the system will automatically cancel the later order  to prevent self-trading.

## Fees

To enhance market liquidity and participation, HKVAX adopts a maker-taker fee schedule, in which the maker fee is lower than the taker fee. Maker: 0.2%, Maker: 0.1%

## Sim-trade

Sim-trade can be used to test API connectivity and trading. The function of the simulated environment is independent of the formal environment. You need to register, log in, and create an API key separately.

In addition, once you complete the registration, we will provide sufficient simulated funds for you to test.

**website**: [https://simtrade.hkvax.com](https://simtrade.hkvax.com/)



# Access to API

## Manual

To use the API, you need to complete the API Key creation and permission configuration first. Click here to create an API Key, and then develop and trade according to this document.

Each user can create 10 groups of API Keys, and each API Key can set three permissions for market quotation, trade, and user information query. Please remember the following information after successful creation:

- `Access Key` API access key
- `Secret Key` The key used for signature authentication and encryption (only visible at the time of creation completed)

## Access URLs

**Trade**: [https://api.trade.hkvax.com/v1](https://api.trade.hkvax.com/v1)

**Sim-trade**: [https://api.simtrade.hkvax.com/v1 ](https://api.simtrade.hkvax.com/v1 )

## Request Format

All requests are based on the https protocol.

### Request Parameter

A legitimate request consists of a header part and a requestbody part.

**Header**


| **Parameters** | Type  | Required | Description                        |
| ------------ | ------ | -------- | -------------------------------------- |
| APIKEY       | string | true     | Public key   |
| SIGNATURE    | string | true     | **Request data signature string**                     |
| CEXTOKEN     | string | false    | Required to get the apikey interface, and the token needs to be logged in to get |
| DEVICEID     | string | true     | Random string (can be fixed to a value)                 |
| DEVICESOURCE | string | true     | fixed value: web                            |
| Lang         | string | true     | zh-CN or en-US                           |
| Client       | string | true     | Fixed value, client customize name or id                     |

**Requestbody**

| **Parameters** | Type | Required | Description                                              |
| ------ | -------- | -------- | ------------------------------------------------------------ |
| lang   | string   | true     | Consistent with header                                           |
| data   | any type | false    | String/int/array/jsonObject/jsonArray<br>and other types are available, the actual type is subject to the interface description |

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

| **Parameters** | Type | Description                                                  |
| ------ | -------- | ------------------------------------------------------------ |
| code   | string   | 000000 means success, others means failure                                |
| msg    | string   | returned messages                                                    |
| data   | any type | string/int/array/jsonObject/jsonArray<br>and other types are available, the actual type is subject to the interface description |

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

Parameter | Meaning               | Type | Description              
---|---|---|---
pageSize | Total pages | int | &nbsp;
currPage | Actual page | int | from page 1
totalRows | Total number of rows | int | &nbsp;
pageRows | Number of rows per page | int | Page size (maximum 1000)

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

| Pair | Order Quantity Precision | Order Price Precision | Order Amount Precision |
| ---------- | ------------ | ------------ | ------------ |
| BTC/USD    | 4 | 2 | 2 |
| ETH/USD    | 4 | 2 | 2 |
| BCHABC/USD | 4 | 2 | 2 |
| LTC/USD    | 4 | 2 | 2 |
| XRP/USD    | 4 | 6 | 6 |

## Response parameter error code

| Error code | Meaning | Description |
| ------ | ------------------- | ---------------------------------------- |
| 100006 | Bad request | Invalid request format: invalid request parameter or header parameter |
| 100037 | Orders not from users | The order number the user is requesting to cancel is not his own order number |
| 100038 | Order does not exist | |
| 100039 | Order has been cancelled | |
| 100040 | Order has been completed | |
| 100041 | Order cancellation failed | |
| 100043 | Unsupported trading pair | The trading pair is offline or the trading pair does not exist |
| 100044 | Order action is not supported | An action other than buying and selling is passed |
| 100046 | Trading pair is not allowed to place orders | This trading pair is not allowed to place orders |
| 100047 | Trading prohibited | User group permission restrictions                           |
| 100048 | Invalid order price or quantity | Excessive order quantity or price                            |
| 100049 | Invalid order price or quantity | Order quantity or price is less than the minimum value |
| 100050 | Invalid order price or quantity | Order quantity or price is greater than the maximum value |
| 100051 | Price exceeds limit | |
| 100052 | Order type is not supported | An unknown order type was provided                           |
| 100054 | Insufficient balance |                                                              |
| 100055 | Order placing failed | |
| 100112 | The account does not exist | Funds account not created                                    |
| 100212 | Security authentication failed | Failure of a safety check for a parameter or result          |
| 100152 | Frequent orders | Exceeding order flow limits                                  |
| 500100 | Signature error | |
| 500202 | ApiKey has no access permission | |

## Interface Groups

| Interface grouping | Containing interface |
| ------------ | --------------------------------------------------------- |
| Quotation | 24hr Ticker  (POST /quotation/quotation/query/symbolQuotationGroupBy)<br />Market Depth (POST /quotation/quotation/query/depthData) <br />Candles (POST /quotation/kline/get/quotationHistory)<br />The Latest Trade (POST /quotation/quotation/query/latestTradeQuotation)<br />Order Book (POST /quotation/order/query/orderBook)<br />Latest Trades (POST /quotation/order/query/dealList)<br />Best Bid& Best Ask (POST /quotation/order/query/marketBestPrice) |
| Trade | Place an order (POST /order/cex/user/order/create)<br />Cancel an order (POST /order/cex/user/order/cancel) |
| User Data Query | Account Balance (POST /user/cex/user/asset/query/currencyAsset)<br/>Order List (POST /user/cex/user/order/query/orderList)<br/ >Order Details (POST /user/cex/user/mm/query/orderDetail)<br/>Trades (POST /user/cex/user/order/query/dealOrderList)<br/>Order Trades (POST /user/cex/user/order/query/dealDetail) |



# Quotation

## 24hr Ticker  

Get the 24hr volume and price of all trading pairs.

**HTTP Request**

> POST /quotation/quotation/query/symbolQuotationGroupBy

**Request Parameters**

None

**Request Example**

```json
{
    "lang" : "zh-CN"
}
```

**curl request example**

> curl -X POST "[https://api.trade.hkvax.com/v1/quotation/quotation/query/symbolQuotationGroupBy](https://api.trade.hkvax.com/v1/quotation/quotation/query/symbolQuotationGroupBy)" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\"}"

**Response** 

| **Parameter** | Meaning                 | Type | Required | **Description**           |
| ------------------- | ------------- | --------- | -------- | -------------- |
| group               | Group name         | string    | true     | &nbsp;         |
| list                | Market information    | jsonArray | true     | &nbsp;         |
| list.symbol         | Pair        | string    | true     | &nbsp;         |
| list.max            | Highest price | string    | true     | &nbsp;         |
| list.min            | Lowest price | string    | true     | &nbsp;         |
| list.lastPrice      | Closing price        | string    | true     | &nbsp;         |
| list.quantity       | 24H volume     | string    | true     | &nbsp;         |
| list.amount         | 24H Amount   | string    | true     | &nbsp;         |
| list.lastUsdPrice   | Closing USD price | string    | true     | &nbsp;         |
| list.waveChange     | Change ratio | string    | true     | &nbsp;         |
| list.wavePrice      | Change value       | string    | true     | &nbsp;         |
| list.wavePercent    | Change percentage   | string    | true     | &nbsp;         |
| list.direction      | Change direction        | int       | true     | 1rise、0unchanged、-1fall |
| list.priceDirection | Current price change  | int       | true     |1rise、0unchanged、-1fall |
| list.maxUsdPrice    | Maximum USD price | string    | true     | &nbsp;         |
| list.minUsdPrice    | Minimum USD price | string    | true     | &nbsp;         |
| list.steps          | Optional decimal places | jsonArray | true     | &nbsp;         |
| steps.code          | Decimal code       | string    | true     | &nbsp;         |
| steps.value         | Decimal Value       | string    | true     | &nbsp;         |
| steps.decimalValue  | Decimal places      | int       | true     | &nbsp;         |

**Response Example**

```json
{
	"code": "000000",
	"msg": "success",
	"data": [{
		"group": "USD",
		"list": [{
		    "amount" : "0",
			"areaUsdPrice": "1.00000000",
			"direction": 0,
			"lastPrice": "1.5600",
			"lastUsdPrice": "1.56000000",
			"max": "1.5600",
			"maxUsdPrice": "1.56000000",
			"min": "1.5600",
			"minUsdPrice": "1.56000000",
			"quantity": "0",
			"steps": [{
				"code": "00001",
				"decimalValue": 4,
				"value": "0.0001"
			}, {
				"code": "001",
				"decimalValue": 2,
				"value": "0.01"
			}, {
				"code": "1",
				"decimalValue": 0,
				"value": "1.0"
			}],
			"symbol": "UPT/USD",
			"wavePercent": "0.00",
			"wavePrice": "0.0000"
		}]
	}]
}
```
## Market Depth

Get the depth data of the specified pair.

**HTTP Request**

> POST /quotation/quotation/query/depthData

**Request Parameters**

| Parameter | Meaning | Type | Required | Description |
| ------ | -------- | ------ | -------- | ------ |
| symbol | Pair   | string | true     | &nbsp; |

**Request Example**

```json
{
    "lang" : "zh-CN",
    "data" : {
        "symbol" : "BTC/USD"
    }
}
```

**curl call example**

> curl -X POST "[https://api.trade.hkvax.com/v1/quotation/quotation/query/depthData](https://api.trade.hkvax.com/v1/quotation/quotation/query/depthData)" -H "accept: \*/*" -H "SIGNATURE:   your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"symbol\\": \\"BTC/USD\\"}}"

**Response Parameters**

| Parameter     | Meaning | Type | Required | Description                       |
| ------------- | ------------- | --------- | -------- | -------------- |
| bidsMaxPrice  | Maximum buy order price | string    | true     | &nbsp;         |
| asksMinPrice  | Maximum sell order price  | string    | true     | &nbsp;         |
| depthPercent  | Depth percentage    | string    | true     | &nbsp;         |
| xMinPrice     | x-axis minimum price   | string    | true     | &nbsp;         |
| xMaxPrice     | y-axis minimum price   | string    | true     | &nbsp;         |
| yQuantity     | Maximum number of y-axis | string    | true     | &nbsp;         |
| symbol        | Pair        | string    | true     | &nbsp;         |
| bids          | Buyer in-depth data  | jsonArray | true     | &nbsp;         |
| bids.price    | Price         | string    | true     | &nbsp;         |
| bids.quantity | Quantity         | string    | true     | &nbsp;         |
| asks          | Seller in-depth data | jsonArray | true     | The structure is the same as bids |

**Response Example**

```json
{
	"code": "000000",
	"data": {
		"asksMinPrice": "0.70560000",
		"bidsMaxPrice": "0.23000000",
		"depthPercent": "206.79",
		"symbol": "BTC/USD",
		"xMaxPrice": "0.49119000",
		"xMinPrice": "0.44441000",
		"yQuantity": "0.0000",
		"bids": [{
			"price": "0.46780000",
			"quantity": "0.0000"
		}, {
			"price": "0.46733220",
			"quantity": "0.0000"
		}],
		"asks": [{
			"price": "0.46826780",
			"quantity": "0.0000"
		}, {
			"price": "0.46873560",
			"quantity": "0.0000"
		}]
	},
	"msg": "success"
}
```

## Candles

Get historical candle data of a specified pair.

**HTTP Request**

> POST /quotation/kline/get/quotationHistory

**Request Parameters**

| Parameter | Meaning | Type | Required | Description                                      |
| --------- | ------------ | ------ | -------- | ------------------------------------------------------------ |
| symbol    | Pair      | string | true     | &nbsp;                                                       |
| startTime | Start time    | long   | true     | unix timestamp                                                  |
| endTime   | End time    | long   | true     | unix timestamp                                                  |
| range     | Time range of data | int    | true     | Corresponding value<br>1MIN(60 * 1000)<br>5MIN(5 * 60 * 1000)<br>15MIN(15 * 60 * 1000)<br>30MIN(30 * 60 * 1000)<br>1HOUR(60 * 60 * 1000)<br>2HOUR(2 * 60 * 60 * 1000)<br>4HOUR(4 * 60 * 60 * 1000)<br>6HOUR(6 * 60 * 60 * 1000)<br>12HOUR(12 * 60 * 60 * 1000)<br>1DAY(24 * 60 * 60 * 1000)<br>1WEEK(7 * 24 * 60 * 60 * 1000) |

**Request Example**

```json
{
    "lang" : "zh-CN",
    "data" : {
        "symbol" : "BTC/USD",
        "startTime" : 1569405600000,
        "endTime" : 1569407400000,
        "range" : 300000
    }
}
```

**curl call example**

> curl -X POST "[https://api.trade.hkvax.com/v1/quotation/kline/get/quotationHistory](https://api.trade.hkvax.com/v1/quotation/kline/get/quotationHistory)" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"symbol\\": \\"BTC/USD\\", \\"startTime\\" : 1569405600000, \\"endTime\\" : 1569407400000, \\"range\\" : 300000}}"

**Response Parameters**

Under data is a jsonArray with quotationHistory, and the detailes are as follows:

| Parameter | Meaning | Type | Required | Description |
| -------- | -------- | ------ | -------- | ---------- |
| first    | Opening price   | string | true     | &nbsp;     |
| last     | Closing price    | string | true     | &nbsp;     |
| max      | Highest price  | string | true     | &nbsp;     |
| min      | Lowest price  | string | true     | &nbsp;     |
| quantity | Volume  | string | true     | &nbsp;     |
| time     | Time    | long   | true     | unix timestamp |

**Response Example**

```json
{
	"code": "000000",
	"msg": "success",
	"data": {
		"quotationHistory": [{
			"first": "0.23000000",
			"last": "0.23000000",
			"max": "0.23000000",
			"min": "0.23000000",
			"quantity": "0.000000"
			"time": 1569405600000
		}, {
			"first": "0.23000000",
			"last": "0.23000000",
			"max": "0.23000000",
			"min": "0.23000000",
			"quantity": "0.000000",
			"time": 1569405900000
		}]
	}
}
```

## The Latest Trade

Get the latest trade data of a specified pair.

**HTTP Request**

> POST /quotation/quotation/query/latestTradeQuotation

**Request Parameters**

| Parameter | Meaning | Type | Required | Description |
| ------ | -------- | ------ | -------- | ------ |
| symbol | pair   | string | true     | &nbsp; |

**Request Example**

```json
{
    "lang" : "zh-CN",
    "data" : {
        "symbol" : "BTC/USD"
    }
}
```

**curl call example**

> curl -X POST "[https://api.trade.hkvax.com/v1/quotation/quotation/query/latestTradeQuotation](https://api.trade.hkvax.com/v1/quotation/quotation/query/latestTradeQuotation)" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"symbol\\": \\"BTC/USD\\"}}"

**Response Parameters**

| Parameter | Meaning | Type | Required | Description |
| -------- | -------- | ------ | -------- | ----------------- |
| dealNo   | No. | long   | true     | &nbsp;            |
| symbol   | Pair  | string | true     | &nbsp;            |
| price    | Execution price | string | true     | &nbsp;            |
| action   | Buy or sell    | string | true     | buy:BUY<br>sell:SELL |
| quantity | Quantity | string | true     | &nbsp;            |
| utcDeal  | Time    | long   | true     | unix timestamp        |

**Response Example**

```json
{
	"code": "000000",
	"msg": "success",
	"data": {
		"action": "SELL",
		"dealNo": 1645640033529819137,
		"price": "0.230000000000000000000000000000",
		"quantity": "1000.000000000000000000000000000000",
		"symbol": "BTC/USD",
		"utcDeal": 1569404634172
	}
}
```

## Order Book

Get the current order book data of the specified pair.

**HTTP Request**

> POST /quotation/order/query/orderBook

**Request Parameters**

| **Parameter** | Meaning | Type | Required | Description |
| --------- | -------- | ------ | -------- | ------------------------------------------------------------ |
| symbol    | pair   | string | true     | &nbsp;                                                       |
| depthStep | Depth value | string | true | Corresponding value<br>Ten digits of depth: 10<br>Integer digits: 1<br>One decimal place: 01<br>Two decimal places: 001<br>Three decimal places: 0001<br > And so on |

**curl call example**

> curl -X POST "https://api.trade.hkvax.com/v1/quotation/order/query/orderBook" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"symbol\\": \\"DIMT/USD\\", \\"depthStep\\":\\"0000001\\"}}"

**Response Parameters**

| **Parameter**     | Meaning | Type | Required | Description                              |
| ---------------------- | -------------- | --------- | -------- | ----------------- |
| symbol                 | Pair         | string    | true     | &nbsp;            |
| limitSize              | Order book lines     | int       | true     | &nbsp;            |
| step                   | Depth     | string    | true     | &nbsp;            |
| askQuantity            | The number of seller progress bars | string    | true     | &nbsp;            |
| bidQuantity            | The number of buyer progress bars | string    | true     | &nbsp;            |
| askList                | Sell orders           | jsonArray | true     | &nbsp;            |
| askList.amount         | Price          | string    | true     | &nbsp;            |
| askList.quantity       | Quantity          | string    | true     | &nbsp;            |
| askList.limitPrice     | Limit price          | string    | true     | &nbsp;            |
| askList.amountAccuracy | Accurate amount | string    | true     | &nbsp;            |
| bidList                | Buy orders          | jsonArray | true     | The structure is same as askList |

**Response Example**

```json
{
	"code": "000000",
	"data": {
		"askList": [{
			"amount": "0.549000",
			"amountAccuracy": "0.549000",
			"limitPrice": "0.001830",
			"quantity": "300.0000"
		}],
		"askQuantity": "300.0000",
		"bidList": [{
			"amount": "0.182000",
			"amountAccuracy": "0.182000",
			"limitPrice": "0.001820",
			"quantity": "100.0000"
		}],
		"bidQuantity": "100.0000",
		"latestDeal": {
			"direction": 0,
			"price": "0.001813",
			"usdPrice": "0.00181300"
		},
		"limitSize": 15,
		"step": "0000001",
		"symbol": "DIMT/USD"
	},
	"msg": "success",
	"success": true,
	"traceId": "6c5be4dde02e4a8f"
}
```

## Latest trades

Get the latest 12 trades of a specified pair.

**HTTP Request**

> POST /quotation/order/query/dealList

**Request Parameters**

| Parameter | Meaning | Type | Required | Description |
| ------ | -------- | ------ | -------- | ------ |
| symbol | pair   | string | true     | &nbsp; |

**curl call example**

> curl -X POST "[https://api.trade.hkvax.com/v1/quotation/order/query/dealList](https://api.trade.hkvax.com/v1/quotation/order/query/dealList)" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"symbol\\": \\"DIMT/USD\\"}}"

**Response Parameters**

| Parameter | Meaning | Type | Required | Description |
| -------- | -------- | ------ | -------- | -------------- |
| action   | Buy or sell    | string | true     | buy:BUY sell:SELL |
| dealNo   | No. | long   | true     | &nbsp;         |
| price    | Execution price | string | true     | &nbsp;         |
| quantity | Quantity    | string | true     | &nbsp;         |
| symbol   | Pair  | string | true     | &nbsp;         |
| utcDeal  | Execution Time | long   | true     | unix timestamp     |

**Response Example**

```json
{
	"code": "000000",
	"data": [{
		"action": "BUY",
		"dealNo": 1647517554993250305,
		"price": "0.001813",
		"quantity": "200.0000",
		"symbol": "DIMT/USD",
		"utcDeal": 1571195178105
	}, {
		"action": "SELL",
		"dealNo": 1647517531935064065,
		"price": "0.001812",
		"quantity": "100.0000",
		"symbol": "DIMT/USD",
		"utcDeal": 1571195156310
	}],
	"msg": "success",
	"success": true,
	"traceId": "b701341fe31412ad"
}
```

## Best Bid& Best Ask

Get the best bid or best ask of the specified pair.

**HTTP Request**

> POST /quotation/order/query/marketBestPrice

**Request Parameters**

| Parameter | Meaning | Type | Required | Description |
| ------ | -------- | ------ | -------- | ----------------- |
| symbol | Pair  | string | true     | &nbsp;            |
| action | Buy or sell | string | true     | buy:BUY<br>sell:SELL |

**curl call example**

> curl -X POST "https://api.trade.hkvax.com/v1/quotation/order/query/marketBestPrice" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"symbol\\": \\"IAP/USD\\",\\"action\\":\\"BUY\\"}}"

**Request Parameters**

| Parameter | Meaning | Type | Required | Description |
| --------- | -------- | ------ | -------- | -------------- |
| action    | Buy or sell | string | true     | buy:BUY<br>sell:SELL |
| bestPrice | Execution price | string | true     | &nbsp;         |
| symbol    | Pair  | string | true     | &nbsp;            |

**response example**

```json
{
	"code": "000000",
	"data": {
		"action": "BUY",
		"bestPrice": "0.046912",
		"symbol": "IAP/USD"
	}
}
```



# Trade

## Place an Order

Place a new order.

**Frequency Limit**

Place an order at most once every 2 seconds

**HTTP Request**

> POST /order/cex/user/order/create

**Request Parameters**

Parameter| Meaning        | Type | Required | Description 
---|---|---|---|---
symbol | Pair | string | true | &nbsp;
action | Buy or sell | string | true | buy:BUY<br>sell:SELL
orderType | Order type | string | true | Limit order: LMT<br>Market order: MKT
limitPrice | Order price | decimal | false | It is not necessary to pass when it's a market order 
quantity | Order quantity | decimal | false | It is not necessary to pass when buying at market price 
amount | Amount | decimal | false | It is necessary to pass when buying at market price 

**Request Example**

> Buy at market price: The amount below indicates that the frozen USD amount is 10,000

```json
{
    "lang" : "zh-CN",
    "data" : {
        "symbol" : "LTC/USD",
        "action" : "BUY",
        "orderType" : "MKT",
        "amount" : 10000
    }
}
```

> Sell at market price: The quantity below indicates that the number of LTC to be frozen is 10,000

```json
{
    "lang" : "zh-CN",
    "data" : {
        "symbol" : "LTC/USD",
        "action" : "SELL",
        "orderType" : "MKT",
        "quantity" : 10000
    }
}
```

> Example of limit buying: The amount of frozen USD is (0.2 * 10000 = 2000)

```json
{
    "lang" : "zh-CN",
    "data" : {
        "symbol" : "LTC/USD",
        "action" : "BUY",
        "orderType" : "LMT",
        "limitPrice" : 0.2,
        "quantity" : 10000
    }
}
```

> Example of limit selling: The quantity below indicates that the number of LTC to be frozen is 10,000

```json
{
    "lang" : "zh-CN",
    "data" : {
        "symbol" : "LTC/USD",
        "action" : "SELL",
        "orderType" : "LMT",
        "limitPrice" : 0.3,
        "quantity" : 10000
    }
}
```

**Response Parameters**

Parameter | Meaning | Type | Required | Description 
---|---|---|---|---
orderNo | Order No. | long | true | successful return of order placed 

**Example of Successful Response**

```json
{
    "code" : "000000",
    "msg" : "success",
    "data" : {
        "orderNo" : 123
    }
}
```

**curl call example**

> curl -X POST "[https://api.trade.hkvax.com/v1/order/cex/user/order/create](https://api.trade.hkvax.com/v1/order/cex/user/order/create)" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"symbol\\": \\"BTC/USD\\", \\"action\\": \\"BUY\\", \\"orderType\\":\\"MKT\\", \\"amount\\": 10000}}"

## Cancel an Order

Cancel a previously placed order.

**HTTP Request**

> POST /order/cex/user/order/cancel

**Request Parameters**

Parameter | Meaning | Type | Required | Description 
---|---|---|---|---
orderNo | Order No. | long | true | &nbsp;

**Request Example**

```json
{
    "lang" : "zh-CN",
    "data" : {
        "orderNo" : 123
    }
}
```

**curl all example**

> curl -X POST "https://api.trade.hkvax.com/v1/order/cex/user/order/cancel" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"orderNo\\": 123}}"

**Response Parameters**

Parameter | Meaning | Type | Required | Description 
---|---|---|---|---
orderNo | Order No. | long | true | &nbsp;

**Example of Successful Response**

```json
{
    "code" : "000000",
    "msg" : "success",
    "data" : {
        "orderNo" : 123
    }
}
```



# User Data Query

## Account Balance

Query the available balance of the specified currency.

**HTTP Request**

> POST /user/cex/user/asset/query/currencyAsset

**Request Parameters**

| Parameter | Meaning | Type | Required | Description |
| -------- | -------- | ------ | -------- | ------ |
| currency | Currency     | string | true     | &nbsp; |

**Request Example**

```json
{
    "lang" : "zh-CN",
    "data" : {
        "currency" : "USD"
    }
}
```

**curl call example**

> curl -X POST "https://api.trade.hkvax.com/v1/user/cex/user/asset/query/currencyAsset" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"currency\\": \\"USD\\"}}"

**Response Parameters**

| Parameter | Meaning | Type | Required | Description |
| --------------- | ------------ | ------ | -------- | ------ |
| availableAmount | Current available balance| string | true     | &nbsp; |

**Response Example**

```json
{
    "code" : "000000",
    "msg" : "success",
    "data" : {
        "availableAmount" : "1372.624"
    }
}
```

## Order List

Query the order list by criteria.

**HTTP Request**

> POST /user/cex/user/order/query/orderList

**Request Parameters**

Parameter | Meaning | Type | Required | Description 
---|---|---|---|---
gmtStart | Start time | long | true | unix timestamp
gmtEnd | End time | long | true | unix timestamp
symbol | Pair | string | false | &nbsp;
orderAction | Buy or sell | string | false | buy:BUY<br>sell:SELL
orderType | Order type| string | true | Limit order: LMT<br>Market order: MKT<br>All: ALL
orderStatus | Order status | string | true | All filled`:DEAL<br/>Unfilled：MISMATCH<br/>Partially filled：PARTIAL<br/>Canceling：FIRSTTRIAL<br/>A partially order is successfully canceled：CANCEL_PARTIAL<br/>An unfilled order is successfully canceled：CANCEL_MISMATCH 
pageRows | Number of rows per page | int | true | &nbsp;
currPage | Current page | int | true | from 1

**Request Example**

```json
{
    "lang" : "zh-CN",
    "data" : {
        "gmtStart" : 1569291600000,
        "gmtEnd" : 1569377997876,
        "orderStatus" : 0,
        "pageRows" : 10,
        "currPage" : 1
    }
}
```

**curl call example**

> curl -X POST "https://api.trade.hkvax.com/v1/user/cex/user/order/query/orderList" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"gmtStart\\": 1569291600000, \\"gmtEnd\\": 1569377997876, \\"orderStatus\\": 0,\\"pageRows\\": 10,\\"currPage\\": 1}}"

**Response Parameters (Pagination Response)**

Parameter | Meaning | Type | Required | Description 
---|---|---|---|---
orderNo | Order No. | long | true | &nbsp;
symbol | Pair | string | true | &nbsp;
action | Buy or sell | string | true | Buy: BUY<br> Sell: SELL
quantity | Quantity | string | false | Buy market price<br> is 0 or empty
quantityDeal | Execution quantity | string | false | &nbsp;
quantityRemain | Remain quantity | string | false | &nbsp;
fee | Fee| string | false | string type number
feeCurrency | Fee currency | string | false | &nbsp;
orderType | Order type| string | false | Limit order: LMT<br>Market order: MKT
priceAverage | Average price | string | false | &nbsp;
priceLimit | Price of a limit order | string | false | &nbsp;
status | Order status | string | true | All filled`:DEAL<br/>Unfilled：MISMATCH<br/>Partially filled：PARTIAL<br/>Canceling：FIRSTTRIAL<br/>A partially order is successfully canceled：CANCEL_PARTIAL<br/>An unfilled order is successfully canceled：CANCEL_MISMATCH 
gmtCreate | Order creation  time | string | true | unix timestamp
amount | Order amount | string | false | &nbsp;
amountRemain | Remaining amount | string | false | &nbsp;

**Response Example**

```json
{
    "code" : "000000",
    "msg" : "success",
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
                "symbol" : "BTC/USD",
                "action" : "BUY",
                "quantity" : "0",
                "quantityDeal" : "10",
                "quantityRemain" : "30",
                "fee" : "3.3",
                "feeCurrency" : "BTC",
                "orderType" : "MKT",
                "priceAverage" : "300",
                "status" : "DEAL",
                "gmtCreate" : "1569291600000",
                "amount" : "0",
                "amountRemain" : "0"
            }
        ]
    }
}
```

## Order Details

Query the status and details of the specified order.

**HTTP Request**

> POST /user/cex/user/mm/query/orderDetail

**Request Parameters**

A list of order No to be queried, separated by commas

**Request Example**

```json
{
    "lang" : "zh-CN",
    "data" : [
        123,
        456,
        789
    ]
}
```

**curl call example**

> curl -X POST "[https://api.trade.hkvax.com/v1/user/cex/user/mm/query/orderDetail](https://api.trade.hkvax.com/v1/user/cex/user/mm/query/orderDetail)" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":[123, 456, 789]}"

**Response Parameters**

Parameter | Meaning | Type | Required | Description 
---|---|---|---|---
orderNo | Order No. | long | true | &nbsp;
symbol | Pair | string | true | &nbsp;
action | Buy or sell | int | true | buy:2<br>sell:1
orderType | Order type| int | true | Limit order: 1<br>Market order: 2
priceLimit | Limit price of a limit order | decimal | false | &nbsp;
quantity | Quantity | decimal | false | Buy market price<br> is 0 or empty
quantityRemain | Remaining quantity | decimal | false | &nbsp;
status | Order status | int | true | Normal: 0<br>Retired: 1
gmtCreate | Order creation time | long | true | unix timestamp

**Responce Example**

```json
{
    "code" : "000000",
    "msg" : "success",
    "data" : [
        {
            "orderNo" : 123,
            "symbol" : "BTC/USD",
            "action" : 2,
            "quantity" : 0,
            "quantityRemain" : 30,
            "orderType" : 2,
            "status" : 0,
            "gmtCreate" : 1569291600000,
            "priceLimit" : 0
        }
    ]
}
```

## Trades

Query your trades list by criteria.

**HTTP Request**

> POST /user/cex/user/order/query/dealOrderList

**Request Parameters**

Parameter | Meaning | Type | Required | Description 
---|---|---|---|---
gmtStart | Start time | long | false | unix timestamp
gmtEnd | End time | long | false | unix timestamp
symbol | Pair | string | false | &nbsp;
orderAction | Buy or sell | string | false | Buy: BUY Sell: SELL  vacant means all
orderType | Order type| string | false | Limit order: LMT Market order: MKT Empty means all 
pageRows | Row numbers per page | int | true | &nbsp;
currPage | Current page | int | true | from 1

**Request Example**

```json
{
    "lang" : "zh-CN",
    "data" : {
        "gmtStart" : 1569291600000,
        "pageRows" : 10,
        "currPage" : 1
    }
}
```

**curl call example**

> curl -X POST "https://api.trade.hkvax.com/v1/user/cex/user/order/query/dealOrderList" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"gmtStart\\": 1569291600000, \\"pageRows\\": 10,\\"currPage\\": 1}}"

**Response Parameters(Pagination Response)**

Parameter | Meaning | Type | Required | Description 
---|---|---|---|---
symbol | Pair | string | true | &nbsp;
action | Buy or sell | string | true | buy:BUY<br>sell:SELL
price | Execution price | decimal | true | &nbsp;
quantity | Execution quantity | decimal | true | &nbsp;
amount | Execution amount | decimal | true | &nbsp;
utcDeal | Execution time | long | true | unix timestamp
dealNo | Execution No. | long | true | &nbsp;
orderType | Order type| string | true | limit order:LMT<br>market order:MKT
fee | Fee | decimal | true | &nbsp;
feeCurrency | Fee currency | string | false | &nbsp;

**Response Example**

```json
{
    "code" : "000000",
    "msg" : "success",
    "data" : {
        "pagination" : {
            "pageSize" : 22,
            "currPage" : 1,
            "totalRows" : 212,
            "pageRows" : 10
        },
        "list" : [
            {
                "symbol" : "BTC/USD",
                "action" : "BUY",
                "price" : 319.22,
                "quantity" : 2.54,
                "amount" : 12.3,
                "utcDeal" : 1569291600000,
                "dealNo" : 1232,
                "fee" : "3.3",
                "feeCurrency" : "BTC",
                "orderType" : "MKT"
            }
        ]
    }
}
```

## Order Trades

Query trades details of the specified order.

**HTTP Request**

> POST /user/cex/user/order/query/dealDetail

**Request Parameters**

Parameter | Meaning | Type | Required | Description 
---|---|---|---|---
orderNo | Order No. | long | true | &nbsp;

**Request Example**

```json
{
    "lang" : "zh-CN",
    "data" : {
        "orderNo" : 123
    }
}
```
**curl call example**

> curl -X POST "[https://api.trade.hkvax.com/v1/user/cex/user/order/query/dealDetail](https://api.trade.hkvax.com/v1/user/cex/user/order/query/dealDetail)" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"orderNo\\": 123}}"

**Response Parameters(Array)**

Parameter | Meaning | Type | Required | Description 
---|---|---|---|---
dealTime | Execution time | string | true | unix timestamp
dealPrice | Execution price | string | true | &nbsp;
dealQuantity | Execution quantity | string | true | &nbsp;

**Response Example**

```json
{
    "code" : "000000",
    "msg" : "success",
    "data" : [
        {
            "dealTime" : "1569291600000",
            "dealPrice" : "3.3",
            "dealQuantity" : "300"
        }
    ]
}
```
