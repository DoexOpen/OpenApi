# Contract API Documentation Table of Contents

- [Public Endpoints](#Broker-Contract-Public-Endpoint)
  - [Base Url](#base-url)
  - [brokerInfo](#brokerinfo)
  - [Index Price](#index)
  - [Order Book Depth](#depth)
  - [Latest Trades](#trades)
  - [Kline Data](#klines)

- [Private Endpoints](#private-endpoints)
  - [Signature Lookup](#signature)
  - [Place Contract Order](#place-contract-order)
  - [Unfilled Futures Orders](#unfilled-futures-orders)
  - [Cancel Contract Order](#Contract-Order-Cancellation)
  - [Contract Order History](#contract-order-history)
  - [Order Details](#order-details)
  - [Trade History](#trade-history)
  - [Current Positions](#current-positions)
  - [Contract Account Balance](#contract-account-balance)
  - [Modify Margin](#modify-margin)
  - [Modify Margin Mode](#modify-margin-mode)
  - [Query Margin Mode](#query-margin-mode)
  - [Modify Merge Mode](#modify-merge-mode)
  - [Query Merge Mode](#query-merge-mode)
  - [Set Leverage](#set-leverage)
  - [Query Leverage](#query-leverage)

- [Key Parameter Explanations](#Key-parameter-explanation)
  - [side](#side)
  - [priceType](#pricetype)
  - [timeInForce](#timeinforce)
  - [orderType](#ordertype)

# Broker Contract Public Endpoint

## Base Url

The base url of broker open API can be found [here](../endpoint.md)

## `brokerInfo`

Current broker trading rules and symbol information.

### **Request Weight:**

0

### **Request Url:**

```bash
GET /exapi/v1/brokerInfo
```

### **Parameters：**

无

### **Response:**
name|type|example|description
------------ | ------------ | ------------ | ------------
`timezone`|string|`UTC`|Timezone of timestamp
`serverTime`|long|`1554887652929`|Retrieves the current time on server (in ms).

In the `rateLimits` field:
Order api request limit will be displayed.

name|type|example|description
------------ | ------------ | ------------ | ------------
`rateLimitType`|string|`ORDERS`|Rate Limit type
`interval`|string|`SECOND`|Rate Limit interval
`limit`|string|`1500`|Rate Limit value within the interval.

In the `contracts` field:
All actively trading contracts will be displayed.

name|type|example|description
------------ | ------------ | ------------ | ------------
`symbol`|string|`BTCUSD`|Name of the contract.
`status`|string|`TRADING`|Status of the contract.
`baseAsset`|string|`BTCUSD`|Base Asset of the trading pair. In the case with contracts, the contract itself is the base asset.
`baseAssetPrecision`|float|`0.001`|Precision of the contract quantity (baseAsset).
`quoteAsset`|string|`USDT`|Quote asset for the contract. Meaning the contract is quoted in that currency.
`quoteAssetPrecision`|float|`0.001`|Precision of the contract price (quoteAsset).
`inverse`|bool|`true`|Whether the contract is inverse.
`index`|string|`BTCUSDT`|Index symbol of the underlying asset. Index price can be accessed at the `index` endpoint. For instance, `BTC-SWAP-USDT` uses `BTCUSDT` for index price.
`contractMultiplier`|string|`true`|The multiplier of contract.
`icebergAllowed`|string|`false`|Whether iceberg orders are allowed.


For `filters` in `contracts` field:

name|type|example|description
------------ | ------------ | ------------ | ------------
`filterType`|string|`PRICE_FILTER`|Type of the filter.
`minPrice`|float|`0.001`|Minimum price allowed
`maxPrice`|float|`100000.00000000`|Maximum price allowed
`tickSize`|float|`0.001`|Precision of the contract price.
`minQty`|float|`0.01`|Minimum quantity allowed
`maxQty`|float|`100000.00000000`|Maximum quantity allowed
`stepSize`|float|`0.001`|Precision of the contract quantity
`minNotional`|float|`1`|Minimum trade size (quantity * price)

In the `riskLimits` filed:

name|type|example|description
------------ | ------------ | ------------ | ------------
`quantity`|float|`100`| Positions below this amount follows the following requirement.
`initialMargin`|float|`0.1`|Initial margin rate requirement.
`maintMargin`|float|`0.03`|Minimum maintenance margin rate requirement.

### **Example:**
```json
{
  "timezone": "UTC",
  "serverTime": "1736752701537",
  "brokerFilters": [],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalUnit": 1,
      "limit": 600
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalUnit": 30,
      "limit": 30
    }
  ],
  "contracts": [
    {
      "filters": [
        {
          "minPrice": "0.00001",
          "maxPrice": "100000.00000000",
          "tickSize": "0.00001",
          "filterType": "PRICE_FILTER"
        },
        {
          "minQty": "300",
          "maxQty": "100000.00000000",
          "stepSize": "1",
          "filterType": "LOT_SIZE"
        },
        {
          "minNotional": "20",
          "filterType": "MIN_NOTIONAL"
        }
      ],
      "exchangeId": "301",
      "symbol": "1000BONK-SWAP-USDT",
      "symbolName": "1000BONK-SWAP-USDTUSDT",
      "status": "TRADING",
      "baseAsset": "1000BONK-SWAP-USDT",
      "baseAssetPrecision": "1",
      "quoteAsset": "USDT",
      "quoteAssetPrecision": "0.00001",
      "icebergAllowed": false,
      "inverse": false,
      "index": "1000BONKUSDT",
      "marginToken": "USDT",
      "marginPrecision": "0.00000001",
      "contractMultiplier": "1.0",
      "underlying": "1000BONK",
      "riskLimits": [
        {
          "riskLimitId": "200000629",
          "quantity": "175000.0",
          "initialMargin": "0.02",
          "maintMargin": "0.01"
        },
        {
          "riskLimitId": "200000630",
          "quantity": "850000.0",
          "initialMargin": "0.05",
          "maintMargin": "0.025"
        },
        {
          "riskLimitId": "200000631",
          "quantity": "3500000.0",
          "initialMargin": "0.1",
          "maintMargin": "0.05"
        },
        {
          "riskLimitId": "200000632",
          "quantity": "7000000.0",
          "initialMargin": "0.2",
          "maintMargin": "0.1"
        }
      ]
    }
  ],
  "tokens": [
    {
      "orgId": "9001",
      "tokenId": "ACT",
      "tokenName": "ACT",
      "tokenFullName": "The AI Prophecy",
      "allowWithdraw": true,
      "allowDeposit": true,
      "chainTypes": [
        {
          "chainType": "Solana",
          "allowDeposit": true,
          "allowWithdraw": true
        }
      ]
    },
    {
      "orgId": "9001",
      "tokenId": "ACX",
      "tokenName": "ACX",
      "tokenFullName": "Across Protocol",
      "allowWithdraw": true,
      "allowDeposit": true,
      "chainTypes": [
        {
          "chainType": "ERC20",
          "allowDeposit": true,
          "allowWithdraw": true
        }
      ]
    }
  ]
}
```

## `index`

Underlying asset index price.

### **Request Weight:**
0

### **Request Url:**
```bash
GET /exapi/quote/v1/contract/index
```
### **Parameters：**
name|type|required|default|description
------------ | ------------ | ------------ | ------------ | ----
symbol|string|`NO`||Underlying index symbol. If this parameter is not sent, all symbols
will be returned.

### **Response:**
name|type|example|description
------------ | ------------ | ------------ | ------------
`index`|float|`8342.73`|The index price of the instrument.
`EDP`|float|`8432.32`|The EDP (estimated delivery price of the contract). The average price of the index in the last 1o minutes. This will be the price on which the contract is going to be settled.

### **Example:**
```js
{
  "index":{
    "BTCUSDT":"8243.21666667",
    "OKBUSDT":"1.482",
    "BNBUSDT":"31.2658",
    "HTUSDT":"3.1209",...
    },
  "edp":{
    "BTCUSDT":"8258.98505556",
    "OKBUSDT":"1.48578333",
    "BNBUSDT":"31.48741917",
    "HTUSDT":"3.14308",...
  }
}
```

## `depth`

Retrieves the contracts order book.


### **Request Weight:**

Adjusted based on the limit:

Limit|Weight
------------ | ------------
5, 10, 20, 50, 100|1
500|5
1000|10


### **Request Url:**
```
GET /exapi/quote/v1/contract/depth
```

### **Parameters:**
parameter|type|required|default|description
------------ | ------------ | ------------ | ------------ | -----
`symbol`|string|`YES`||The contract name for which to retrieve the order book
`limit`|integer|`NO`|`100` (max = 100)|The number of entries to return for bids and asks.

### **Response:**
name|type|example|description
------------ | ------------ | ------------ | ------------
`time`|long|`1550829103981`|Current timestamp (ms)
`bids`|list|(see below)|List of all bids, best bids first. See below for entry details.
`asks`|list|(see below)|List of all asks, best asks first. See below for entry details.

The fields `bids` and `asks` are lists of orderbook price level entries, sorted from best to worst.

name|type|example|description
------------ | ------------ | ------------ | ------------
`''`|float|`123.10`|price level
`''`|float|`300`|The total quantity of orders for this price level

### **Example:**

```js
{
  "time": 1555049455783,
  "bids": [
   ["78.82", "0.526"],//[Price, Quantity]
   ["77.24", "1.22"],
   ["76.65", "1.043"],
   ["76.58", "1.34"],
   ["75.67", "1.52"],
   ["75.12", "0.635"],
   ["75.02", "0.72"],
   ["75.01", "0.672"],
   ["73.73", "1.282"],
   ["73.58", "1.116"],
   ["73.45", "0.471"],
   ["73.44", "0.483"],
   ["72.32", "0.383"],
   ["72.26", "1.283"],
   ["72.11", "0.703"],
   ["70.61", "0.454"]],
   "asks": [
     ["122.96", "0.381"],//[Price, Quantity]
     ["144.46", "1"],
     ["155.55", "0.065"],
     ["160.16", "0.052"],
     ["200", "0.775"],
     ["249", "0.17"],
     ["250", "1"],
     ["300", "1"],
     ["400", "1"],
     ["499", "1"]]
   }

```

## `trades`

Retrieve the latest trades that have occurred for a specific contract.

### **Request Weight:**

1

### **Request URL:**
```
GET /exapi/quote/v1/contract/trades
```
### **Parameters：**
Parameter|type|required|default|description
------------ | ------------ | ------------ | ------------ | ------
`symbol`|string|`YES`||The name of the contract.
`limit`|integer|`NO` (clamped to max 60)|`60`|The number of trades returned

### **Response:**

name|type|example|description
------------ | ------------ | ------------ | ------------
`price`|float|`0.055`|The price of the trade
`time`|long|`1537797044116`|Current timestamp (ms)
`qty`|float|`5`|The quantity traded
`isBuyerMaker`|string|`true`|Maker or taker of the trade. `true`= maker, `false` = taker


### **Example:**
```js
[
  {
    "price": "1.21",
    "time": 1555034474064,
    "qty": "0.725",
    "isBuyerMaker": False
  },...
]
```

## `klines`

Retrieves the kline information (open, high, trade volume, etc.) for a specific contract.

### **Request Weight:**

1

### **Request URL:**
```
GET /exapi/quote/v1/contract/klines
```

### **Parameters：**
| Parameter|type|required|default|description |
| ------------ | ------------ | ------------ | ------------ | ---- |
|`symbol`|string|`YES`||Name of the contract.|
|`interval`|string|`YES`||Interval of the kline. Possible values include: `1m`,`5m`,`15m`,`30m`,`1h`,`1d`,`1w`,`1M`|
|`limit`|integer|`NO`|`1000`|Number of entries returned. Max is capped at 1000.|
|`to`|integer|`NO`||timestamp of the last datapoint|

### **Response:**
name|type|example|description
------------ | ------------ | ------------ | ------------
`''`|long|`1538728740000`|Open Time
`''`|float|`36.00000`|Open
`''`|float|`36.00000`|High
`''`|float|`36.00000`|Low
`''`|float|`36.00000`|Close
`''`|float|`148976.11427815`|Trade volume amount
`''`|long|`1538728740000`|Close time
`''`|float|`2434.19055334`|Quote asset volume
`''`|integer|`308`|Number of trades
`''`|float|`1756.87402397`|Taker buy base asset volume
`''`|float|`28.46694368`|Taker buy quote asset volume


### **Example:**
```js
[
  [
    1538728740000, //'opentime'
    "36.000000000000000000", //'open'
    "36.000000000000000000", //'high'
    "36.000000000000000000", //'low':
    "36.000000000000000000", //'close'
    "148976.11427815",  // Volume
    1499644799999,      // Close time
    "2434.19055334",    // Quote asset volume
    308,                // Number of trades
    "1756.87402397",    // Taker buy base asset volume
    "28.46694368"       // Taker buy quote asset volume
  ],...
]
```
`base asset` means the quantity is expressed as the amount of contracts that were received by the buyer.

`quote asset` means the amount of tokens paid to acquire the contracts.


# Private Endpoints

## `Signature`
signature look up ：[this](signature.md)

## `Place Contract Order`

This endpoint requires signed access.

### **Request Weight:**

1

### **Request URL:**
```bash
POST /exapi/contract/v1/order
```

### **Parameters:**
Name | Type | Required | Default | Description
------------ |-------------|-------------| ------------ | ------------
`symbol_id` | string | `YES` | | Contract name.
`client_order_id`| string/long | `YES` | | User-defined order ID.
`side`| string | `YES` | | Order direction, types are `BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, `SELL_CLOSE`.
`type`| string | `YES` | | Order type, supports `LIMIT` and `STOP`.
`price`| float | `NO`. (`LIMIT` & `INPUT` orders **required**) | | Order price.
`price_type`| string | `NO` | `INPUT` | Price type, supports `INPUT`, `OPPONENT`, `QUEUE`, `OVER`, `MARKET_PRICE`.
`quantity`| float | `YES` | | Order contract quantity.
`leverage`| float | `YES`. (Not required for \*\_CLOSE orders) | | Order leverage.
`time_in_force`| string | `NO` | `GTC` | Time in Force for `LIMIT` orders, supports `GTC`, `FOK`, `IOC`, `LIMIT_MAKER`.
`is_long`| number | `YES` | `GTC` | 0 - Short position; 1 - Long position;
`position_index` | number | `NO` | 0 | Position index, required for closing positions in sub-accounts; otherwise, not required or 0.

**Note:** For **market orders**, set `orderType` to **`LIMIT`** and `priceType` to **`MARKET_PRICE`**.

You can get contract price and quantity precision configuration information from the `brokerInfo` endpoint.

Note: If your balance does not meet the required margin (initial margin + opening fee + closing fee), an "*insufficient balance*" error will be returned.

For detailed explanations of *price types* and *order types*, see the end of the document.

### **Response:**
Name | Type | Example | Description
------------ |----------|----------------------| ------------
`time` | string | `1570759718825` | Timestamp when the order was created
`orderId`| string | `469961015902208000` | Order ID
`accountId`| string | `469961015902208000` | Account ID
`clientOrderId`| string | `213443` | User-defined order ID
`symbolId`| string | `BTC-SWAP-USDT` | Contract pair ID
`symbolName`| string | `BTC-SWAP-USDT` | Contract pair name
`baseTokenId`| string | `BTC-SWAP-USDT` | Base token ID
`baseTokenName`| string | `BTC-SWAP-USDT` | Base token name
`quoteTokenId`| string | `USDT` | Quote token ID
`quoteTokenName`| string | `USDT` | Quote token name
`price`| string | `8200` | Order price
`origQty`| string | `100` | Order quantity
`executedQty`| string | `100` | Executed quantity
`executedAmount`| string | `100` | Executed amount
`avgPrice`| string | `4754.24` | Average transaction price
`type`| string | `STOP` | Order type, supports `LIMIT` and `STOP`
`side`| string | `BUY` | Order direction (`BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, `SELL_CLOSE`)
`fees`| | | Order fees
`status`| string | `NEW` | Order status (`NEW`, `PARTIALLY_FILLED`, `FILLED`, `CANCELED`, `REJECTED`)
`noExecutedQty`| string | `0` | Unexecuted quantity
`exchangeId`| string | `301` | Exchange ID
`margin`| string | `99` | Margin
`leverage`| string | `4` | Order leverage
`isClose`| boolean | `false` | `false` - Open position; `true` - Close position
`priceType`| string | `INPUT` | Price type (`INPUT`, `OPPONENT`, `QUEUE`, `OVER`, `MARKET`)
`isLiquidationOrder`| boolean | `false` | Is liquidation order; `true` - Yes; `false` - No
`liquidationType`| string | `NO_LIQ` | Liquidation order type: `IOC` for forced liquidation, `ADL` for auto-deleveraging
`liquidationPrice`| string | `0` | Forced liquidation price
`closePnl`| string | `12122` | Close position profit and loss
`updateTime`| string | `1551062936784` | Last update timestamp of the order
`timeInForce`| string | `GTC` | Time in Force type (`GTC`, `FOK`, `IOC`, `LIMIT_MAKER`)

### **Example:**
```json
{
  "time": "1736321774823",
  "orderId": "1858796203149438464",
  "accountId": "1611498967211781634",
  "clientOrderId": "cc881d1d9fc28aca6ddeb03563f668555dce64584fbccc425b3664075f987d5a",
  "symbolId": "BTC-SWAP-USDT",
  "symbolName": "BTC-SWAP-USDT",
  "baseTokenId": "BTC-SWAP-USDT",
  "baseTokenName": "BTC-SWAP-USDT",
  "quoteTokenId": "USDT",
  "quoteTokenName": "USDT",
  "price": "96220",
  "origQty": "100",
  "executedQty": "0",
  "executedAmount": "0",
  "avgPrice": "0",
  "type": "LIMIT",
  "side": "BUY_OPEN",
  "fees": [],
  "status": "FILLED",
  "noExecutedQty": "100",
  "amount": "9622000",
  "exchangeId": "301",
  "margin": "962.2",
  "leverage": "1",
  "isClose": false,
  "priceType": "INPUT",
  "isLiquidationOrder": false,
  "liquidationType": "NO_LIQ",
  "liquidationPrice": "0",
  "closePnl": "0",
  "updateTime": "1736321774823",
  "timeInForce": "GTC"
}
```

## `Unfilled Futures Orders`

### **Request Weight:**

1

### **Request Url:**
```bash
GET /exapi/contract/v1/order/open_orders
```

### **Parameter:**

 Name        | Type      | Required | Default | Description                       
-----------|---------|------|----|--------------------------
| symbol_id | string  | No |    | Order ID                    
| type      | string  | No |    | Order type, supports `LIMIT` and `STOP`. |
| limit     | number  | No |    | Quantity                       |


### **Response:**

Name|Type|Example|Description
------------ | ------------ | ------------ | ------------
`time`   | string   | `1570759718825`      | Timestamp when the order was created
`orderId`| string   | `469961015902208000` | Order ID
`accountId`| string   | `469961015902208000` | Account ID
`clientOrderId`| string   | `213443`             | User-defined order ID
`symbolId`| string   | `BTC-SWAP-USDT`      | Contract pair ID
`symbolName`| string   | `BTC-SWAP-USDT`      | Contract pair name
`baseTokenId`| string   | `BTC-SWAP-USDT`      | Base token ID
`baseTokenName`| string   | `BTC-SWAP-USDT`      | Base token name
`quoteTokenId`| string   | `USDT`               | Quote token ID
`quoteTokenName`| string   | `USDT`               | Quote token name
`price`| string   | `8200`               | Order price
`origQty`| string   | `100`                | Order quantity
`executedQty`| string   | `100`                | Executed quantity
`executedAmount`| string   | `100`                | Executed amount
`avgPrice`| string   | `4754.24`            | Average transaction price
`type`| string   | `STOP`               | Order type, supports `LIMIT` and `STOP`
`side`| string   | `BUY`                | Order direction (`BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, `SELL_CLOSE`)
`fees`|          |                      | Order fees
`status`| string   | `NEW`                | Order status (`NEW`, `PARTIALLY_FILLED`, `FILLED`, `CANCELED`, `REJECTED`)
`noExecutedQty`| string   | `0`                  | Unexecuted quantity
`exchangeId`| string   | `301`                | Exchange ID
`margin`| string   | `99`                 | Margin
`leverage`| string   | `4`                  | Order leverage
`isClose`| boolean  | `false`              | `false` - Open position; `true` - Close position
`priceType`| string   | `INPUT`              | Price type (`INPUT`, `OPPONENT`, `QUEUE`, `OVER`, `MARKET`)
`isLiquidationOrder`| boolean | `false`              | Is liquidation order; `true` - Yes; `false` - No
`liquidationType`| string | `NO_LIQ`             | Liquidation order type: `IOC` for forced liquidation, `ADL` for auto-deleveraging
`liquidationPrice`| string | `0`                  | Forced liquidation price
`closePnl`| string | `12122`              | Close position profit and loss
`updateTime`|string| `1551062936784`      | Last update timestamp of the order
`timeInForce`|string| `GTC`                | Time in Force type (`GTC`, `FOK`, `IOC`, `LIMIT_MAKER`)

In the `fees` information group:

Name|Type|Example|Description
------------ | ------------ | ------------ | ------------
`feeToken`|string|`USDT`| Fee token type
`fee`|float|`0`| Actual fee

###  **Example:**

```json
[
  {
    "time": "1736325254825",
    "orderId": "1858825391386471936",
    "accountId": "1611498967211781634",
    "clientOrderId": "1736325253641",
    "symbolId": "BTC-SWAP-USDT",
    "symbolName": "BTC-SWAP-USDT",
    "baseTokenId": "BTC-SWAP-USDT",
    "baseTokenName": "BTC-SWAP-USDT",
    "quoteTokenId": "USDT",
    "quoteTokenName": "USDT",
    "price": "9595.6",
    "origQty": "111",
    "executedQty": "0",
    "executedAmount": "0",
    "avgPrice": "0",
    "type": "LIMIT",
    "side": "BUY_OPEN",
    "fees": [],
    "status": "NEW",
    "noExecutedQty": "111",
    "amount": "1065111.6",
    "exchangeId": "301",
    "margin": "53.2555",
    "leverage": "2",
    "isClose": false,
    "priceType": "INPUT",
    "isLiquidationOrder": false,
    "liquidationType": "NO_LIQ",
    "liquidationPrice": "0",
    "closePnl": "0",
    "updateTime": "1736325254917",
    "timeInForce": "GTC"
  }
]]
```

## `Contract Order Cancellation`

To cancel an order, you need to send either `orderId` or `clientOrderId`. This API endpoint requires your request to be signed.

### **Request Weight:**

1

### **Request Url:**
```bash
DELETE /exapi/contract/v1/order/cancel
```

### **Parameters:**

Name | Type | Required | Default | Description
------------ | ------------ | ------------ | ------------ | ------------
`orderId` | integer | `NO` | | Order ID
`clientOrderId` | string | `NO` | | User-defined order ID
`orderType` | string | `YES` | | Order type (`LIMIT` and `STOP`)

**Note:** Either `orderId` or `clientOrderId` **must be sent**

### **Response:**

Name | Type | Example | Description
------------ | ------------ | ------------ | ------------
`time` | long | `1570759718825` | Timestamp when the order was created
`updateTime` | long | `1551062936784` | Timestamp when the order was last updated
`orderId` | integer | `469961015902208000` | Order ID
`clientOrderId` | string | `213443` | User-defined order ID
`symbol` | string | `BTC-SWAP-USDT` | Contract name
`price` | float | `8200` | Order price
`leverage` | float | `4` | Order leverage
`origQty` | float | `1.01` | Order quantity
`executedQty` | float | `1.01` | Executed order quantity
`avgPrice` | float | `4754.24` | Average trade price
`marginLocked` | float | `200` | Margin locked by the order. This includes the actual required margin plus the fees for opening and closing the position.
`orderType` | string | `YES` | Order type (`LIMIT` and `STOP`)
`side` | string | `BUY` | Order direction (`BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, `SELL_CLOSE`)
`fees` | | | Order fees
`timeInForce` | string | `GTC` | Time in Force type (`GTC`, `FOK`, `IOC`, `LIMIT_MAKER`)
`status` | string | `NEW` | Order status (`NEW`, `PARTIALLY_FILLED`, `FILLED`, `CANCELED`, `REJECTED`). The order status returned by this endpoint will always be `CANCELED`
`priceType` | string | `INPUT` | Price type (`INPUT`, `OPPONENT`, `QUEUE`, `OVER`, `MARKET`)

In the `fees` information group:

Name | Type | Example | Description
------------ | ------------ | ------------ | ------------
`feeToken` | string | `USDT` | Fee token type
`fee` | float | `0` | Actual fee

### **Example:**

```js
{
  'time': '1570759718825',
  'updateTime': '0',
  'orderId': '469961015902208000',
  'clientOrderId': '6423344174',
  'symbol': 'BTC-SWAP-USDT',
  'price': '8200',
  'leverage': '12.08',
  'origQty': '5',
  'executedQty': '0',
  'avgPrice': '0',
  'marginLocked': '0',
  'orderType': 'LIMIT',
  'side': 'BUY_OPEN',
  'fees': [],
  'timeInForce': 'GTC',
  'status': 'CANCELED',
  'priceType': 'INPUT' // status will always be `CANCELED` for cancel request
}
```



## `Contract Order History`

Retrieves history of orders that have been partially or fully filled or canceled. This API endpoint requires your request to be signed.

### **Request Weight:**

1

### **Request Url:**
```bash
GET /exapi/contract/v1/historyOrders
```

### **Parameters:**

Name | Type | Required | Default | Description
------------ | ------------ | ------------ | ------------ | ------------
`symbol` | string | `NO` | | Symbol to return open orders for. If not sent, orders of all contracts will be returned.
`orderId` | integer | `NO` | | Order ID
`orderType` | string | `YES` | | The order type, possible types: `LIMIT`, `STOP`
`limit` | integer | `NO` | `20` | Number of entries to return.

If `orderId` is set, it will get orders < that `orderId`. Otherwise most recent orders are returned.

### **Response:**

Name | Type | Example | Description
------------ | ------------ | ------------ | ------------
`time` | long | `1570759718825` | Timestamp when the order was created
`updateTime` | long | `1551062936784` | Timestamp when the order was last updated
`orderId` | integer | `469961015902208000` | Order ID
`clientOrderId` | string | `213443` | User-defined order ID
`symbol` | string | `BTC-SWAP-USDT` | Contract name
`price` | float | `8200` | Order price
`leverage` | float | `4` | Order leverage
`origQty` | float | `1.01` | Order quantity
`executedQty` | float | `1.01` | Executed order quantity
`avgPrice` | float | `4754.24` | Average trade price
`marginLocked` | float | `200` | Margin locked by the order. This includes the actual required margin plus the fees for opening and closing the position.
`orderType` | string | `YES` | Order type (`LIMIT` and `STOP`)
`side` | string | `BUY` | Order direction (`BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, `SELL_CLOSE`)
`fees` | | | Order fees
`timeInForce` | string | `GTC` | Time in Force type (`GTC`, `FOK`, `IOC`, `LIMIT_MAKER`)
`status` | string | `NEW` | Order status (`NEW`, `PARTIALLY_FILLED`, `FILLED`, `CANCELED`, `REJECTED`)
`priceType` | string | `INPUT` | Price type (`INPUT`, `OPPONENT`, `QUEUE`, `OVER`, `MARKET`)

In the `fees` information group:

Name | Type | Example | Description
------------ | ------------ | ------------ | ------------
`feeToken` | string | `USDT` | Fee token type
`fee` | float | `0` | Actual fee

### **Example:**

```js
[
  {
    'time': '1570759718825',
    'updateTime': '0',
    'orderId': '469961015902208000',
    'clientOrderId': '6423344174',
    'symbol': 'BTC-SWAP-USDT',
    'price': '8200',
    'leverage': '12.08',
    'origQty': '5',
    'executedQty': '0',
    'avgPrice': '0',
    'marginLocked': '0',
    'orderType': 'LIMIT',
    'side': 'BUY_OPEN',
    'fees': [],
    'timeInForce': 'GTC',
    'status': 'CANCELED',
    'priceType': 'INPUT'
  },
  ...
]
```



## `Order Details`

Retrieve detailed information of a specific order.

### **Request Weight:**

1

### **Request Url:**
```bash
GET /exapi/contract/v1/getOrder
```

### **Parameters:**

Name | Type | Required | Default | Description
------------ | ------------ | ------------ | ------------ | ------------
`orderId` | integer | `NO` | | Order ID
`clientOrderId` | string | `NO` | | User-defined order ID
`orderType` | string | `YES` | | Order type (`LIMIT` and `STOP`)

**Note:** Either `orderId` or `clientOrderId` **must be sent**

### **Response:**

Name | Type | Example | Description
------------ | ------------ | ------------ | ------------
`time` | long | `1570759718825` | Timestamp when the order was created
`updateTime` | long | `1551062936784` | Timestamp when the order was last updated
`orderId` | integer | `469961015902208000` | Order ID
`clientOrderId` | string | `213443` | User-defined order ID
`symbol` | string | `BTC-SWAP-USDT` | Contract name
`price` | float | `8200` | Order price
`leverage` | float | `4` | Order leverage
`origQty` | float | `1.01` | Order quantity
`executedQty` | float | `1.01` | Executed order quantity
`avgPrice` | float | `4754.24` | Average trade price
`marginLocked` | float | `200` | Margin locked by the order. This includes the actual required margin plus the fees for opening and closing the position.
`orderType` | string | `YES` | Order type (`LIMIT` and `STOP`)
`priceType` | string | `INPUT` | Price type (`INPUT`, `OPPONENT`, `QUEUE`, `OVER`, `MARKET`)
`side` | string | `BUY` | Order direction (`BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, `SELL_CLOSE`)
`status` | string | `NEW` | Order status (`NEW`, `PARTIALLY_FILLED`, `FILLED`, `CANCELED`, `REJECTED`)
`timeInForce` | string | `GTC` | Time in Force type (`GTC`, `FOK`, `IOC`, `LIMIT_MAKER`)
`fees` | | | Order fees

In the `fees` information group:

Name | Type | Example | Description
------------ | ------------ | ------------ | ------------
`feeToken` | string | `USDT` | Fee token type
`fee` | float | `0` | Actual fee

### **Example:**

```js
{
  'time': '1570760254539',
  'updateTime': '0',
  'orderId': '469965509788581888',
  'clientOrderId': '1570760253946',
  'symbol': 'BTC-SWAP-USDT',
  'price': '8502.34',
  'leverage': '20',
  'origQty': '222',
  'executedQty': '0',
  'avgPrice': '0',
  'marginLocked': '0.00130552',
  'orderType': 'LIMIT',
  'side': 'BUY_OPEN',
  'fees': [],
  'timeInForce': 'GTC',
  'status': 'NEW',
  'priceType': 'INPUT'
}
```



## `Trade History`

Returns the trade history of the account. This API endpoint requires your request to be signed.

### **Request Weight:**

1

### **Request Url:**
```bash
GET /exapi/contract/v1/myTrades
```

### **Parameters:**

Name | Type | Required | Default | Description
------------ | ------------ | ------------ | ------------ | ------------
`symbol` | string | `NO` | | Contract name. If not sent, the trade history of all contracts will be returned.
`limit` | integer | `NO` | `20` | Return limit (maximum value is 1000)
`fromId` | integer | `NO` | | Start from TradeId (used to query trade orders)
`toId` | integer | `NO` | | End at TradeId (used to query trade orders)

### **Response:**

Name | Type | Example | Description
------------ | ------------ | ------------ | ------------
`time` | long | `1551062936784` | Timestamp when the order was created
`tradeId` | long | `49366` | Trade ID
`orderId` | long | `630491436` | Order ID
`matchOrderId` | long | `630491432` | Matched order ID
`symbolId` | string | `BTC-SWAP-USDT` | Contract name
`price` | float | `4765.29` | Trade price
`quantity` | float | `1.01` | Trade quantity
`feeTokenId` | string | `USDT` | Fee token type (Token name)
`fee` | | | Actual fee
`side` | string | `BUY` | Order direction (`BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, `SELL_CLOSE`)
`orderType` | string | `LIMIT` | Order type (`LIMIT`, `MARKET`)
`pnl` | float | `100.1` | Profit and loss of the trade

### **Example:**

```js
[
  {
    'time': '1570760582848',
    'tradeId': '469968263995080704',
    'orderId': '469968263793737728',
    'matchOrderId': '436002617267469062',
    'accountId': '456552319339779840',
    'symbolId': 'BTC-SWAP-USDT',
    'price': '8531.17',
    'quantity': '100',
    'feeTokenId': 'TBTC',
    'fee': '0.00000586',
    'type': 'LIMIT',
    'side': 'BUY_OPEN',
    'pnl': '100.1'
  },
  ...
]
```



## `Current Positions`

Returns the current position information. This API requires your request to be signed.

### **Request Weight:**

1

### **Request Url:**
```bash
GET /exapi/contract/v1/positions
```

### **Parameters:**

Name | Type | Required | Default | Description
------------ | ------------ | ------------ | ------------ | ------------
`symbol` | string | `YES` | | Contract name. If not sent, position information for all contracts will be returned.
`side` | string | `YES` | | Position direction, `LONG` (long position) or `SHORT` (short position). If not sent, position information for both directions will be returned.

### **Response:**

Name | Type | Example | Description
------------ | ------------ | ------------ | ------------
`accountId` | string | `1611498967211781634` | Account ID
`positionId` | string | `1858796203384319488` | Position ID
`symbolId` | string | `BTC-SWAP-USDT` | Contract pair ID
`leverage` | string | `0.99` | Current leverage of the position
`total` | string | `100` | Total balance
`positionValues` | string | `956.368` | Position value (USDT)
`margin` | string | `961.4842` | Position margin
`minMargin` | string | `953.8498` | Maximum margin reduction (USDT)
`orderMargin` | string | `0` | Order margin
`avgPrice` | string | `95430` | Average opening price
`liquidationPrice` | string | `0` | Estimated liquidation price
`marginRate` | string | `1.0076` | Current margin rate of the position
`indices` | string | `95647.4` | Settlement index
`futureRefPrice` | string | `95647.4` | Contract price (returns mark or index price based on quote-server configuration)
`available` | string | `100` | Available quantity for closing (contracts)
`coinAvailable` | string | `0` | Available margin
`isLong` | string | `1` | Position direction: 1=long, 0=short
`realisedPnl` | string | `-0.7157` | Realized profit and loss
`unrealisedPnl` | string | `2.1743` | Unrealized profit and loss
`profitRate` | string | `0.0022` | Profit rate of the current position
`isCross` | integer | `0` | Position mode: 1=cross margin, 0=isolated margin
`positionIndex` | string | `1858796203241713152` | Position index
`settledUpnl` | string | `0` | Settled unrealized profit and loss
`openValue` | string | `954.3` | Cumulative opening value

### **Example:**

```js
[
  {
    "accountId": "1611498967211781634",
    "positionId": "1858796203384319488",
    "symbolId": "BTC-SWAP-USDT",
    "leverage": "0.99",
    "total": "100",
    "positionValues": "956.368",
    "margin": "961.4842",
    "minMargin": "953.8498",
    "orderMargin": "0",
    "avgPrice": "95430",
    "liquidationPrice": "0",
    "marginRate": "1.0076",
    "indices": "95647.4",
    "futureRefPrice": "95647.4",
    "available": "100",
    "coinAvailable": "0",
    "isLong": "1",
    "realisedPnl": "-0.7157",
    "unrealisedPnl": "2.1743",
    "profitRate": "0.0022",
    "isCross": 0,
    "positionIndex": "1858796203241713152",
    "settledUpnl": "0",
    "openValue": "954.3"
  }
]
```

## `Contract Account Balance`

Returns the contract account balance. This endpoint requires your request to be signed.

### **Request Weight:**
1

### **Request Url:**
```bash
GET /exapi/contract/v1/account
```

### **Parameters:**
None

### **Response:**

Name | Type | Example | Description
------------ | ------------ | ------------ | ------------
`tokenId` | string | ETH | Token ID
`tokenName` | string | ETH | Token name
`total` | string | 10000 | Total balance
`availableMargin` | string | 10000 | Available margin
`positionMargin` | string | 0 | Position margin
`orderMargin` | string | 0 | Order margin (locked for orders)
`experienceBalance` | string | 0 | Experience balance
`deductionBalance` | string | 0 | Deduction balance
`grantGoldBalance` | string | 0 | Grant gold balance
`canTransfer` | string | 10000 | Transferable balance
`totalEquity` | string | 10000 | Total equity
`unrealisedPnl` | string | 0 | Unrealized profit and loss
`walletBalance` | string | 10000 | Wallet balance
`todayProfit` | string | 0.00 | Today's profit and loss

### **Example:**

```js
{
    "ETH": {
        "tokenId": "ETH",
        "tokenName": "ETH",
        "total": "10000",
        "availableMargin": "10000",
        "positionMargin": "0",
        "orderMargin": "0",
        "experienceBalance": "0",
        "deductionBalance": "0",
        "grantGoldBalance": "0",
        "canTransfer": "10000",
        "totalEquity": "10000",
        "unrealisedPnl": "0",
        "walletBalance": "10000",
        "todayProfit": "0.00"
    },
    "USDT": {
        "tokenId": "USDT",
        "tokenName": "USDT",
        "total": "13955.91957457",
        "availableMargin": "13955.91957457",
        "positionMargin": "0",
        "orderMargin": "0",
        "experienceBalance": "0",
        "deductionBalance": "0",
        "grantGoldBalance": "0",
        "canTransfer": "13955.91957457",
        "totalEquity": "13955.91957457",
        "unrealisedPnl": "0",
        "walletBalance": "13955.91957457",
        "todayProfit": "0.00"
    }
}
```

## `Modify Margin`

Modify the margin of a contract position. This endpoint requires your request to be signed.

### **Request Weight:**
1

### **Request Url:**
```bash
POST /exapi/contract/v1/modifyMargin
```

### **Parameters:**

Name | Type | Required | Default | Description
------------ | ------------ | ------------ | ------------ | ------------
`symbol` | string | `YES` | | Contract name.
`side` | string | `YES` | | Position direction, `LONG` (long position) or `SHORT` (short position).
`amount` | float | `YES` | | Amount to increase (positive value) or decrease (negative value) the margin. Note that this amount refers to the contract's underlying pricing asset (i.e., the asset in which the contract is settled).

### **Response:**

Name | Type | Example | Description
------------ | ------------ | ------------ | ------------
`symbol` | string | `BTC-SWAP-USDT` | Contract name
`margin` | float | `12.3` | Updated position margin
`timestamp` | long | `1541161088303` | Update timestamp

### **Example:**

```js
{
  'symbol': 'BTC-SWAP-USDT',
  'margin': 15,
  'timestamp': 1541161088303
}
```

## `Modify Margin Mode`

Modify the margin mode.

### **Request Weight:**
1

### **Request Url:**
```bash
POST /exapi/contract/v1/set_position_mode
```

### **Parameters:**

Name | Type | Required | Default | Description
------------ | ------------ | ------------ | ------------ | ------------
`symbol_id` | string | `YES` | | Contract ID
`is_cross` | integer | `YES` | | 0 - Isolated margin; 1 - Cross margin

### **Response:**

Name | Type | Example | Description
------------ | ------------ | ------------ | ------------
`success` | boolean | true | true: Success; false: Failure

### **Example:**

```json
{
  "success": true
}
```

## `Query Margin Mode`

Query the margin mode.

### **Request Weight:**

1

### **Request URL:**

```bash
GET /exapi/contract/v1/get_position_mode
```

### **Parameters:**

Name | Type | Required | Default | Description
------------ | ------------ | ------------ | ------------ | ------------
`symbol_id` | string | `YES` | | Contract ID

### **Response:**

Name | Type | Required | Description
------------ | ------------ | ------------ | ------------
`isCross` | boolean | true | true: Cross margin; false: Isolated margin
`symbolId` | string | true | Contract ID
`switchFlag` | boolean | true | Whether modification is allowed

### **Example:**

```json
{
  "isCross": true,
  "symbolId": "BTC-SWAP-USDT",
  "switchFlag": true
}
```

## `Modify Merge Mode`

Modify the merge mode.

### **Request Weight:**

1

### **Request URL:**
```bash
POST /exapi/contract/v1/set_position_merge_mode
```

### **Parameters:**

Name | Type | Required | Description
------------ | ------------ | ------------ | ------------
`symbol_id` | string | `YES` | Contract ID
`merge_mode` | string | `NO` | Merge mode: 0 - Merge mode; 1 - Split mode;

### **Response:**

Name | Type | Required | Description
------------ | ------------ | ------------ | ------------
`success` | boolean | true | true: Success; false: Failure

### **Example:**

```json
{
  "success": true
}
```

## `Query Merge Mode`

Query the merge mode.

### **Request Weight:**

1

### **Request URL:**
```bash
GET /exapi/contract/v1/get_position_merge_mode
```

### **Parameters:**

Name | Type | Required | Description
------------ | ------------ | ------------ | ------------
`symbol_id` | string | `YES` | Contract ID

### **Response:**

Name | Type | Required | Description
------------ | ------------ | ------------ | ------------
`mergeMode` | integer | true | Merge mode: 0 - Merge mode; 1 - Split mode; 
`symbolId` | string | true | Contract ID

### **Example:**

```json
{
  "mergeMode": 1,
  "symbolId": "BTC-SWAP-USDT"
}
```

## `Set Leverage`

Set the leverage.

### **Request Weight:**

1

### **Request URL:**

```bash
POST /exapi/contract/v1/update_leverage_merge
```

### **Parameters:**

Name | Type | Required | Description
------------ | ------------ | ------------ | ------------
`symbol_id` | string | `YES` | Contract ID
`leverage` | integer | `YES` | Leverage
`is_long` | integer | `YES` | 1 for long position, 2 for short position

### **Response:**

Name | Type | Required | Description
------------ | ------------ | ------------ | ------------
`success` | boolean | true | true: Success; false: Failure

### **Example:**

```json
{
  "success": true
}
```

## `Query Leverage`

Query the leverage.

### **Request Weight:**

1

### **Request URL:**
```bash
GET /exapi/contract/v1/query_leverage_merge
```

### **Parameters:**

Name | Type | Required | Description
------------ | ------------ | ------------ | ------------
`symbol_id` | string | `YES` | Contract ID

### **Response:**

Name | Type | Required | Description
------------ | ------------ | ------------ | ------------
`accountId` | integer | true | Account ID
`isCross` | boolean | true | true: Cross margin; false: Isolated margin
`leverage` | integer | true | Leverage
`leverageLong` | integer | true | Long position leverage
`leverageShort` | integer | true | Short position leverage
`symbol_id` | string | true | Contract pair ID

### **Example:**

```json
{
  "accountId": 0,
  "isCross": true,
  "leverage": 0,
  "leverageLong": 0,
  "leverageShort": 0,
  "symbol_id": "string"
}
```


# Key parameter explanation:

## `side`

The side of the trade.

`BUY_OPEN`: open a long position.

`SELL_CLOSE`: close a long position.

`SELL_OPEN`: open a short position.

`BUY_CLOSE`: close a short position.

## `priceType`

Price types.

`INPUT`: The system will use the price you entered exactly to fill the orders.

`OPPONENT`: Orders will be filled using the opposite side's best quote.

For example, if you are opening 10 contracts long, the best buy price is 10 and the best sell price is 11. You will send an order buying 10 contracts at 11. If the order, is not fully filled, the rest will be left on the orderbook.

`QUEUE`: Order will be send using the same side's best quote.

For example, if you are opening 10 contracts long, the best buy price is 10 and the best sell price is 11. You will be send an order buying 10 contracts at 10.

`OVER`: The price will be the best opposite's quote + overPrice(not a fixed value).

For example, if you are opening 10 contracts long, the best buy price is 10 and the best sell price is 11, you set the overPrice at 3. You will be send an order buying 10 contracts at (11+3)=14.

`MARKET`: The price will be newest price * (1 ± 5%).

For example, if you are opening 10 contracts long, the latest price is 10. Then you will be sending out an order buying 10 contracts at (10 * 1.05)=10.5.

## `timeInForce`

Time in force.

`GTC`: Good till canceled. Meaning the order will stand unless otherwise cancelled.

`IOC`: Immediate or cancel. Meaning the order will be cancelled if not executed immediately. Recommended if you want to fill the entire order immediately.

`FOK`: Fill or kill. Meaning the order will be canceled if not immediately filled. Recommended if you want to fill as much as possible, but not necessarily all of, the order immediately.

`LIMIT_MAKER`: Order will be cancelled if executed immediately.

## `orderType`

Order type.

`LIMIT`: Orders to be executed given a specified price or better.


`STOP`: Order that will be triggered once it reaches the `triggerPrice`.
