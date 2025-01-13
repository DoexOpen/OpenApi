# Public Broker Rest API (2025-01-01)

## General API Information

* All endpoints return a JSON object or array.
* Data is returned in **ascending** order. Older data is first, newer data is last.
* All time/timestamp related variables are in milliseconds.
* HTTP `4XX` return codes indicate a request error; the issue is on the sender's side.
* HTTP `429` return codes indicate the request rate limit has been exceeded.
* HTTP `418` return codes indicate the IP has been auto-banned after receiving a `429` code and continuing to send requests.
* HTTP `5XX` return codes indicate internal system errors; the issue is on the broker's side. **Do not** treat this as a failed task because the execution status is **unknown**; it could be successful or failed.
* Any endpoint can return an ERROR; the error payload is as follows:

```javascript
{
  "code": -1121,
  "msg": "Invalid symbol."
}
```

* Detailed error codes and messages can be found in the error code file.
* For `GET` endpoints, parameters must be sent as a `query string`.
* For `POST`, `PUT`, and `DELETE` endpoints, parameters must be sent as a `query string` or in the `request body` with the content type set to `application/x-www-form-urlencoded`. Parameters can be mixed in the `query string` and `request body` if needed.
* Parameters can be sent in any order.
* If a parameter exists in both the `query string` and `request body`, only the `query string` parameter will be used.

### Rate Limits

* The `rateLimits` array in `/exapi/v1/brokerInfo` contains the current broker's `REQUEST_WEIGHT` and `ORDER` rate limits.
* If any rate limit is exceeded, a `429` code will be returned.
* Each endpoint has a `weight` attribute, which determines how much capacity the request consumes (e.g., `weight`=2 means the request counts as two requests). Endpoints that return more data or perform tasks on multiple symbols may have a higher `weight`.
* When a `429` code is returned, you must stop sending requests.
* **Repeatedly violating rate limits and/or not stopping requests after receiving a 429 code will result in an IP ban (error code 418).**
* IP bans are tracked and **ban durations are adjusted** (ranging from **2 minutes to 3 days** for repeated violations).

### Endpoint Security Types

* Each endpoint has a security type, which determines how you interact with it.
* API keys must be sent in the REST API header with the name `X-BH-APIKEY`.
* API keys and secret keys are **case-sensitive**.
* By default, API keys can access all security endpoints.

Security Type | Description
------------ | ------------
NONE | Endpoint can be accessed freely.
TRADE | Endpoint requires a valid API key and signature.
USER_DATA | Endpoint requires a valid API key and signature.
USER_STREAM | Endpoint requires a valid API key.
MARKET_DATA | Endpoint requires a valid API key.

* `TRADE` and `USER_DATA` endpoints are `SIGNED` endpoints.

### SIGNED (TRADE and USER_DATA) Endpoint Security

* `SIGNED` endpoints require a `signature` parameter, sent in the `query string` or `request body`.
* Endpoints are signed using `HMAC SHA256`. The `HMAC SHA256 signature` is the result of encrypting the key with `HMAC SHA256`. Use your `secretKey` as the key and `totalParams` as the value to complete this encryption.
* `signature` is **case-insensitive**.
* `totalParams` refers to the concatenation of the `query string` and `request body`.

### Timing Security

* A `SIGNED` endpoint also requires a `timestamp` parameter, which is the millisecond timestamp when the request is made.
* An optional parameter, `recvWindow`, specifies how long the request is valid in milliseconds. If `recvWindow` is not sent, **the default is 5000**.
* Currently, `recvWindow` is only used when creating orders.
* The logic for this parameter is as follows:

  ```javascript
  if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
    // process request
  } else {
    // reject request
  }
  ```

**Strict trading is closely related to timing**. Networks can sometimes be unstable or unreliable, causing inconsistent request times to the server. With `recvWindow`, you can specify how long the request is valid; otherwise, it will be rejected by the server.

**It is recommended to use a relatively small recvWindow (5000 or less)!**

### SIGNED Example (for POST /exapi/v1/order)

Here is a detailed example using Linux `echo`, `openssl`, and `curl` to demonstrate how to send a valid signed payload.

Key | Value
------------ | ------------
apiKey | tAQfOrPIZAhym0qHISRt8EFvxPemdBm5j5WMlkm3Ke9aFp0EGWC2CGM8GHV4kCYW
secretKey | lH3ELTNiFxCQTmi9pPcWWikhsjO04Yoqw3euoHUuOLC3GYBW64ZqzQsiOEHXQS76

Parameter Name | Parameter Value
------------ | ------------
symbol | ETHBTC
side | BUY
type | LIMIT
timeInForce | GTC
quantity | 1
price | 0.1
recvWindow | 5000
timestamp | 1538323200000

#### Example 1: In `queryString`

* **queryString:** symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000
* **HMAC SHA256 signature:**

```shell
[linux]$ echo -n "symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000" | openssl dgst -sha256 -hmac "lH3ELTNiFxCQTmi9pPcWWikhsjO04Yoqw3euoHUuOLC3GYBW64ZqzQsiOEHXQS76"
(stdin)= 5f2750ad7589d1d40757a55342e621a44037dad23b5128cc70e18ec1d1c3f4c6
```

* **curl command:**

```shell
(HMAC SHA256)
[linux]$ curl -H "X-BH-APIKEY: tAQfOrPIZAhym0qHISRt8EFvxPemdBm5j5WMlkm3Ke9aFp0EGWC2CGM8GHV4kCYW" -X POST 'https://$HOST/exapi/v1/order?symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000&signature=5f2750ad7589d1d40757a55342e621a44037dad23b5128cc70e18ec1d1c3f4c6'
```

#### Example 2: In `request body`

* **requestBody:** symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000
* **HMAC SHA256 signature:**

```shell
[linux]$ echo -n "symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000" | openssl dgst -sha256 -hmac "lH3ELTNiFxCQTmi9pPcWWikhsjO04Yoqw3euoHUuOLC3GYBW64ZqzQsiOEHXQS76"
(stdin)= 5f2750ad7589d1d40757a55342e621a44037dad23b5128cc70e18ec1d1c3f4c6
```

* **curl command:**

```shell
(HMAC SHA256)
[linux]$ curl -H "X-BH-APIKEY: tAQfOrPIZAhym0qHISRt8EFvxPemdBm5j5WMlkm3Ke9aFp0EGWC2CGM8GHV4kCYW" -X POST 'https://$HOST/exapi/v1/order' -d 'symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000&signature=5f2750ad7589d1d40757a55342e621a44037dad23b5128cc70e18ec1d1c3f4c6'
```

#### Example 3: Mixed `queryString` and `request body`

* **queryString:** symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC
* **requestBody:** quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000
* **HMAC SHA256 signature:**

```shell
[linux]$ echo -n "symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTCquantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000" | openssl dgst -sha256 -hmac "lH3ELTNiFxCQTmi9pPcWWikhsjO04Yoqw3euoHUuOLC3GYBW64ZqzQsiOEHXQS76"
(stdin)= 885c9e3dd89ccd13408b25e6d54c2330703759d7494bea6dd5a3d1fd16ba3afa
```

* **curl command:**

```shell
(HMAC SHA256)
[linux]$ curl -H "X-BH-APIKEY: tAQfOrPIZAhym0qHISRt8EFvxPemdBm5j5WMlkm3Ke9aFp0EGWC2CGM8GHV4kCYW" -X POST 'https://$HOST/exapi/v1/order?symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC' -d 'quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000&signature=885c9e3dd89ccd13408b25e6d54c2330703759d7494bea6dd5a3d1fd16ba3afa'
```

***Note the difference in Example 3, there is no & between "GTC" and "quantity=1".***

## Public API Endpoints

### Terminology

* `base asset` refers to the `quantity` of the symbol.
* `quote asset` refers to the `price` of the symbol.

### ENUM Definitions

**Symbol Status:**

* TRADING - Trading
* HALT - Halted
* BREAK - Break

**Symbol Type:**

* SPOT - Spot

**Asset Type:**

* CASH - Cash
* MARGIN - Margin

**Order Status:**

* NEW - New order, no fills
* PARTIALLY_FILLED - Partially filled
* FILLED - Fully filled
* CANCELED - Canceled
* PENDING_CANCEL - Pending cancel
* REJECTED - Rejected

**Order Type:**

* LIMIT - Limit order
* MARKET - Market order
* LIMIT_MAKER - Limit maker order
* STOP_LOSS (unavailable now) - Unavailable
* STOP_LOSS_LIMIT (unavailable now) - Unavailable
* TAKE_PROFIT (unavailable now) - Unavailable
* TAKE_PROFIT_LIMIT (unavailable now) - Unavailable
* MARKET_OF_PAYOUT (unavailable now) - Unavailable

**Order Side:**

* BUY - Buy order
* SELL - Sell order

**Order Time in Force:**

* GTC
* IOC
* FOK

**Kline/Candlestick Intervals:**

m -> minutes; h -> hours; d -> days; w -> weeks; M -> months

* 1m
* 3m
* 5m
* 15m
* 30m
* 1h
* 2h
* 4h
* 6h
* 8h
* 12h
* 1d
* 3d
* 1w
* 1M

**Rate Limit Types (rateLimitType)**

* REQUESTS_WEIGHT
* ORDERS

**Rate Limit Intervals**

* SECOND
* MINUTE
* DAY

### General Endpoints

#### Broker Information

```shell
GET /exapi/v1/brokerInfo
```

Current broker trading rules and symbol information.

**Weight:**
0

**Parameters:**
NONE

**Response:**

```javascript
{
  "timezone": "UTC",
  "serverTime": 1538323200000,
  "rateLimits": [{
      "rateLimitType": "REQUESTS_WEIGHT",
      "interval": "MINUTE",
      "limit": 1500
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "limit": 20
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "limit": 350000
    }
  ],
  "brokerFilters": [],
  "symbols": [{
    "symbol": "ETHBTC",
    "status": "TRADING",
    "baseAsset": "ETH",
    "baseAssetPrecision": "0.001",
    "quoteAsset": "BTC",
    "quotePrecision": "0.01",
    "icebergAllowed": false,
    "filters": [{
      "filterType": "PRICE_FILTER",
      "minPrice": "0.00000100",
      "maxPrice": "100000.00000000",
      "tickSize": "0.00000100"
    }, {
      "filterType": "LOT_SIZE",
      "minQty": "0.00100000",
      "maxQty": "100000.00000000",
      "stepSize": "0.00100000"
    }, {
      "filterType": "MIN_NOTIONAL",
      "minNotional": "0.00100000"
    }]
  }]
}
```

### Market Data Endpoints

#### Order Book

```shell
GET /exapi/quote/v1/depth
```

**Weight:**

Depends on the limit:

Limit | Weight
------------ | ------------
5, 10, 20, 50, 100 | 1
500 | 5
1000 | 10

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 100; max 100.

**Note:** Setting limit=0 will return a lot of data.

**Response:**

[Price, Quantity]

```javascript
{
  "bids": [
    [
      "3.90000000",   // Price
      "431.00000000"  // Quantity
    ],
    [
      "4.00000000",
      "431.00000000"
    ]
  ],
  "asks": [
    [
      "4.00000200",  // Price
      "12.00000000"  // Quantity
    ],
    [
      "5.10000000",
      "28.00000000"
    ]
  ]
}
```

#### Recent Trades

```shell
GET /exapi/quote/v1/trades
```

Get the most recent trades (up to 60).

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 60; max 60.

**Response:**

```javascript
[
  {
    "price": "4.00000100",
    "qty": "12.00000000",
    "time": 1499865549590,
    "isBuyerMaker": true
  }
]
```

#### Kline/Candlestick Data

```shell
GET /exapi/quote/v1/klines
```

Kline/candlestick data for a symbol. Klines are uniquely identified by their open time.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
interval | ENUM | YES |
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.

* If startTime and endTime are not sent, only the most recent klines are returned.

**Response:**

```javascript
[
  [
    1499040000000,      // Open time
    "0.01634790",       // Open price
    "0.80000000",       // High price
    "0.01575800",       // Low price
    "0.01577100",       // Close price
    "148976.11427815",  // Volume
    1499644799999,      // Close time
    "2434.19055334",    // Quote asset volume
    308,                // Number of trades
    "1756.87402397",    // Taker buy base asset volume
    "28.46694368"       // Taker buy quote asset volume
  ]
]
```


