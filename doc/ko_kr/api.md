# 문서 디렉토리

- [Broker 계약 공용 엔드포인트](#broker-계약-공용-엔드포인트)
  - [Broker API URL](#broker-api-url)
  - [계약 정보](#계약-정보)
  - [지수 가져오기](#지수-가져오기)
  - [주문서 가져오기](#주문서-가져오기)
  - [최근 거래 가져오기](#최근-거래-가져오기)
  - [K선 정보 가져오기](#k선-정보-가져오기)

- [거래 인터페이스 (서명 인증 필요)](#거래-인터페이스-서명-인증-필요)
  - [인증서 포함](#자격-증명)
  - [계약 주문](#계약-주문)
  - [미체결 주문](#미체결-주문)
  - [주문 취소](#주문-취소)
  - [주문 이력](#주문-이력)
  - [주문 상세 정보](#주문-상세-정보)
  - [거래 내역](#거래-내역)
  - [현재 포지션](#현재-포지션)
  - [계약 계좌 잔액](#계약-계좌-잔액)
  - [증거금 수정](#증거금-수정)
  - [증거금 모드 수정](#증거금-모드-수정)
  - [증거금 모드 조회](#증거금-모드-조회)
  - [포지션 병합 모드 수정](#포지션-병합-모드-수정)
  - [포지션 병합 모드 조회](#포지션-병합-모드-조회)
  - [레버리지 설정](#레버리지-설정)
  - [레버리지 조회](#레버리지-조회)

- [핵심 매개변수 설명](#주요-매개변수-설명)
  - [`side`](#side)
  - [`priceType`](#pricetype)
  - [`timeInForce`](#timeinforce)
  - [`orderType`](#ordertype)


# Broker 계약 공용 엔드포인트
## Broker API URL

Broker API URL
Broker Open API의 주소는 여기를 참조하세요.

## 계약 정보

현재 broker의 거래 규칙과 계약 symbol의 정보(정밀도 단위 등 정보)를 가져옵니다. 여기에는 계약의 리스크 한도와 승수 등의 정보가 포함됩니다.

### **Request Weight:**

0

### **Request Url:**

```bash
GET /exapi/v1/brokerInfo
```

### **Parameters:**

없음

### **Response:**
이름|유형|예시|설명
------------ | ------------ | ------------ | ------------
`timezone`|string|`UTC`|타임스탬프의 시간대.
`serverTime`|long|`1554887652929`|현재 서버의 타임스탬프를 반환합니다(밀리초 단위).

`rateLimits` 정보 그룹에서:
주문 API의 요청 제한이 표시됩니다.

이름|유형|예시|설명
------------ | ------------ | ------------ | ------------
`rateLimitType`|string|`ORDERS`|속도 제한 유형
`interval`|string|`SECOND`|속도 제한 간격
`limit`|string|`1500`|속도 제한 간격 값

`contracts` 정보 그룹에서:
현재 브로커가 거래 중인 모든 계약의 정보가 반환됩니다:

이름|유형|예시|설명
------------ | ------------ | ------------ | ------------
`symbol`|string|`BTC-SWAP-USDT`|계약 이름
`status`|string|`TRADING`|계약 상태
`baseAsset`|string|`BTC-SWAP-USDT`|기초 자산. 계약의 경우, 계약 자체가 기초 자산입니다.
`baseAssetPrecision`|float|`0.001`|기초 자산(계약 수량)의 정밀도
`quoteAsset`|string|`USDT`|가격 자산. 계약의 경우, 계약이 무엇으로 결제되는지를 나타냅니다.
`quoteAssetPrecision`|float|`0.001`|가격 자산(계약 가격)의 정밀도.
`inverse`|bool|`true`|계약이 역방향 계약인지 여부(true=역방향 계약, false=정방향 계약).
`index`|string|`BTCUSDT`|기초 지수의 이름. 기초 지수의 실시간 가격은 `index` 엔드포인트에서 확인할 수 있습니다. 예를 들어 `BTC-SWAP-USDT`는 `BTCUSDT`를 기초 지수로 사용하므로 `index` 엔드포인트에서 `BTCUSDT`의 실시간 가격을 찾을 수 있습니다.
`contractMultiplier`|string|`true`|계약의 승수.
`icebergAllowed`|string|`false`|빙산 주문을 지원하는지 여부.

`contracts`의 `filters` 정보 그룹에서:

이름|유형|예시|설명
------------ | ------------ | ------------ | ------------
`filterType`|string|`PRICE_FILTER`|필터 유형.
`minPrice`|float|`0.001`|허용되는 최소 가격.
`maxPrice`|float|`100000.00000000`|허용되는 최대 가격.
`tickSize`|float|`0.001`|계약 가격의 정밀도.
`minQty`|float|`0.01`|허용되는 최소 수량.
`maxQty`|float|`100000.00000000`|허용되는 최대 수량.
`stepSize`|float|`0.001`|계약 수량의 정밀도.
`minNotional`|float|`1`|최소 거래 금액 제한(수량 * 가격)

`riskLimits` 정보 그룹에서:

이름|유형|예시|설명
------------ | ------------ | ------------ | ------------
`quantity`|float|`100`|포지션이 이 수량보다 작고 이전 단계의 `quantity`보다 큰 경우 다음 요구 사항을 참조해야 합니다.
`initialMargin`|float|`0.1`|초기 증거금 비율.
`maintMargin`|float|`0.03`|최소 유지 증거금 비율.

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



## `지수 가져오기`

기초 지수 가격

### **Request Weight:**
0

### **Request Url:**
```bash
GET /exapi/quote/v1/contract/index
```
### **Parameters:**
이름|유형|필수 여부|기본값|설명
------------ | ------------ | ------------ | ------------ | ----
symbol|string|`NO`||기초 지수 이름. 이 값이 없으면 모든 기초 지수의 가격이 반환됩니다.

### **Response:**
이름|유형|예시|설명
------------ | ------------ | ------------ | ------------
`index`|float|`8342.73`|기초 지수의 가격.
`EDP`|float|`8432.32`|기초 지수의 EDP(예상 결제 가격, 지난 10분 동안의 지수 가격 평균).

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

## `주문서 가져오기`

현재 주문서의 데이터를 가져옵니다.

### **Request Weight:**

수량에 따라 다릅니다. 요청 수량이 많을수록 무게가 커집니다:

수량|요청 무게
------------ | ------------
5, 10, 20, 50, 100|1
500|5
1000|10

### **Request Url:**
```
GET /exapi/quote/v1/contract/depth
```

### **Parameters:**
이름|유형|필수 여부|기본값|설명
------------ | ------------ | ------------ | ------------ | -----
`symbol`|string|`YES`||주문서를 가져올 계약 이름.
`limit`|integer|`NO`|`100` (최대 = 100)|반환할 `bids`와 `asks`의 수량

### **Response:**
이름|유형|예시|설명
------------ | ------------ | ------------ | ------------
`time`|long|`1550829103981`|현재 시간 (Unix Timestamp, 밀리초 ms)
`bids`|list|(아래와 같음)|모든 bid의 가격과 수량 정보, 최적의 bid 가격이 위에서 아래로 정렬됨.
`asks`|list|(아래와 같음)|모든 ask의 가격과 수량 정보, 최적의 ask 가격이 위에서 아래로 정렬됨.

`bids`와 `asks`에 해당하는 정보 그룹은 주문서의 모든 가격과 가격에 해당하는 수량 정보를 나타내며, 최적의 가격이 위에서 아래로 정렬됩니다.

이름|유형|예시|설명
------------ | ------------ | ------------ | ------------
`''`|float|`123.10`|가격
`''`|float|`300`|현재 가격에 해당하는 수량

### **Example:**

```js
{
  "time": 1555049455783,
  "bids": [
   ["78.82", "0.526"],//[가격, 수량]
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
     ["122.96", "0.381"],//[가격, 수량]
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



## `최근 거래 가져오기`

특정 계약의 최근 거래 주문 정보를 가져옵니다.

### **Request Weight:**

1

### **Request URL:**
```
GET /exapi/quote/v1/contract/trades
```
### **Parameters:**
이름|유형|필수 여부|기본값|설명
------------ | ------------ | ------------ | ------------ | ------
`symbol`|string|`YES`||계약 이름
`limit`|integer|`NO` (최대값 60)|`60`|반환할 거래 주문의 수량

### **Response:**
이름|유형|예시|설명
------------ | ------------ | ------------ | ------------
`price`|float|`0.055`|거래 가격
`time`|long|`1537797044116`|현재 Unix 타임스탬프, 밀리초(ms)
`qty`|float|`5`|수량(장수)
`isBuyerMaker`|string|`true`|매도자 또는 매수자. `true`=매도자, `false`=매수자

### **Example:**
```js
[
  {
    "price": "1.21",
    "time": 1555034474064,
    "qty": "0.725",
    "isBuyerMaker": false
  },...
]
```

## `K선 정보 가져오기`

특정 계약의 K선 정보(고가, 저가, 시가, 종가, 거래량 등)를 가져옵니다.

### **Request Weight:**

1

### **Request URL:**
```
GET /exapi/quote/v1/contract/klines
```

### **Parameters:**
이름|유형|필수 여부|기본값|설명
------------ | ------------ | ------------ | ------------ | ----
`symbol`|string|`YES`||계약 이름
`interval`|string|`YES`||K선 구간. 인식 가능한 값: `1m`, `5m`, `15m`, `30m`, `1h`, `1d`, `1w`, `1M` (`m`=분, `h`=시간, `d`=일, `w`=주, `M`=월)
`limit`|integer|`NO`|`1000`|반환할 값의 수, 최대값은 1000
`to`|integer|`NO`||마지막 데이터 포인트의 타임스탬프

### **Response:**
이름|유형|예시|설명
------------ | ------------ | ------------ | ------------
`''`|long|`1538728740000`|시작 타임스탬프, 밀리초(ms)
`''`|float|`36.00000`|시가
`''`|float|`36.00000`|고가
`''`|float|`36.00000`|저가
`''`|float|`36.00000`|종가
`''`|float|`148976.11427815`|계약 거래 금액
`''`|long|`1538728740000`|종료 타임스탬프, 밀리초(ms)
`''`|float|`2434.19055334`|거래 수량(장수)
`''`|integer|`308`|체결 수량(장수)
`''`|float|`1756.87402397`|매수자 구매 금액
`''`|float|`28.46694368`|매수자 구매 수량(장수)

### **Example:**
```js
[
  [
    1538728740000, //'시작 시간'
    '36.000000000000000000', //'시가'
    '36.000000000000000000', //'고가'
    '36.000000000000000000', //'저가'
    '36.000000000000000000', //'종가'
    '148976.11427815',  // 계약 거래 금액
    1499644799999,      // 종료 시간
    '2434.19055334',    // 거래 수량(장수)
    308,                // 체결 수량(장수)
    '1756.87402397',    // 매수자 구매 금액
    '28.46694368'       // 매수자 구매 수량(장수)
  ],...
]
```
`base asset`은 매수자가 받은 계약 수량을 나타냅니다.

`quote asset`은 계약을 얻기 위해 사용된 토큰 수량을 나타냅니다.

# 거래 인터페이스 (서명 인증 필요)
## 자격 증명
참조: [인터페이스 서명](signature.md)

## `계약 주문`

계약 주문, 이 엔드포인트는 서명 인증이 필요합니다.

### **Request Weight:**

1

### **Request URL:**
```bash
POST /exapi/contract/v1/order
```

### **Parameters:**
이름| 유형          | 필수 여부        |기본값|설명
------------ |-------------|-------------| ------------ | ------------
`symbol_id`| string      | `YES`       ||계약 이름.
`client_order_id`| string/long | `YES`       ||주문 ID, 사용자가 정의.
`side`| string      | `YES`       ||주문 방향, 방향 유형은 `BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, `SELL_CLOSE`.
`type`| string      | `YES`       ||주문 유형, 지원되는 주문 유형은 `LIMIT` 및 `STOP`.
`price`| float       | `NO`. (`LIMIT`&`INPUT`) 주문 **필수** ||주문 가격.
`price_type`| string      | `NO`|`INPUT`|가격 유형, 지원되는 가격 유형은 `INPUT`, `OPPONENT`, `QUEUE`, `OVER`, `MARKET_PRICE`.
`quantity`| float       | `YES`       ||주문 계약 수량.
`leverage`| float       | `YES`.（\*\_CLOSE 평창 주문 **필수 아님**） ||주문 레버리지.
`time_in_force`| string      | `NO`        |`GTC`|`LIMIT` 주문의 시간 지시(Time in Force), 현재 지원되는 유형은 `GTC`, `FOK`, `IOC`, `LIMIT_MAKER`.
`is_long`| number      | `YES`        |`GTC`|0-공매도; 1-공매수;
`position_index` | number | `NO`        | 0 | 포지션 인덱스, 분할 포지션에서 평창 시 필수; 다른 경우에는 전달하지 않거나 0을 전달

**주의:** **시장 주문**의 경우, `orderType`을 **`LIMIT`**으로 설정하고 **`priceType`**을 **`MARKET_PRICE`**로 설정해야 합니다.

계약 가격 및 수량의 정밀도 구성 정보는 `brokerInfo` 엔드포인트에서 확인할 수 있습니다.

주의: 잔액이 필요한 증거금 요구 사항(초기 증거금 + 개설 수수료 + 평창 수수료)을 충족하지 못하면 "*insufficient balance*" (잔액 부족) 오류가 반환됩니다.

*가격 유형* 및 *주문 유형*에 대한 자세한 설명은 문서 끝을 참조하세요.

### **Response:**
이름| 유형       | 예시                   |설명
------------ |----------|----------------------| ------------
`time`   | string   | `1570759718825`      |주문 생성 시 타임스탬프
`orderId`| string   | `469961015902208000` |주문 ID
`accountId`| string   | `469961015902208000` |계정 ID
`clientOrderId`| string   | `213443`             |사용자 정의 주문 ID
`symbolId`| string   | `BTC-SWAP-USDT`      | 계약 통화 쌍 ID
`symbolName`| string   | `BTC-SWAP-USDT`      | 계약 통화 쌍 이름
`baseTokenId`| string   | `BTC-SWAP-USDT`      | 통화 ID
`baseTokenName`| string   | `BTC-SWAP-USDT`      | 통화 이름
`quoteTokenId`| string   | `USDT`               | 토큰 통화 ID
`quoteTokenName`| string   | `USDT`               | 토큰 통화 이름
`price`| string   | `8200`               |주문 가격
`origQty`| string   | `100`                |주문 수량
`executedQty`| string   | `100`                |실행된 주문 수량
`executedAmount`| string   | `100`                |실행된 주문 금액
`avgPrice`| string   | `4754.24`            |평균 거래 가격
`type`| string   | `STOP`               |주문 유형, 지원되는 주문 유형은 `LIMIT` 및 `STOP`
`side`| string   | `BUY`                |주문 방향 (`BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, `SELL_CLOSE`)
`fees`|          |                      |주문 수수료
`status`| string   | `NEW`                |주문 상태 (`NEW`, `PARTIALLY_FILLED`, `FILLED`, `CANCELED`, `REJECTED`)
`noExecutedQty`| string   | `0`                  |실행되지 않은 주문 수량
`exchangeId`| string   | `301`                |거래소 ID
`margin`| string   | `99`                 | 증거금
`leverage`| string   | `4`                  |주문 레버리지
`isClose`| boolean  | `false`              |false- 개설; true-평창
`priceType`| string   | `INPUT`              |가격 유형 (`INPUT`, `OPPONENT`, `QUEUE`, `OVER`, `MARKET`)
`isLiquidationOrder`| boolean | `false`              |평창 주문 여부; true-예; false-아니오
`liquidationType`| string | `NO_LIQ`             |청산 주문 유형: IOC 청산 시 강제 청산 주문 ADL 청산 시 감산 주문
`liquidationPrice`| string | `0`                  |강제 청산 가격
`closePnl`| string | `12122`              |평창 손익
`updateTime`|string| `1551062936784`      |주문 마지막 업데이트 타임스탬프
`timeInForce`|string| `GTC`                |시간 지시(Time in Force) 유형(`GTC`, `FOK`, `IOC`, `LIMIT_MAKER`)

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

## `미체결 주문`


### **Request Weight:**

1

### **Request Url:**
```bash
GET /exapi/contract/v1/order/open_orders
```

### **Parameter:**

 이름        | 타입      | 필수 여부 | 기본값 | 설명                       
-----------|---------|------|----|--------------------------
| symbol_id | string  | 아니오 |    | 주문의 ID                    
| type      | string  | 아니오 |    | 주문 유형, 지원되는 주문 유형은 LIMIT과 STOP입니다. |
| limit     | number  | 아니오 |    | 수량                       |


### **Response:**

이름|타입|예시|설명
------------ | ------------ | ------------ | ------------
`time`   | string   | `1570759718825`      |주문 생성 시의 타임스탬프
`orderId`| string   | `469961015902208000` |주문 ID
`accountId`| string   | `469961015902208000` |계정 ID
`clientOrderId`| string   | `213443`             |사용자 정의 주문 ID
`symbolId`| string   | `BTC-SWAP-USDT`      | 계약 코인 페어 ID
`symbolName`| string   | `BTC-SWAP-USDT`      | 계약 코인 페어 이름
`baseTokenId`| string   | `BTC-SWAP-USDT`      | 코인 ID
`baseTokenName`| string   | `BTC-SWAP-USDT`      | 코인 이름
`quoteTokenId`| string   | `USDT`               | 토큰 코인 ID
`quoteTokenName`| string   | `USDT`               | 토큰 코인 이름
`price`| string   | `8200`               |주문 가격
`origQty`| string   | `100`                |주문 수량
`executedQty`| string   | `100`                |실행된 주문 수량
`executedAmount`| string   | `100`                |실행된 주문 금액
`avgPrice`| string   | `4754.24`            |평균 거래 가격
`type`| string   | `STOP`               |주문 유형, 지원되는 주문 유형은 `LIMIT`과 `STOP`
`side`| string   | `BUY`                |주문 방향 (`BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, `SELL_CLOSE`)
`fees`|          |                      |주문 수수료
`status`| string   | `NEW`                |주문 상태 (`NEW`, `PARTIALLY_FILLED`, `FILLED`, `CANCELED`, `REJECTED`)
`noExecutedQty`| string   | `0`                  |미실행 주문 수량
`exchangeId`| string   | `301`                |거래소 ID
`margin`| string   | `99`                 | 증거금
`leverage`| string   | `4`                  |주문 레버리지
`isClose`| boolean  | `false`              |false- 개시; true- 청산
`priceType`| string   | `INPUT`              |가격 유형 (`INPUT`, `OPPONENT`, `QUEUE`, `OVER`, `MARKET`)
`isLiquidationOrder`| boolean | `false`              |청산 주문 여부; true-예; false-아니오
`liquidationType`| string | `NO_LIQ`             |청산 주문 유형: IOC 청산 시의 강제 청산 주문 ADL 청산 시의 감량 주문
`liquidationPrice`| string | `0`                  |강제 청산 가격
`closePnl`| string | `12122`              |청산 손익
`updateTime`|string| `1551062936784`      |주문 마지막 업데이트 타임스탬프
`timeInForce`|string| `GTC`                |유효 시간 (Time in Force) 유형 (`GTC`, `FOK`, `IOC`, `LIMIT_MAKER`)

`fees` 정보 그룹 내:

이름|타입|예시|설명
------------ | ------------ | ------------ | ------------
`feeToken`|string|`USDT`|수수료 계산 유형
`fee`|float|`0`|실제 수수료

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



## `주문 취소`

특정 주문을 취소하려면 `orderId` 또는 `clientOrderId` 중 하나를 보내야 합니다. 이 API 엔드포인트는 요청 서명이 필요합니다.

### **Request Weight:**

1

### **Request Url:**
```bash
DELETE /exapi/contract/v1/order/cancel
```

### **Parameter:**

이름|타입|필수 여부|기본값|설명
------------ | ------------ | ------------ | ------------ | -----
`orderId`|integer|`NO`||주문 ID
`clientOrderId`|string|`NO`||사용자 정의 주문 ID
`orderType`|string|`YES`||주문 유형 (`LIMIT` 및 `STOP`)

**주의:** `orderId` 또는 `clientOrderId` **중 하나는 반드시 보내야 합니다**

### **Response:**

이름|타입|예시|설명
------------ | ------------ | ------------ | ------------
`time`|long|`1570759718825`|주문 생성 시의 타임스탬프
`updateTime`|long|`1551062936784`|주문 마지막 업데이트 타임스탬프
`orderId`|integer|`469961015902208000`|주문 ID
`clientOrderId`|string|`213443`|사용자 정의 주문 ID
`symbol`|string|`BTC-SWAP-USDT`|계약 이름
`price`|float|`8200`|주문 가격
`leverage`|float|`4`|주문 레버리지
`origQty`|float|`1.01`|주문 수량
`executedQty`|float|`1.01`|실행된 주문 수량
`avgPrice`|float|`4754.24`|평균 거래 가격
`marginLocked`|float|`200`|해당 주문에 잠긴 증거금. 여기에는 실제 필요한 증거금 외에 개시 및 청산에 필요한 비용이 포함됩니다.
`orderType`|string|`YES`|주문 유형 (`LIMIT` 및 `STOP`)
`side`|string|`BUY`|주문 방향 (`BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, `SELL_CLOSE`)
`fees`|||주문 수수료
`timeInForce`|string|`GTC`|유효 시간 (Time in Force) 유형 (`GTC`, `FOK`, `IOC`, `LIMIT_MAKER`)
`status`|string|`NEW`|주문 상태 (`NEW`, `PARTIALLY_FILLED`, `FILLED`, `CANCELED`, `REJECTED`). 이 엔드포인트가 반환하는 주문 상태는 모두 `CANCELED`입니다.
`priceType`|string|`INPUT`|가격 유형 (`INPUT`, `OPPONENT`, `QUEUE`, `OVER`, `MARKET`)

`fees` 정보 그룹 내:

이름|타입|예시|설명
------------ | ------------ | ------------ | ------------
`feeToken`|string|`USDT`|수수료 계산 유형
`fee`|float|`0`|실제 수수료

###  **Example:**

```json
{
  "time": "1570759718825",
  "updateTime": "0",
  "orderId": "469961015902208000",
  "clientOrderId": "6423344174",
  "symbol": "BTC-SWAP-USDT",
  "price": "8200",
  "leverage": "12.08",
  "origQty": "5",
  "executedQty": "0",
  "avgPrice": "0",
  "marginLocked": "0",
  "orderType": "LIMIT",
  "side": "BUY_OPEN",
  "fees": [],
  "timeInForce": "GTC",
  "status": "CANCELED",
  "priceType": "INPUT"
}
```

## `주문 이력`

부분적으로 또는 완전히 체결되었거나 취소된 주문의 이력을 조회합니다. 이 API 엔드포인트는 요청 서명이 필요합니다.

### **Request Weight:**

1

### **Request Url:**
```bash
GET /exapi/contract/v1/historyOrders
```

### **Parameters:**
이름|타입|필수 여부|기본값|설명
------------ | ------------ | ------------ | ------------ | --------
`symbol`|string|`NO`||열린 주문을 반환할 심볼. 보내지 않으면 모든 계약의 주문이 반환됩니다.
`orderId`|integer|`NO`|| 주문 ID
`orderType`|string|`YES`||주문 유형, 가능한 유형: `LIMIT`, `STOP`
`limit`|integer|`NO`|`20`|반환할 항목 수.

`orderId`가 설정된 경우, 해당 `orderId`보다 작은 주문을 가져옵니다. 그렇지 않으면 가장 최근의 주문이 반환됩니다.

### **Response:**
이름|타입|예시|설명
------------ | ------------ | ------------ | ------------
`time`|long|`1570759718825`|주문 생성 시의 타임스탬프
`updateTime`|long|`1551062936784`|주문 마지막 업데이트 타임스탬프
`orderId`|integer|`469961015902208000`|주문 ID
`clientOrderId`|string|`213443`|사용자 정의 주문 ID
`symbol`|string|`BTC-SWAP-USDT`|계약 이름
`price`|float|`8200`|주문 가격
`leverage`|float|`4`|주문 레버리지
`origQty`|float|`1.01`|주문 수량
`executedQty`|float|`1.01`|실행된 주문 수량
`avgPrice`|float|`4754.24`|평균 거래 가격
`marginLocked`|float|`200`|해당 주문에 잠긴 증거금. 여기에는 실제 필요한 증거금 외에 개시 및 청산에 필요한 비용이 포함됩니다.
`orderType`|string|`YES`|주문 유형 (`LIMIT` 및 `STOP`)
`side`|string|`BUY`|주문 방향 (`BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, `SELL_CLOSE`)
`fees`|||주문 수수료
`timeInForce`|string|`GTC`|유효 시간 (Time in Force) 유형 (`GTC`, `FOK`, `IOC`, `LIMIT_MAKER`)
`status`|string|`NEW`|주문 상태 (`NEW`, `PARTIALLY_FILLED`, `FILLED`, `CANCELED`, `REJECTED`)
`priceType`|string|`INPUT`|가격 유형 (`INPUT`, `OPPONENT`, `QUEUE`, `OVER`, `MARKET`)

`fees` 정보 그룹 내:

이름|타입|예시|설명
------------ | ------------ | ------------ | ------------
`feeToken`|string|`USDT`|수수료 계산 유형
`fee`|float|`0`|실제 수수료

### **Example:**

```json
[
  {
    "time": "1570759718825",
    "updateTime": "0",
    "orderId": "469961015902208000",
    "clientOrderId": "6423344174",
    "symbol": "BTC-SWAP-USDT",
    "price": "8200",
    "leverage": "12.08",
    "origQty": "5",
    "executedQty": "0",
    "avgPrice": "0",
    "marginLocked": "0",
    "orderType": "LIMIT",
    "side": "BUY_OPEN",
    "fees": [],
    "timeInForce": "GTC",
    "status": "CANCELED",
    "priceType": "INPUT"
  }
]
```

## `주문 상세 정보`

특정 주문의 상세 정보를 조회합니다.

### **Request Weight:**

1

### **Request Url:**
```bash
GET /exapi/contract/v1/getOrder
```

### **Parameters:**
이름|타입|필수 여부|기본값|설명
------------ | ------------ | ------------ | ------------ | --------
`orderId`|integer|`NO`||주문 ID
`clientOrderId`|string|`NO`||사용자 정의 주문 ID
`orderType`|string|`YES`||주문 유형 (`LIMIT` 및 `STOP`)

**주의:** `orderId` 또는 `clientOrderId` **중 하나는 반드시 보내야 합니다**

### **Response:**
이름|타입|예시|설명
------------ | ------------ | ------------ | ------------
`time`|long|`1570759718825`|주문 생성 시의 타임스탬프
`updateTime`|long|`1551062936784`|주문 마지막 업데이트 타임스탬프
`orderId`|integer|`469961015902208000`|주문 ID
`clientOrderId`|string|`213443`|사용자 정의 주문 ID
`symbol`|string|`BTC-SWAP-USDT`|계약 이름
`price`|float|`8200`|주문 가격
`leverage`|float|`4`|주문 레버리지
`origQty`|float|`1.01`|주문 수량
`executedQty`|float|`1.01`|실행된 주문 수량
`avgPrice`|float|`4754.24`|평균 거래 가격
`marginLocked`|float|`200`|해당 주문에 잠긴 증거금. 여기에는 실제 필요한 증거금 외에 개시 및 청산에 필요한 비용이 포함됩니다.
`orderType`|string|`YES`|주문 유형 (`LIMIT` 및 `STOP`)
`priceType`|string|`INPUT`|가격 유형 (`INPUT`, `OPPONENT`, `QUEUE`, `OVER`, `MARKET`)
`side`|string|`BUY`|주문 방향 (`BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, `SELL_CLOSE`)
`status`|string|`NEW`|주문 상태 (`NEW`, `PARTIALLY_FILLED`, `FILLED`, `CANCELED`, `REJECTED`)
`timeInForce`|string|`GTC`|유효 시간 (Time in Force) 유형 (`GTC`, `FOK`, `IOC`, `LIMIT_MAKER`)
`fees`|||주문 수수료

`fees` 정보 그룹 내:

이름|타입|예시|설명
------------ | ------------ | ------------ | ------------
`feeToken`|string|`USDT`|수수료 계산 유형
`fee`|float|`0`|실제 수수료

### **Example:**

```json
{
  "time": "1570760254539",
  "updateTime": "0",
  "orderId": "469965509788581888",
  "clientOrderId": "1570760253946",
  "symbol": "BTC-SWAP-USDT",
  "price": "8502.34",
  "leverage": "20",
  "origQty": "222",
  "executedQty": "0",
  "avgPrice": "0",
  "marginLocked": "0.00130552",
  "orderType": "LIMIT",
  "side": "BUY_OPEN",
  "fees": [],
  "timeInForce": "GTC",
  "status": "NEW",
  "priceType": "INPUT"
}
```

## `거래 내역`

계정의 거래 내역을 반환합니다. 이 API 엔드포인트는 요청 서명이 필요합니다.

### **Request Weight:**

1

### **Request Url:**
```bash
GET /exapi/contract/v1/myTrades
```

### **Parameters:**
이름|타입|필수 여부|기본값|설명
------------ | ------------ | ------------ | ------------ | -------
`symbol`|string|`NO`|| 계약 이름. 보내지 않으면 모든 계약의 거래 내역이 반환됩니다.
`limit`|integer|`NO`|`20`|반환 제한 (최대값은 1000)
`fromId`|integer|`NO`||TradeId부터 시작 (거래 주문 조회에 사용)
`toId`|integer|`NO`||TradeId까지 끝 (거래 주문 조회에 사용)

### **Response:**
이름|타입|예시|설명
------------ | ------------ | ------------ | ------------
`time`|long|`1551062936784`|주문 생성 시의 타임스탬프
`tradeId`|long|`49366`|거래 ID
`orderId`|long|`630491436`|주문 ID
`matchOrderId`|long|`630491432`| 거래 상대 주문 ID
`symbolId`|string|`BTC-SWAP-USDT`|계약 이름
`price`|float|`4765.29`|거래 가격
`quantity`|float|`1.01`|거래 수량
`feeTokenId`|string|`USDT`|수수료 유형 (토큰 이름)
`fee`|float|`0.00000586`|실제 수수료
`side`|string|`BUY`|주문 방향 (`BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, `SELL_CLOSE`)
`orderType`|string|`LIMIT`|주문 유형 (`LIMIT`, `MARKET`)
`pnl`|float|`100.1`|거래 손익

### **Example:**

```json
[
  {
    "time": "1570760582848",
    "tradeId": "469968263995080704",
    "orderId": "469968263793737728",
    "matchOrderId": 436002617267469062,
    "accountId": "456552319339779840",
    "symbolId": "BTC-SWAP-USDT",
    "price": "8531.17",
    "quantity": "100",
    "feeTokenId": "TBTC",
    "fee": "0.00000586",
    "type": "LIMIT",
    "side": "BUY_OPEN",
    "pnl": "100.1"
  }
]
```

## `현재 포지션`

현재 포지션 정보를 반환합니다. 이 API는 요청 서명이 필요합니다.

### **Request Weight:**

1

### **Request Url:**
```bash
GET /exapi/contract/v1/positions
```

### **Parameters:**
이름|타입|필수 여부|기본값|설명
------------ | ------------ |-------| ------------ | --------
`symbol`|string| `YES` ||계약 이름. 보내지 않으면 모든 계약의 포지션 정보가 반환됩니다.
`side`|string| `YES` || 포지션 방향, `LONG`(롱 포지션) 또는 `SHORT`(숏 포지션). 보내지 않으면 두 방향의 포지션 정보가 반환됩니다.

### **Response:**
이름|타입|예시|설명
------------------|----------|---------------------| ------------
`accountId`       | string   | 1611498967211781634 |계정 ID
`positionId`      | string   | 1858796203384319488 |포지션 ID
`symbolId`        | string   | BTC-SWAP-USDT       |계약 코인 페어 ID
`leverage`        | string   | 0.99                |현재 포지션 레버리지
`total`           | string   | 100                 |총 잔액
`positionValues`  | string   | 956.368             |포지션 가치 (USDT)
`margin`          | string   | 961.4842            |포지션 증거금
`minMargin`       | string   | 953.8498            |최대 감소 가능 증거금 (USDT)
`orderMargin`     | string   | 0                   |주문 증거금
`avgPrice`        | string   | 95430               |평균 개시 가격
`liquidationPrice`| string   | 0                   |예상 청산 가격
`marginRate`      | string   | 1.0076              |현재 포지션의 증거금 비율
`indices`         | string   | 95647.4             |인덱스 가격
`futureRefPrice`  | string   | 95647.4             |계약 가격 (quote-server 설정에 따라 마크 또는 인덱스 가격 반환)
`available`       | string   | 100                 |청산 가능 수량 (장)
`coinAvailable`   | string   | 0                   |사용 가능 증거금
`isLong`          | string   | 1                   |포지션 방향: 1=롱 포지션, 0=숏 포지션
`realisedPnl`     | string   | -0.7157             |실현 손익
`unrealisedPnl`   | string   | 2.1743              |미실현 손익
`profitRate`      | string   | 0.0022              |현재 포지션의 수익률
`isCross`         | integer  | 0                   |포지션 모드: 1=교차, 0=격리
`positionIndex`   | string   | 1858796203241713152 |포지션 인덱스
`settledUpnl`     | string   | 0                   |정산된 미실현 손익
`openValue`       | string   | 954.3               |누적 개시 가치

### **Example:**
```json
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

## `계약 계좌 잔액`

계약 계좌 잔액을 반환합니다. 이 엔드포인트는 요청 서명이 필요합니다.

### **Request Weight:**
1

### **Request Url:**
```bash
GET /exapi/contract/v1/account
```

### **Parameters:**
없음

### **Response:**
이름|타입|예시|설명
------------| ------------ |-------| ------------
`tokenId`   |string| ETH   |코인 ID
`tokenName`  |string| ETH   |코인 이름
`total`      |string| 10000 |총 잔액
`availableMargin` |string| 10000 |사용 가능 증거금
`positionMargin` |string| 0     |포지션 증거금
`orderMargin` |string| 0     |주문 증거금 (주문 시 잠금)
`experienceBalance` |string| 0     |체험 잔액
`deductionBalance` |string| 0     |차감 잔액
`grantGoldBalance` |string| 0     |보너스 잔액
`canTransfer` |string| 10000 |이체 가능 여부
`totalEquity` |string| 10000 |총 자산
`unrealisedPnl` |string| 0     |미실현 손익
`walletBalance` |string| 10000 |지갑 잔액
`todayProfit` |string| 0.00  |오늘의 손익

### **Example:**

```json


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

## `증거금 수정`

특정 계약 포지션의 증거금을 변경합니다. 이 엔드포인트는 요청 서명이 필요합니다.

### **Request Weight:**
1

### **Request Url:**
```bash
POST /exapi/contract/v1/modifyMargin
```

### **Parameters:**

이름|타입|필수 여부|기본값|설명
------------ | ------------ | ------------ | ------------ | -------
`symbol`|string|`YES`||계약 이름
`side`|string|`YES`||포지션 방향, `LONG`(롱 포지션) 또는 `SHORT`(숏 포지션)
`amount`|float|`YES`||증가(양수) 또는 감소(음수)할 증거금의 수량. 이 수량은 계약의 기초 자산(즉, 계약 결제의 기초 자산)을 기준으로 합니다.

### **Response:**

이름|타입|예시|설명
------------ | ------------ | ------------ | ------------
`symbol`|string|`BTC-SWAP-USDT`|계약 이름
`margin`|float|`12.3`|업데이트된 포지션 증거금
`timestamp`|long|`1541161088303`|업데이트 타임스탬프

### **Example:**

```json
{
  "symbol": "BTC-SWAP-USDT",
  "margin": 15,
  "timestamp": 1541161088303
}
```

## 증거금 모드 수정

증거금 모드를 수정합니다.

### **Request Weight:**
1

### **Request Url:**
```bash
POST /exapi/contract/v1/set_position_mode
```

### **Parameters:**

이름|타입|필수 여부|기본값|설명
------------|---------|---|----|--------------------
`symbol_id`|string|`YES`||계약 ID
`is_cross`|integer|`YES`||0-격리; 1-교차

### **Response:**

이름|타입|예시|설명
------------|----------|-------|----------------------
`success`|boolean|true|true: 성공; false: 실패

### **Example:**

```json
{
  "success": true
}
```

## 증거금 모드 조회

증거금 모드를 조회합니다.

### **Request Weight:**

1

### **Request URL:**

```bash
GET /exapi/contract/v1/get_position_mode
```

### **Parameters：**

이름|타입|필수 여부|기본값|설명
-------------|------|---|----|---
`symbol_id`|string|`YES`||계약 ID

### **Response:**

이름|타입|필수 여부|설명
------------|----------|---|---------------------
`isCross`|boolean|true|true: 교차; false: 격리
`symbolId`|string|true|계약 ID
`switchFlag`|boolean|true|수정 가능 여부

### **Example:**

```json
{
  "isCross": true,
  "symbolId": "BTC-SWAP-USDT",
  "switchFlag": true
}
```

## 포지션 병합 모드 수정

포지션 병합 모드를 수정합니다.

### **Request Weight:**

1

### **Request URL:**

```bash
POST /exapi/contract/v1/set_position_merge_mode
```

### **Parameters：**

이름|타입|필수 여부|설명
-------------|---------|---|---
`symbol_id`|string|`YES`|계약 ID
`merge_mode`|string|`NO`|포지션 병합 모드

#### 상세 설명

**merge_mode**: 포지션 병합 모드
<pre>
  0 - 병합 모드;
  1 - 분리 모드;
</pre>

### **Response:**

이름|타입|필수 여부|설명
---|---|------|--------------------------
`success`|boolean|true|true: 성공; false: 실패

### **Example:**

```json
{
  "success": true
}
```

## 포지션 병합 모드 조회

포지션 병합 모드를 조회합니다.

### **Request Weight:**

1

### **Request URL:**

```bash
GET /exapi/contract/v1/get_position_merge_mode
```

### **Parameters：**

이름|타입|필수 여부|설명
---|---|---|---
`symbol_id`|string|`YES`|계약 ID

### **Response:**

이름|타입|필수 여부|설명
------------|----------|-------|-----------------------------------------------------------------------------------------
`mergeMode`|integer|true|포지션 병합 모드<br /><pre><br />  0 - 병합 모드;<br />  1 - 분리 모드;<br /><br /></pre>
`symbolId`|string|true|계약 ID

### **Example:**

```json
{
  "mergeMode": 1,
  "symbolId": "BTC-SWAP-USDT"
}
```

## 레버리지 설정

레버리지를 설정합니다.

### **Request Weight:**

1

### **Request URL:**

```bash
POST /exapi/contract/v1/update_leverage_merge
```

### **Parameters：**

이름|타입|필수 여부|설명
------------|---------|---|---
`symbol_id`|string|`YES`|계약 ID
`leverage`|integer|`YES`|레버리지
`is_long`|integer|`YES`|1: 롱 포지션, 2: 숏 포지션

### **Response:**

이름|타입|필수 여부|설명
----------|----------|-------|---
`success`|boolean|true|true: 성공; false: 실패

### **Example:**

```json
{
  "success": true
}
```

## 레버리지 조회

레버리지를 조회합니다.

### **Request Weight:**

1

### **Request URL:**

```bash
GET /exapi/contract/v1/query_leverage_merge
```

### **Parameters：**

이름|타입|필수 여부|설명
-----------|---|---|---
`symbol_id`|string|`YES`|계약 ID

### **Response:**

이름|타입|필수 여부|설명
---------------|----------|-------|----------------------
`accountId`|integer|true|계정 ID
`isCross`|boolean|true|true: 교차; false: 격리
`leverage`|integer|true|레버리지
`leverageLong`|integer|true|롱 포지션 레버리지
`leverageShort`|integer|true|숏 포지션 레버리지
`symbol_id`|string|true|계약 ID

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

# 주요 매개변수 설명:

## `side`

거래 방향

`BUY_OPEN`: 매수 포지션 열기

`SELL_CLOSE`: 매수 포지션 닫기

`SELL_OPEN`: 매도 포지션 열기

`BUY_CLOSE`: 매도 포지션 닫기

## `priceType`

가격 유형

`INPUT`: 시스템이 입력한 가격으로 주문을 매칭합니다.

`OPPONENT`: 주문이 상대방의 최적 가격으로 매칭됩니다.

예를 들어, 10개의 계약을 매수하고, 최적 매수 가격이 10, 최적 매도 가격이 11인 경우, 11의 가격으로 10개의 계약 주문을 하게 됩니다. 만약 주문량이 10개를 충족하지 못하면, 남은 수량은 주문서에 남게 됩니다.

`QUEUE`: 주문이 동일 방향의 최적 가격으로 매칭됩니다.

예를 들어, 10개의 계약을 매수하고, 최적 매수 가격이 10, 최적 매도 가격이 11인 경우, 10의 가격으로 10개의 계약 주문을 하게 됩니다. 만약 기존에 10의 매수 주문이 5개 있었다면, 총 15개의 10 가격의 매수 주문이 됩니다.

`OVER`: 주문이 상대방의 최적 가격 + 초과 가격(변동)으로 매칭됩니다.

예를 들어, 10개의 계약을 매수하고, 최적 매수 가격이 10, 최적 매도 가격이 11이며 초과 가격이 3인 경우, 14(11+3)의 가격으로 10개의 매수 주문을 하게 됩니다. 만약 주문량이 10개를 충족하지 못하면, 남은 수량은 주문서에 남게 됩니다.

`MARKET`: 주문이 최신 거래 가격 * (1 ± 5%)로 매칭됩니다.

예를 들어, 10개의 계약을 매수하고, 최신 거래 가격이 10인 경우, 10.5(10*1.05)의 가격으로 10개의 매수 주문을 하게 됩니다.

## `timeInForce`

유효 주문 유형.

`GTC`: 취소될 때까지 유효합니다. 주문은 취소될 때까지 계속 유효합니다.

`IOC`: 즉시 체결 또는 취소. 주문은 최적의 가격으로 가능한 한 많은 수량을 체결하고, 남은 부분은 자동으로 취소됩니다.

`FOK`: 전량 체결 또는 취소. 주문은 최적의 가격으로 전량 체결되거나 즉시 취소됩니다.

`LIMIT_MAKER`: 주문이 즉시 체결될 경우, 주문이 취소됩니다.

## `orderType`

주문 유형

`LIMIT`: 주문이 지정된 가격(또는 더 나은 가격)으로 체결됩니다.

`STOP`: 가격이 `triggerPrice`(트리거 가격)에 도달하면 주문이 체결됩니다.