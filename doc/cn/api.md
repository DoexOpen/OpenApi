# 文档目录

- [Broker 合约公共端点](#broker-合约公共端点)
    - [Broker API URL](#broker-api-url)
    - [合约信息](#合约信息)
    - [获取指数](#获取指数)
    - [获取订单簿](#获取订单簿)
    - [获取最近成交](#获取最近成交)
    - [获取K线信息](#获取k线信息)

- [交易接口 (需签名认证)](#交易接口-需签名认证)
    - [凭证携带](#凭证携带)
    - [合约下单](#合约下单)
    - [期货未成交委托单](#期货未成交委托单)
    - [合约撤单](#合约撤单)
    - [合约历史委托](#合约历史委托)
    - [订单详情](#订单详情)
    - [历史成交](#历史成交)
    - [当前持仓](#当前持仓)
    - [合约账户余额](#合约账户余额)
    - [修改保证金](#修改保证金)
    - [修改保证金模式](#修改保证金模式)
    - [查询保证金模式](#查询保证金模式)
    - [修改合并模式](#修改合并模式)
    - [查询合并模式](#查询合并模式)
    - [设置杠杆](#设置杠杆)
    - [查询杠杆](#查询杠杆)

- [关键参数解释说明](#关键参数解释说明)
    - [`side`](#side)
    - [`priceType`](#pricetype)
    - [`timeInForce`](#timeinforce)
    - [`orderType`](#ordertype)


# Broker 合约公共端点

## Broker API URL

Broker Open API的地址请见[这里](../endpoint.md)

## 合约信息

获取当前broker的交易规则和合约symbol的信息（精度单位等信息），包括合约的风险限额和乘数等信息。

### **Request Weight:**

0

### **Request Url:**

```bash
GET /exapi/v1/brokerInfo
```

### **Parameters：**

无

### **Response:**
名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`timezone`|string|`UTC`|时间戳的时区。
`serverTime`|long|`1554887652929`|返回现在的服务器时间戳（毫秒级）

在`rateLimits`信息组里:
下单api的请求限制将会被展示。

名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`rateLimitType`|string|`ORDERS`|速度限制类型
`interval`|string|`SECOND`|速度限制区间
`limit`|string|`1500`|速度限制区间价值

在`contracts`信息组里:
所有当前券商正在交易的合约的信息将会被返回：

名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`symbol`|string|`BTC-SWAP-USDT`|合约名称
`status`|string|`TRADING`|合约状态
`baseAsset`|string|`BTC-SWAP-USDT`|基础资产。对于合约来说，合约本身就是基础资产。
`baseAssetPrecision`|float|`0.001`|基础资产（合约数量）的精度
`quoteAsset`|string|`USDT`|定价资产。对于合约来说，这个是合约是以什么来结算的。
`quoteAssetPrecision`|float|`0.001`|定价资产（合约价格）的精度。
`inverse`|bool|`true`|合约是否为反向合约（true=是反向合约，false=是正向合约）。
`index`|string|`BTCUSDT`|标的指数的名称。标的指数实时价格可在`index`端点访问得到。比如`BTC-SWAP-USDT`使用`BTCUSDT`为标的指数，那么可以在`index`端点寻找`BTCUSDT`的实时价格。
`contractMultiplier`|string|`true`|合约的乘数。
`icebergAllowed`|string|`false`|是否支持冰山订单。


在 `contracts`的`filters`信息组里:

名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`filterType`|string|`PRICE_FILTER`|过滤器类型。
`minPrice`|float|`0.001`|允许的最小价格。
`maxPrice`|float|`100000.00000000`|允许的最大价格。
`tickSize`|float|`0.001`|合约价格的精度。
`minQty`|float|`0.01`|允许的最小数量。
`maxQty`|float|`100000.00000000`|允许的最大数量。
`stepSize`|float|`0.001`|合约数量的精度。
`minNotional`|float|`1`|最小交易额限制(数量 * 价格)

在`riskLimits`信息组里:

名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`quantity`|float|`100`|仓位小于此数且大于前一个档位的`quantity`的需要参照以下要求。
`initialMargin`|float|`0.1`|初始保证金率。
`maintMargin`|float|`0.03`|最小维持保证金率。

### **Example:**
```js
{
  "timezone":"UTC",
  "serverTime":"1570701444309",
  "brokerFilters":[],
  "rateLimits":[
    {
      "rateLimitType":"ORDERS",
      "interval":"SECOND",
      "limit":20
    },
    {"rateLimitType":"ORDERS",
    "interval":"DAY",
    "limit":350000
  },{
    "rateLimitType":"REQUEST_WEIGHT",
    "interval":"MINUTE",
    "limit":1500
  }],
  "contracts":[
    {
      "filters":[
        {"minPrice":"0.01",
        "maxPrice":"100000.00000000",
        "tickSize":"0.01",
        "filterType":"PRICE_FILTER"
      },
      {
        "minQty":"1",
        "maxQty":"100000.00000000",
        "stepSize":"1",
        "filterType":"LOT_SIZE"
      },{
        "minNotional":"0.000001",
        "filterType":"MIN_NOTIONAL"
      }],
      "exchangeId":"301",
      "symbol":"BTC-SWAP-USDT",
      "symbolName":"BTC-SWAP-USDT",
      "status":"TRADING",
      "baseAsset":"BTC-SWAP-USDT",
      "baseAssetPrecision":"1",
      "quoteAsset":"USDT",
      "quoteAssetPrecision":"0.01",
      "icebergAllowed":false,
      "inverse":true,
      "index":"BTCUSDT",
      "marginToken":"TBTC",
      "marginPrecision":"0.00000001",
      "contractMultiplier":"1.0",
      "riskLimits":[
        {
          "riskLimitId":"200000001",
          "quantity":"1000000.0",
          "initialMargin":"0.01",
          "maintMargin":"0.005"
        },
        {
          "riskLimitId":"200000002",
          "quantity":"2000000.0",
          "initialMargin":"0.02",
          "maintMargin":"0.01"
        },
        {
          "riskLimitId":"200000003",
          "quantity":"3000000.0",
          "initialMargin":"0.03",
          "maintMargin":"0.015"
        },
        {
          "riskLimitId":"200000004",
          "quantity":"4000000.0",
          "initialMargin":"0.04",
          "maintMargin":"0.02"
        }
      ]
    }
  ]
}
```

## `获取指数`

标的指数价格

### **Request Weight:**
0

### **Request Url:**
```bash
GET /exapi/quote/v1/contract/index
```
### **Parameters：**
名称|类型|是否强制|默认|描述
------------ | ------------ | ------------ | ------------ | ----
symbol|string|`NO`||标的指数名称。如果这个没有发送，所有标的指数的价格都会被返回。

### **Response:**
名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`index`|float|`8342.73`|标的指数的价格。
`EDP`|float|`8432.32`|标的指数的EDP(预估交割价，过去10分钟指数价格的平均值）。

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

## `获取订单簿`

获取当前订单簿的数据。

### **Request Weight:**

根据数量会不一样，请求数量越多，重量越大:

数量|请求重量
------------ | ------------
5, 10, 20, 50, 100|1
500|5
1000|10

### **Request Url:**
```
GET /exapi/quote/v1/contract/depth
```

### **Parameters:**
名称|类型|是否强制|默认|描述
------------ | ------------ | ------------ | ------------ | -----
`symbol`|string|`YES`||用来获取订单簿的合约名称。
`limit`|integer|`NO`|`100` (max = 100)|返回`bids`和`asks`的数量

### **Response:**
名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`time`|long|`1550829103981`|当前时间（Unix Timestamp，毫秒ms）
`bids`|list|(如下)|所有bid的价格和数量信息，最优bid价格由上到下排列。
`asks`|list|(如下)|所有ask的价格和数量信息，最优ask价格由上到下排列。

`bids`和`asks`所对应的信息组代表了订单簿的所有价格以及价格对应数量的信息，由最优价格从上到下排列。

名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`''`|float|`123.10`|价格
`''`|float|`300`|当前价格对应的数量

### **Example:**

```js
{
  "time": 1555049455783,
  "bids": [
   ["78.82", "0.526"],//[价格, 数量]
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
     ["122.96", "0.381"],//[价格, 数量]
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

## `获取最近成交`

获取某个合约最近成交订单的信息。

### **Request Weight:**

1

### **Request URL:**
```
GET /exapi/quote/v1/contract/trades
```
### **Parameters：**
名称|类型|是否强制|默认|描述
------------ | ------------ | ------------ | ------------ | ------
`symbol`|string|`YES`||合约名称
`limit`|integer|`NO` (最大值为60)|`60`|返回成交订单的数量

### **Response:**

名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`price`|float|`0.055`|交易价格
`time`|long|`1537797044116`|当前Unix时间戳，毫秒(ms)
`qty`|float|`5`|数量（张数）
`isBuyerMaker`|string|`true`|卖方还是买方。`true`=卖方，`false`=买方


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

## `获取k线信息`

获取某个合约的K线信息（高，低，开，收，交易量...)

### **Request Weight:**

1

### **Request URL:**
```
GET /exapi/quote/v1/contract/klines
```

### **Parameters：**
 名称|类型|是否强制|默认|描述 
 ------------ | ------------ | ------------ | ------------ | ---- 
`symbol`|string|`YES`||合约名称
`interval`|string|`YES`||K线图区间。可识别发送的值为：  `1m`,`5m`,`15m`,`30m`,`1h`,`1d`,`1w`,`1M`（`m`=分钟，`h`=小时,`d`=天，`w`=星期，`M`=月）
`limit`|integer|`NO`|`1000`|返回值的数量，最大值为1000
`to`|integer|`NO`||最后一个数据点的时间戳

### **Response:**
名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`''`|long|`1538728740000`|开始时间戳，毫秒（ms）
`''`|float|`36.00000'`|开盘价
`''`|float|`36.00000`|最高价
`''`|float|`36.00000`|最低价
`''`|float|`36.00000`|收盘价
`''`|float|`148976.11427815`|合约交易金额
`''`|long|`1538728740000`|停止时间戳，毫秒（ms）
`''`|float|`2434.19055334`|交易数量（张数）
`''`|integer|`308`|已成交数量（张数）
`''`|float|`1756.87402397`|买方购买金额
`''`|float|`28.46694368`|买方购买数量（张数）


### **Example:**
```js
[
  [
    1538728740000, //'开盘时间'
    '36.000000000000000000', //'开盘价'
    '36.000000000000000000', //'最高价'
    '36.000000000000000000', //'最低价':
    '36.000000000000000000', //'收盘价'
    '148976.11427815',  // 合约交易金额
    1499644799999,      // 收盘时间
    '2434.19055334',    // 交易数量（张数）
    308,                // 已成交数量（张数）
    '1756.87402397',    // 买方购买金额
    '28.46694368'       // 买方购买数量（张数）
  ],...
]
```
`base asset` 代表买方收到了该数量的合约

`quote asset` 代表该数量的token用来获得合约

# 交易接口 (需签名认证)
## 凭证携带
参见：[接口签名](signature.md)

## `合约下单`

合约下单，这个端点需要签名访问。

### **Request Weight:**

1

### **Request URL:**
```bash
POST /exapi/contract/v1/order
```

### **Parameters：**
名称| 类型          | 是否强制        |默认|描述
------------ |-------------|-------------| ------------ | ------------
`symbol_id`| string      | `YES`       ||合约名称。
`client_order_id`| string/long | `YES`       ||订单的ID，用户自己定义。
`side`| string      | `YES`       ||下单方向，方向类型为`BUY_OPEN`、`SELL_OPEN`、`BUY_CLOSE`、`SELL_CLOSE`。
`type`| string      | `YES`       ||订单类型，支持订单类型为 `LIMIT`和`STOP`。
`price`| float       | `NO`. (`LIMIT`&`INPUT`)订单 **强制需要** ||订单价格。
`price_type`| string      | `NO`|`INPUT`|价格类型，支持的价格类型为`INPUT`、`OPPONENT`、`QUEUE`、`OVER`、`MARKET_PRICE`。
`quantity`| float       | `YES`       ||订单的合约数量。
`leverage`| float       | `YES`.（\*\_CLOSE平仓单**不强制**） ||订单的杠杆。
`time_in_force`| string      | `NO`        |`GTC`|`LIMIT`订单的时间指令（Time in Force），目前支持的类型为`GTC`、`FOK`、`IOC`、`LIMIT_MAKER`。
`is_long`| number      | `YES`        |`GTC`|0-空仓； 1-多仓；
`position_index` | number | `NO`        | 0 | 仓位索引,  分仓下平仓必传；其他情况不传或传0

**注意：** 对于 **市价订单**, 你需要设置`orderType`为 **`LIMIT`** **并且设置** `priceType` 为 **`MARKET_PRICE`**.

你可以在`brokerInfo`端点获取合约价格，数量的精度配置信息。

注意：如果你的余额没有达到需要保证金的要求（初始保证金+开仓手续费+平仓手续费），将会有"*insufficient balance*"（余额不足）的错误返回。

对于 *价格类型* 和 *订单类型* 的详细解释，请参见文末。

### **Response:**
名称| 类型       | 例子                   |描述
------------ |----------|----------------------| ------------
`time`   | string   | `1570759718825`      |订单生成时的时间戳
`orderId`| string   | `469961015902208000` |订单ID
`accountId`| string   | `469961015902208000` |订单ID
`clientOrderId`| string   | `213443`             |用户定义的订单ID
`symbolId`| string   | `BTC-SWAP-USDT`      | 合约币对id
`symbolName`| string   | `BTC-SWAP-USDT`      | 合约币对名称
`baseTokenId`| string   | `BTC-SWAP-USDT`      | 币种id
`baseTokenName`| string   | `BTC-SWAP-USDT`      | 币种名
`quoteTokenId`| string   | `USDT`               | 代币币种id
`quoteTokenName`| string   | `USDT`               | 代币币种名称
`price`| string   | `8200`               |订单价格
`origQty`| string   | `100`                |订单数量
`executedQty`| string   | `100`                |订单已执行数量
`executedAmount`| string   | `100`                |订单已执行金额
`avgPrice`| string   | `4754.24`            |平均交易价格
`type`| string   | `STOP`               |订单类型，支持订单类型为 `LIMIT`和`STOP`
`side`| string   | `BUY`                |订单方向（`BUY_OPEN`、`SELL_OPEN`、`BUY_CLOSE`、`SELL_CLOSE`）
`fees`|          |                      |订单的手续费
`status`| string   | `NEW`                |订单状态（`NEW`、`PARTIALLY_FILLED`、`FILLED`、`CANCELED`、`REJECTED`）
`noExecutedQty`| string   | `0`                  |订单未执行数量
`exchangeId`| string   | `301`                |交易所id
`margin`| string   | `99`                 | 保证金
`leverage`| string   | `4`                  |订单杠杆
`isClose`| boolean  | `false`              |false- 开仓； true-平仓
`priceType`| string   | `INPUT`              |价格类型（`INPUT`、`OPPONENT`、`QUEUE`、`OVER`、`MARKET`）
`isLiquidationOrder`| booolean | `false`              |是否为平仓订单;  true-是； false-不是
`liquidationType`| string | `NO_LIQ`             |爆仓单类型： IOC爆仓时候的强平单  ADL爆仓时候的减仓单
`liquidationPrice`| string | `0`                  |强制平仓价格
`closePnl`| string | `12122`              |平仓损益
`updateTime`|string| `1551062936784`      |订单上次更新的时间戳
`timeInForce`|string| `GTC`                |时效单（Time in Force)类型(`GTC`、`FOK`、`IOC`、`LIMIT_MAKER`)

### **Example:**
```js
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

## `期货未成交委托单`


### **Request Weight:**

1

### **Request Url:**
```bash
GET /exapi/contract/v1/order/open_orders
```

### **Parameter:**

 名称        | 类型      | 是否强制 | 默认 | 描述                       
-----------|---------|------|----|--------------------------
| symbol_id | string  | 否 |    | 订单的ID                    |
| type      | string  | 否 |    | 订单类型，支持订单类型为 LIMIT和STOP。 |
| limit     | number  | 否 |    | 数量                       |


### **Response:**

名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`time`   | string   | `1570759718825`      |订单生成时的时间戳
`orderId`| string   | `469961015902208000` |订单ID
`accountId`| string   | `469961015902208000` |订单ID
`clientOrderId`| string   | `213443`             |用户定义的订单ID
`symbolId`| string   | `BTC-SWAP-USDT`      | 合约币对id
`symbolName`| string   | `BTC-SWAP-USDT`      | 合约币对名称
`baseTokenId`| string   | `BTC-SWAP-USDT`      | 币种id
`baseTokenName`| string   | `BTC-SWAP-USDT`      | 币种名
`quoteTokenId`| string   | `USDT`               | 代币币种id
`quoteTokenName`| string   | `USDT`               | 代币币种名称
`price`| string   | `8200`               |订单价格
`origQty`| string   | `100`                |订单数量
`executedQty`| string   | `100`                |订单已执行数量
`executedAmount`| string   | `100`                |订单已执行金额
`avgPrice`| string   | `4754.24`            |平均交易价格
`type`| string   | `STOP`               |订单类型，支持订单类型为 `LIMIT`和`STOP`
`side`| string   | `BUY`                |订单方向（`BUY_OPEN`、`SELL_OPEN`、`BUY_CLOSE`、`SELL_CLOSE`）
`fees`|          |                      |订单的手续费
`status`| string   | `NEW`                |订单状态（`NEW`、`PARTIALLY_FILLED`、`FILLED`、`CANCELED`、`REJECTED`）
`noExecutedQty`| string   | `0`                  |订单未执行数量
`exchangeId`| string   | `301`                |交易所id
`margin`| string   | `99`                 | 保证金
`leverage`| string   | `4`                  |订单杠杆
`isClose`| boolean  | `false`              |false- 开仓； true-平仓
`priceType`| string   | `INPUT`              |价格类型（`INPUT`、`OPPONENT`、`QUEUE`、`OVER`、`MARKET`）
`isLiquidationOrder`| booolean | `false`              |是否为平仓订单;  true-是； false-不是
`liquidationType`| string | `NO_LIQ`             |爆仓单类型： IOC爆仓时候的强平单  ADL爆仓时候的减仓单
`liquidationPrice`| string | `0`                  |强制平仓价格
`closePnl`| string | `12122`              |平仓损益
`updateTime`|string| `1551062936784`      |订单上次更新的时间戳
`timeInForce`|string| `GTC`                |时效单（Time in Force)类型(`GTC`、`FOK`、`IOC`、`LIMIT_MAKER`)

在`fees`信息组里:

名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`feeToken`|string|`USDT`|手续费计价类型
`fee`|float|`0`|实际手续费

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
]
```

## `合约撤单`

取消某个订单，需要发送`orderId`或者`clientOrderId`其中之一。这个API端点需要你的请求签名。

### **Request Weight:**

1

### **Request Url:**
```bash
DELETE /exapi/contract/v1/order/cancel
```

### **Parameter:**

名称|类型|是否强制|默认|描述
------------ | ------------ | ------------ | ------------ | -----
`orderId`|integer|`NO`||订单的ID
`clientOrderId`|string|`NO`||用户定义的订单ID
`orderType`|string|`YES`||订单类型（`LIMIT`和`STOP`）

**注意：**` orderId` 或者 `clientOrderId` **必须发送其中之一**

### **Response:**

名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`time`|long|`1570759718825`|订单生成时的时间戳
`updateTime`|long|`1551062936784`|订单上次更新的时间戳
`orderId`|integer|`469961015902208000`|订单ID
`clientOrderId`|string|`213443`|用户定义的订单ID
`symbol`|string|`BTC-SWAP-USDT`|合约名称
`price`|float|`8200`|订单价格
`leverage`|float|`4`|订单杠杆
`origQty`|float|`1.01`|订单数量
`executedQty`|float|`1.01`|订单已执行数量
`avgPrice`|float|`4754.24`|平均交易价格
`marginLocked`|float|`200`|该订单锁定的保证金。这包括实际需要的保证金外加开仓和平仓所需的费用。
`orderType`|string|`YES`|订单类型（`LIMIT`和`STOP`）
`side`|string|`BUY`|订单方向（`BUY_OPEN`、`SELL_OPEN`、`BUY_CLOSE`、`SELL_CLOSE`）
`fees`|||订单的手续费
`timeInForce`|string|`GTC`|时效单（Time in Force)类型(`GTC`、`FOK`、`IOC`、`LIMIT_MAKER`)
`status`|string|`NEW`|订单状态（`NEW`、`PARTIALLY_FILLED`、`FILLED`、`CANCELED`、`REJECTED`）。该端点返回的订单状态都是`CANCELED`
`priceType`|string|`INPUT`|价格类型（`INPUT`、`OPPONENT`、`QUEUE`、`OVER`、`MARKET`）

在`fees`信息组里:

名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`feeToken`|string|`USDT`|手续费计价类型
`fee`|float|`0`|实际手续费

###  **Example:**

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
  'priceType': 'INPUT' //status will always be `CANCELED` for cancel request
}
```

## `合约历史委托`

Retrieves history of orders that have been partially or fully filled or canceled. This API endpoint requires your request to be signed.

### **Request Weight:**

1

### **Request Url:**
```bash
GET /exapi/contract/v1/historyOrders
```

### **Parameters:**
名称|类型|是否强制|默认|描述
------------ | ------------ | ------------ | ------------ | --------
`symbol`|string|`NO`||Symbol to return open orders for. If not sent, orders of all contracts will be returned.
`orderId`|integer|`NO`|| Order ID
`orderType`|string|`YES`||The order type, possible types: `LIMIT`, `STOP`
`limit`|integer|`NO`|`20`|Number of entries to return.

If `orderId` is set, it will get orders < that `orderId`. Otherwise most recent orders are returned.

### **Response:**
名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`time`|long|`1570759718825`|订单生成时的时间戳
`updateTime`|long|`1551062936784`|订单上次更新的时间戳
`orderId`|integer|`469961015902208000`|订单ID
`clientOrderId`|string|`213443`|用户定义的订单ID
`symbol`|string|`BTC-SWAP-USDT`|合约名称
`price`|float|`8200`|订单价格
`leverage`|float|`4`|订单杠杆
`origQty`|float|`1.01`|订单数量
`executedQty`|float|`1.01`|订单已执行数量
`avgPrice`|float|`4754.24`|平均交易价格
`marginLocked`|float|`200`|该订单锁定的保证金。这包括实际需要的保证金外加开仓和平仓所需的费用。
`orderType`|string|`YES`|订单类型（`LIMIT`和`STOP`）
`side`|string|`BUY`|订单方向（`BUY_OPEN`、`SELL_OPEN`、`BUY_CLOSE`、`SELL_CLOSE`）
`fees`|||订单的手续费
`timeInForce`|string|`GTC`|时效单（Time in Force)类型(`GTC`、`FOK`、`IOC`、`LIMIT_MAKER`)
`status`|string|`NEW`|订单状态（`NEW`、`PARTIALLY_FILLED`、`FILLED`、`CANCELED`、`REJECTED`）
`priceType`|string|`INPUT`|价格类型（`INPUT`、`OPPONENT`、`QUEUE`、`OVER`、`MARKET`）

在`fees`信息组里:

名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`feeToken`|string|`USDT`|手续费计价类型
`fee`|float|`0`|实际手续费

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
  },...
]
```

## `订单详情`

获取某个订单的详细信息

### **Request Weight:**

1

### **Request Url:**
```bash
GET /exapi/contract/v1/getOrder
```

### **Parameters:**
名称|类型|是否强制|默认|描述
------------ | ------------ | ------------ | ------------ | --------
`orderId`|integer|`NO`||订单ID
`clientOrderId`|string|`NO`||用户定义的订单ID
`orderType`|string|`YES`||订单类型（`LIMIT`和`STOP`）

**注意：**` orderId` 或者 `clientOrderId` **必须发送其中之一**


### **Response:**
名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`time`|long|`1570759718825`|订单生成时的时间戳
`updateTime`|long|`1551062936784`|订单上次更新的时间戳
`orderId`|integer|`469961015902208000`|订单ID
`clientOrderId`|string|`213443`|用户定义的订单ID
`symbol`|string|`BTC-SWAP-USDT`|合约名称
`price`|float|`8200`|订单价格
`leverage`|float|`4`|订单杠杆
`origQty`|float|`1.01`|订单数量
`executedQty`|float|`1.01`|订单已执行数量
`avgPrice`|float|`4754.24`|平均交易价格
`marginLocked`|float|`200`|该订单锁定的保证金。这包括实际需要的保证金外加开仓和平仓所需的费用。
`orderType`|string|`YES`|订单类型（`LIMIT`和`STOP`）
`priceType`|string|`INPUT`|价格类型（`INPUT`、`OPPONENT`、`QUEUE`、`OVER`、`MARKET`）
`side`|string|`BUY`|订单方向（`BUY_OPEN`、`SELL_OPEN`、`BUY_CLOSE`、`SELL_CLOSE`）
`status`|string|`NEW`|订单状态（`NEW`、`PARTIALLY_FILLED`、`FILLED`、`CANCELED`、`REJECTED`）
`timeInForce`|string|`GTC`|时效单（Time in Force)类型(`GTC`、`FOK`、`IOC`、`LIMIT_MAKER`)
`fees`|||订单的手续费

在`fees`信息组里:

名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`feeToken`|string|`USDT`|手续费计价类型
`fee`|float|`0`|实际手续费

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

## `历史成交`
返回账户的成交历史，这个API端点需要请求签名。

### **Request Weight:**

1

### **Request Url:**
```bash
GET /exapi/contract/v1/myTrades
```

### **Parameters:**
名称|类型|是否强制|默认|描述
------------ | ------------ | ------------ | ------------ | -------
`symbol`|string|`NO`|| 合约名称。如果没有发送，所有合约的成交历史都会被返回。
`limit`|integer|`NO`|`20`|返回限制(最大值为1000)
`fromId`|integer|`NO`||从TradeId开始（用来查询成交订单）
`toId`|integer|`NO`||到TradeId结束（用来查询成交订单）

### **Response:**
名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`time`|long|`1551062936784`|订单生成是的时间戳
`tradeId`|long|`49366`|成交ID
`orderId`|long|`630491436`|订单ID
`matchOrderId`|long|`630491432`| 成交对手订单ID
`symbolId`|string|`BTC-SWAP-USDT`|合约名称
`price`|float|`4765.29`|成交价格
`quantity`|float|`1.01`|成交数量
`feeTokenId`|string|`USDT`|手续费类型（Token名称）
`fee`|||实际手续费
`side`|string|`BUY`|订单方向（`BUY_OPEN`、`SELL_OPEN`、`BUY_CLOSE`、`SELL_CLOSE`）
`orderType`|string|`LIMIT`|订单类型（`LIMIT`、`MARKET`)
`pnl`|float|`100.1`|成交盈亏


### **Example:**

```js
[
  {
    'time': '1570760582848',
    'tradeId': '469968263995080704',
    'orderId': '469968263793737728',
    "matchOrderId": 436002617267469062,
    'accountId': '456552319339779840',
    'symbolId': 'BTC-SWAP-USDT',
    'price': '8531.17',
    'quantity': '100',
    'feeTokenId': 'TBTC',
    'fee': '0.00000586',
    'type': 'LIMIT',
    'side': 'BUY_OPEN',
    'pnl': '100.1'
  },...
]
```

## `当前持仓`

返回现在的仓位信息，这个API需要请求签名。

### **Request Weight:**

1

### **Request Url:**
```bash
GET /exapi/contract/v1/positions
```

### **Parameters:**
名称|类型| 是否强制  |默认|描述
------------ | ------------ |-------| ------------ | --------
`symbol`|string| `YES` ||合约名称。如果没有发送，所有的合约仓位信息都会被返回。
`side`|string| `YES` || 仓位方向，`LONG`（多仓）或者`SHORT`（空仓）。如果没有发送，两个方向的仓位信息都会被返回。

### **Response:**
 名称               | 类型       | 例子                  |描述
------------------|----------|---------------------| ------------
| `accountId       ` | string   | 1611498967211781634 |账号id|
| `positionId      ` | string   | 1858796203384319488 |仓位id|
| `symbolId        ` | string   | BTC-SWAP-USDT       |合约币对id|
| `leverage        ` | string   | 0.99                |仓位现在杠杆|
| `total           ` | string   | 100                 |总余额|
| `positionValues  ` | string   | 956.368             |仓位价值（USDT）|
| `margin          ` | string   | 961.4842            |仓位保证金|
| `minMargin       ` | string   | 953.8498            |最多减少保证金（USDT）|
| `orderMargin     ` | string   | 0                   |委托保证金|
| `avgPrice        ` | string   | 95430               |平均开仓价格|
| `liquidationPrice` | string   | 0                   |预估强平价|
| `marginRate      ` | string   | 1.0076              |当前仓位的保证金率|
| `indices         ` | string   | 95647.4             |交割指数|
| `futureRefPrice  ` | string   | 95647.4             |合约价格(根据quote-server配置返回标记或者指数价格)|
| `available       ` | string   | 100                 |可平仓数量（张）|
| `coinAvailable   ` | string   | 0                   |可用保证金|
| `isLong          ` | string   | 1                   |仓位方向: 1=多仓，0=空仓|
| `realisedPnl     ` | string   | -0.7157             |已实现盈亏|
| `unrealisedPnl   ` | string   | 2.1743              |未实现盈亏|
| `profitRate      ` | string   | 0.0022              |当前仓位的盈利率|
| `isCross         ` | integer  | 0                   |仓位模式: 1=全仓，0=逐仓|
| `positionIndex   ` | string   | 1858796203241713152 |仓位index|
| `settledUpnl     ` | string   | 0                   |已结算未实现盈亏|
| `openValue       ` | string   | 954.3               |累计开仓价值|


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

## `合约账户余额`

返回合约账户余额，这个端点需要请求签名。

### **Request Weight:**
1

### **Request Url:**
```bash
GET  /exapi/contract/v1/account
```

### **Parameters:**
None

### **Response:**
 名称         |类型| 例子    |描述
------------| ------------ |-------| ------------
| `tokenId`   |string| ETH   |币种id|
| `tokenName`  |string| ETH   |币种名称|
| `total`      |string| 10000 |总余额|
| `availableMargin` |string| 10000 |可用保证金|
| `positionMargin` |string| 0     |仓位保证金|
| `orderMargin` |string| 0     |委托保证金（下单锁定）|
| `experienceBalance` |string| 0     |体验金余额|
| `deductionBalance` |string| 0     |扣减余额|
| `grantGoldBalance` |string| 0     |赠金余额|
| `canTransfer` |string| 10000 |是否可转账|
| `totalEquity` |string| 10000 |总权益|
| `unrealisedPnl` |string| 0     |未实现盈亏|
| `walletBalance` |string| 10000 |钱包余额|
| `todayProfit` |string| 0.00  |今日盈亏|

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

## `修改保证金`

更改某个合约仓位的保证金，这个端点需要请求签名。

### **Request Weight:**
1

### **Request Url:**
```bash
POST  /exapi/contract/v1/modifyMargin
```

### **Parameters:**

名称|类型|是否强制|默认|描述
------------ | ------------ | ------------ | ------------ | -------
`symbol`|string|`YES`||合约名称。
`side`|string|`YES`||仓位方向，`LONG`（多仓）或者`SHORT`（空仓）。
`amount`|float|`YES`||增加（正值）或者减少（负值）保证金的数量。请注意这个数量指的是合约标的定价资产（即合约结算的标的）。

### **Response:**

名称|类型|例子|描述
------------ | ------------ | ------------ | ------------
`symbol`|string|`BTC-SWAP-USDT`|合约名称
`margin`|float|`12.3`|更新后的仓位保证金
`timestamp`|long|`1541161088303`|更新时间戳

### **Example:**
```js
{
  'symbol':'BTC-SWAP-USDT',
  'margin': 15,
  'timestamp': 1541161088303
}
```

## 修改保证金模式

修改保证金模式

### **Request Weight:**
1

### **Request Url:**
```bash
POST /exapi/contract/v1/set_position_mode
```

### **Parameters:**

| 名称         | 类型      |是否强制| 默认 | 描述                 |
|------------|---------|---|----|--------------------|
| symbol_id  | string  | 是 |    | 合约id               | 
| is_cross   | integer | 是 |    | 0-逐仓；1-全仓； |

### **Response:**

| 名称         | 类型       | 例子    | 描述                   |
|------------|----------|-------|----------------------|
| success    | boolean  | true  | true: 成功；  false:失败； |

### **Example:**
```json
{
  "success": true
}
```

## 查询保证金模式

查询保证金模式

### **Request Weight:**

1

### **Request URL:**

```bash
 GET /exapi/contract/v1/get_position_mode
```

### **Parameters：**

| 名称          | 类型   |是否强制| 默认 |描述|
|-------------|------|---|----|---|
| symbol_id   |string| 是 |    |合约id|

### **Response:**

| 名称         | 类型       |是否强制| 说明                  |
|------------|----------|---|---------------------|
| isCross    | boolean  |true| true: 全仓； false: 逐仓 |
| symbolId   | string   |true| 合约id                |
| switchFlag | boolean  |true| 是否允许修改              |


### **Example:**

```json
{
  "isCross": true,
  "symbolId": "BTC-SWAP-USDT",
  "switchFlag": true
}
```

## 修改合并模式
修改合并模式

### **Request Weight:**

1

### **Request URL:**
```bash 
POST /exapi/contract/v1/set_position_merge_mode
```

### **Parameters：**

| 名称          | 类型      |是否强制|说明|
|-------------|---------|---|---|
| symbol_id   | string  | 是 |合约id|
| merge_mode  | string  | 否 |分仓合仓模式|

#### 详细说明

**merge_mode**: 分仓合仓模式
<pre>
  0 - 合仓模式;
  1 - 分仓模式;
</pre>

### **Response:**

|名称|类型| 是否强制 | 说明                       |
|---|---|------|--------------------------|
| success|boolean| true | true: 成功；<br/> false:失败； |

### **Example:**

```json
{
  "success": true
}
```

## 查询合并模式

查询合并模式

### **Request Weight:**

1

### **Request URL:**
```bash
GET /exapi/contract/v1/get_position_merge_mode
```

### **Parameters：**

|名称|类型|是否强制|说明|
|---|---|---|---|
|symbol_id|string| 是 |合约id|


### **Response:**

| 名称         | 类型       | 是否强制  | 说明                                                                                      |
|------------|----------|-------|-----------------------------------------------------------------------------------------|
| mergeMode  | integer  | true  |分仓合仓模式<br /><pre><br />  0 - 合仓模式;<br />  1 - 分仓模式;<br /><br /></pre> |
| symbolId   | string   | true  |合约id|


### **Example:**

```json
{
  "mergeMode": 1,
  "symbolId": "BTC-SWAP-USDT"
}
```

## 设置杠杆
设置杠杆

### **Request Weight:**

1

### **Request URL:**

```bash
POST /exapi/contract/v1/update_leverage_merge
```

### **Parameters：**

| 名称         | 类型      |是否强制|说明|
|------------|---------|---|---|
| symbol_id  | string  | 是 |合约id|
| leverage   | integer | 是 |杠杆|
| is_long    | integer | 是 |1多仓 2空仓|

### **Response:**

| 名称       | 类型       | 是否强制  |说明|
|----------|----------|-------|---|
| success  | boolean  | true  |true:成功；  false:失败；|


### **Example:**

```json
{
  "success": true
}
```

## 查询杠杆
查询杠杆

### **Request Weight:**

1

### **Request URL:**
```bash
GET /exapi/contract/v1/query_leverage_merge
```

### **Parameters：**

| 名称        |类型|是否强制|说明|
|-----------|---|---|---|
| symbol_id |string| 是 |合约id|

### **Response:**

| 名称            | 类型       | 是否强制  | 说明                   |
|---------------|----------|-------|----------------------|
| accountId     | integer  | true  | 账号id                 |
| isCross       | boolean  | true  | true: 全仓;  false:逐仓; |
| leverage      | integer  | true  | 杠杆                   |
| leverageLong  | integer  | true  | 多仓杠杆                 |
| leverageShort | integer  | true  | 空仓杠杆                 |
| symbol_id     | string   | true  | 合约比对id               |

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

## 关键参数解释说明:

### `side`

交易的方向

`BUY_OPEN`: 开多仓

`SELL_CLOSE`: 平多仓

`SELL_OPEN`: 开空仓

`BUY_CLOSE`: 平空仓

### `priceType`

价格类型

`INPUT`: 系统将会用你输入的价格来撮合订单。

`OPPONENT`: 订单会以对手盘最优价格撮合。

假设你开多10张合约，盘口最佳买价为10最佳卖价为11，你将会下10张价格为11的合约订单。如果盘口数量不足成交10张，剩下的将留在盘口。

`QUEUE`: 订单会以相同方向的最优价格撮合。

假设你开多10张合约，盘口最佳买价为10最佳卖价为11，你将会下10张价格为10的合约订单。假设盘口原来有5张10的买单，加上你下的单现在一共有15张10的买单。

`OVER`: 订单会以对手盘的最优价格 + 超价（浮动）撮合

假设你开多10张合约，盘口最佳买价为10最佳卖价为11（超价现在为3），你将会下10张（11+3=）14的买单。如果盘口数量不足成交10张，剩下的将留在盘口。

`MARKET`: 订单会以 最新成交价 * (1 ± 5%) 撮合

假设你开多10张合约，最新成交价为10，你将会下10张（10*1.05=)10.5的买单。

### `timeInForce`

时效单类型。

`GTC`: 一直有效直到撤销。订单会一直有效除非撤销。

`IOC`: 马上成交或者撤销。订单会在一个最佳可成交价执行尽量多的交易量, 此订单可能被部份执行,剩余的部份将会自动撤销。

`FOK`: 全部成交或者撤销。订单要么在一个最佳可成交价上全部成交，要么就会直接撤销。

`LIMIT_MAKER`: 如果订单会马上成交，订单会被撤销。

### `orderType`

订单类型

`LIMIT`: 订单会以一个给定（或者更好）的价格成交

`STOP`: 一旦价格到达`triggerPrice`（触发价），订单会被触发

```
