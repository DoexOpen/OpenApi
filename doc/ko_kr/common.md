# 공용 Broker Rest API (2025-01-01)

## 일반 API 정보

* 모든 엔드포인트는 JSON 객체 또는 배열을 반환합니다.
* 데이터는 **오름차순**으로 반환됩니다. 오래된 것이 앞에, 최신 것이 뒤에 있습니다.
* 모든 시간/타임스탬프 관련 변수는 밀리초 단위입니다.
* HTTP `4XX` 오류 코드는 요청 내용에 오류가 있음을 나타내며, 이는 요청 발신자 측의 문제입니다.
* HTTP `429` 오류 코드는 요청 횟수 제한을 초과했음을 나타냅니다.
* HTTP `418` 오류 코드는 IP가 `429` 오류 코드를 받은 후에도 계속 요청을 보낼 때 자동으로 차단되었음을 나타냅니다.
* HTTP `5XX` 오류 코드는 내부 시스템 오류를 나타내며, 이는 브로커 측의 문제입니다. 이 오류를 처리할 때 **절대** 이를 실패한 작업으로 간주하지 마십시오. 실행 상태가 **알 수 없기** 때문에 성공했을 수도 있고 실패했을 수도 있습니다.
* 모든 엔드포인트는 ERROR(오류)를 반환할 수 있습니다. 오류 반환 페이로드는 다음과 같습니다.

```javascript
{
  "code": -1121,
  "msg": "Invalid symbol."
}
```

* 자세한 오류 코드와 오류 메시지는 오류 코드 파일을 참조하십시오.
* `GET` 엔드포인트의 경우, 매개변수는 `query string`(쿼리 문자열)으로 전송해야 합니다.
* `POST`, `PUT`, 및 `DELETE` 엔드포인트의 경우, 매개변수는 `query string`(쿼리 문자열) 또는 `request body`(요청 본문)로 전송하고 content type(콘텐츠 유형)을 `application/x-www-form-urlencoded`로 설정해야 합니다. 필요에 따라 `query string`과 `request body`에 혼합하여 매개변수를 전송할 수 있습니다.
* 매개변수는 임의의 순서로 전송할 수 있습니다.
* `query string`과 `request body`에 동시에 매개변수가 존재하는 경우, `query string`의 매개변수만 사용됩니다.

### 제한 사항

* `/exapi/v1/brokerInfo`의 `rateLimits` 배열에는 현재 브로커의 `REQUEST_WEIGHT` 및 `ORDER` 빈도 제한이 포함되어 있습니다.
* 빈도 제한을 초과하면 `429`가 반환됩니다.
* 각 라인에는 `weight` 특성이 있으며, 이는 이 요청이 얼마나 많은 용량을 차지하는지를 결정합니다(예: `weight`=2는 이 요청이 두 개의 요청 용량을 차지함을 의미합니다). 데이터가 많은 엔드포인트 또는 여러 심볼에서 작업을 수행하는 엔드포인트는 더 높은 `weight`를 가질 수 있습니다.
* `429`가 반환되면 요청을 중지해야 합니다.
* **빈도 제한을 반복적으로 위반하거나 `429`를 받은 후에도 요청을 중지하지 않는 사용자는 IP 차단(오류 코드 418)을 받을 수 있습니다.**
* IP 차단은 추적되며 **차단 기간이 조정**됩니다(반복적으로 규정을 위반하는 사용자의 경우, 기간은 **2분에서 3일 사이**입니다).

### 엔드포인트 보안 유형

* 각 엔드포인트에는 보안 유형이 있으며, 이는 해당 엔드포인트와 상호 작용하는 방식을 결정합니다.
* API 키는 `X-BH-APIKEY`라는 이름으로 REST API 헤더에 전달해야 합니다.
* API 키와 비밀 키는 **대소문자를 구분**해야 합니다.
* 기본적으로 API 키는 모든 보안 노드에 액세스할 수 있습니다.

보안 유형 | 설명
------------ | ------------
NONE | 엔드포인트에 자유롭게 액세스할 수 있습니다.
TRADE | 엔드포인트에 유효한 API 키와 서명을 전송해야 합니다.
USER_DATA | 엔드포인트에 유효한 API 키와 서명을 전송해야 합니다.
USER_STREAM | 엔드포인트에 유효한 API 키를 전송해야 합니다.
MARKET_DATA | 엔드포인트에 유효한 API 키를 전송해야 합니다.

* `TRADE` 및 `USER_DATA` 엔드포인트는 `SIGNED`(서명 필요) 엔드포인트입니다.

### SIGNED(서명 필요) (TRADE 및 USER_DATA) 엔드포인트 보안

* `SIGNED`(서명 필요) 엔드포인트는 `query string` 또는 `request body`에 `signature` 매개변수를 전송해야 합니다.
* 엔드포인트는 `HMAC SHA256`으로 서명됩니다. `HMAC SHA256 서명`은 키를 `HMAC SHA256`으로 암호화한 결과입니다. `secretKey`를 키로 사용하고 `totalParams`를 값으로 사용하여 이 암호화 과정을 완료합니다.
* `signature`는 **대소문자를 구분하지 않습니다**.
* `totalParams`는 `query string`과 `request body`를 연결한 것입니다.

### 시간 보안

* `SIGNED`(서명 필요) 엔드포인트는 요청이 발행된 시점의 밀리초 단위 타임스탬프인 `timestamp` 매개변수를 전송해야 합니다.
* 추가 매개변수(선택 사항)인 `recvWindow`는 이 요청이 몇 밀리초 동안 유효한지를 나타낼 수 있습니다. `recvWindow`가 전송되지 않으면 **기본값은 5000**입니다.
* 현재로서는 주문을 생성할 때만 `recvWindow`를 사용합니다.
* 이 매개변수의 논리는 다음과 같습니다:

  ```javascript
  if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
    // 요청 처리
  } else {
    // 요청 거부
  }
  ```

**엄격한 거래는 시간과 밀접하게 관련되어 있습니다.** 네트워크는 때때로 불안정하거나 신뢰할 수 없으며, 이는 요청이 서버에 도달하는 시간을 일관되지 않게 만들 수 있습니다.
`recvWindow`를 사용하면 요청이 몇 밀리초 동안 유효한지를 지정할 수 있으며, 그렇지 않으면 서버에서 거부됩니다.

**상대적으로 작은 recvWindow(5000 이하)를 사용하는 것이 좋습니다!**

### SIGNED(서명 필요) 예제 (POST /exapi/v1/order의 경우)

여기에는 Linux `echo`, `openssl`, 및 `curl`을 사용하여 유효한 서명 페이로드를 전송하는 방법에 대한 자세한 예제가 있습니다.

Key | 값
------------ | ------------
apiKey | tAQfOrPIZAhym0qHISRt8EFvxPemdBm5j5WMlkm3Ke9aFp0EGWC2CGM8GHV4kCYW
secretKey | lH3ELTNiFxCQTmi9pPcWWikhsjO04Yoqw3euoHUuOLC3GYBW64ZqzQsiOEHXQS76

매개변수 이름 | 매개변수 값
------------ | ------------
symbol | ETHBTC
side | BUY
type | LIMIT
timeInForce | GTC
quantity | 1
price | 0.1
recvWindow | 5000
timestamp | 1538323200000

#### 예제 1: `queryString`에

* **queryString:** symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000
* **HMAC SHA256 서명:**

```shell
[linux]$ echo -n "symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000" | openssl dgst -sha256 -hmac "lH3ELTNiFxCQTmi9pPcWWikhsjO04Yoqw3euoHUuOLC3GYBW64ZqzQsiOEHXQS76"
(stdin)= 5f2750ad7589d1d40757a55342e621a44037dad23b5128cc70e18ec1d1c3f4c6
```

* **curl 명령:**

```shell
(HMAC SHA256)
[linux]$ curl -H "X-BH-APIKEY: tAQfOrPIZAhym0qHISRt8EFvxPemdBm5j5WMlkm3Ke9aFp0EGWC2CGM8GHV4kCYW" -X POST 'https://$HOST/exapi/v1/order?symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000&signature=5f2750ad7589d1d40757a55342e621a44037dad23b5128cc70e18ec1d1c3f4c6'
```

#### 예제 2: `request body`에

* **requestBody:** symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000
* **HMAC SHA256 서명:**

```shell
[linux]$ echo -n "symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000" | openssl dgst -sha256 -hmac "lH3ELTNiFxCQTmi9pPcWWikhsjO04Yoqw3euoHUuOLC3GYBW64ZqzQsiOEHXQS76"
(stdin)= 5f2750ad7589d1d40757a55342e621a44037dad23b5128cc70e18ec1d1c3f4c6
```

* **curl 명령:**

```shell
(HMAC SHA256)
[linux]$ curl -H "X-BH-APIKEY: tAQfOrPIZAhym0qHISRt8EFvxPemdBm5j5WMlkm3Ke9aFp0EGWC2CGM8GHV4kCYW" -X POST 'https://$HOST/exapi/v1/order' -d 'symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000&signature=5f2750ad7589d1d40757a55342e621a44037dad23b5128cc70e18ec1d1c3f4c6'
```

#### 예제 3: `queryString`과 `request body` 혼합

* **queryString:** symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC
* **requestBody:** quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000
* **HMAC SHA256 서명:**

```shell
[linux]$ echo -n "symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTCquantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000" | openssl dgst -sha256 -hmac "lH3ELTNiFxCQTmi9pPcWWikhsjO04Yoqw3euoHUuOLC3GYBW64ZqzQsiOEHXQS76"
(stdin)= 885c9e3dd89ccd13408b25e6d54c2330703759d7494bea6dd5a3d1fd16ba3afa
```

* **curl 명령:**

```shell
(HMAC SHA256)
[linux]$ curl -H "X-BH-APIKEY: tAQfOrPIZAhym0qHISRt8EFvxPemdBm5j5WMlkm3Ke9aFp0EGWC2CGM8GHV4kCYW" -X POST 'https://$HOST/exapi/v1/order?symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC' -d 'quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000&signature=885c9e3dd89ccd13408b25e6d54c2330703759d7494bea6dd5a3d1fd16ba3afa'
```

***예제 3에서는 "GTC"와 "quantity=1" 사이에 &가 없다는 점에 유의하십시오.***

## 공용 API 엔드포인트

### 용어 설명

* `base asset`는 심볼의 `quantity`(즉, 수량)를 나타냅니다.

* `quote asset`는 심볼의 `price`(즉, 가격)를 나타냅니다.

### ENUM 정의

**심볼 상태:**

* TRADING - 거래 중
* HALT - 중지
* BREAK - 중단

**심볼 유형:**

* SPOT - 현물

**자산 유형:**

* CASH - 현금
* MARGIN - 마진

**주문 상태:**

* NEW - 새 주문, 아직 체결되지 않음
* PARTIALLY_FILLED - 부분 체결
* FILLED - 완전 체결
* CANCELED - 취소됨
* PENDING_CANCEL - 취소 대기 중
* REJECTED - 거부됨

**주문 유형:**

* LIMIT - 지정가 주문
* MARKET - 시장가 주문
* LIMIT_MAKER - 메이커 지정가 주문
* STOP_LOSS (현재 사용 불가) - 사용 불가
* STOP_LOSS_LIMIT (현재 사용 불가) - 사용 불가
* TAKE_PROFIT (현재 사용 불가) - 사용 불가
* TAKE_PROFIT_LIMIT (현재 사용 불가) - 사용 불가
* MARKET_OF_PAYOUT (현재 사용 불가) - 사용 불가

**주문 방향:**

* BUY - 매수 주문
* SELL - 매도 주문

**주문 유효 기간 유형:**

* GTC
* IOC
* FOK

**k선/캔들스틱 차트 간격:**

m -> 분; h -> 시간; d -> 일; w -> 주; M -> 월

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

**빈도 제한 유형 (rateLimitType)**

* REQUESTS_WEIGHT
* ORDERS

**빈도 제한 간격**

* SECOND
* MINUTE
* DAY




### 일반 엔드포인트

#### Broker 정보

```shell
GET /exapi/v1/brokerInfo
```

현재 broker의 거래 규칙 및 심볼 정보

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
  "brokerFilters":[],
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

### 시장 데이터 엔드포인트

#### 주문서

```shell
GET /exapi/quote/v1/depth
```

**Weight:**

limit에 따라 다름:

Limit | Weight
------------ | ------------
5, 10, 20, 50, 100 | 1
500 | 5
1000 | 10

**Parameters:**

이름 | 유형 | 필수 여부 | 설명
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | 기본값 100; 최대 100.

**주의:** limit=0으로 설정하면 많은 데이터를 반환합니다.

**Response:**

[가격, 수량]

```javascript
{
  "bids": [
    [
      "3.90000000",   // 가격
      "431.00000000"  // 수량
    ],
    [
      "4.00000000",
      "431.00000000"
    ]
  ],
  "asks": [
    [
      "4.00000200",  // 가격
      "12.00000000"  // 수량
    ],
    [
      "5.10000000",
      "28.00000000"
    ]
  ]
}
```

#### 최근 거래

```shell
GET /exapi/quote/v1/trades
```

현재 최신 거래(최대 60개) 가져오기

**Weight:**
1

**Parameters:**

이름 | 유형 | 필수 여부 | 설명
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | 기본값 60; 최대 60.

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

#### k선/캔들스틱 차트 데이터

```shell
GET /exapi/quote/v1/klines
```

심볼의 k선/캔들스틱 차트 데이터
K선은 개장 시간에 따라 구분됩니다.

**Weight:**
1

**Parameters:**

이름 | 유형 | 필수 여부 | 설명
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
interval | ENUM | YES |
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | 기본값 500; 최대 1000.

* startTime과 endTime이 전송되지 않으면 최신 K선만 반환됩니다.

**Response:**

```javascript
[
  [
    1499040000000,      // 개장 시간
    "0.01634790",       // 개장가
    "0.80000000",       // 최고가
    "0.01575800",       // 최저가
    "0.01577100",       // 종가
    "148976.11427815",  // 거래량
    1499644799999,      // 종가 시간
    "2434.19055334",    // Quote asset 수량
    308,                // 거래 횟수
    "1756.87402397",    // Taker buy base asset 수량
    "28.46694368"       // Taker buy quote asset 수량
  ]
]
```