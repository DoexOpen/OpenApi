# 인터페이스 서명

## 자격 증명 포함
> 모든 거래 인터페이스: <br/>
> `경로 매개변수`에 `timestamp` 및 `signature`가 포함되어야 합니다; <br/>
> `헤더`에 `X-BH-APIKEY`가 포함되어야 합니다;

이름 | 위치 | 유형 | 필수 여부 | 설명
------------ |--------|-----------|------------| ------------
`timestamp`  | query  | string    | `YES`      | 현재 UTC 타임스탬프
`signature`  | query  | string    | `YES`      | 서명, 요청 예제 참조
`X-BH-APIKEY`| header | string    | `YES`      | API 키, [사용자 센터](https://www.doex.com/user/service_api)에서 가져오기

# 요청 예제

## 자바

```java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import javax.xml.bind.DatatypeConverter;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;

public class SignatureHelper {
    /**
     * 서명 생성
     *
     * @param message 원문
     * @param secret  비밀 키
     * @return 암호문
     */
    public static String sign(String message, String secret) {
        try {
            Mac sha256_HMAC = Mac.getInstance("HmacSHA256");
            SecretKeySpec secretKeySpec = new SecretKeySpec(secret.getBytes(), "HmacSHA256");
            sha256_HMAC.init(secretKeySpec);
            byte[] hash = sha256_HMAC.doFinal(message.getBytes());
            return DatatypeConverter.printHexBinary(hash).toLowerCase();
        } catch (Exception e) {
            throw new RuntimeException("Unable to sign message.", e);
        }
    }

    public static void main(String[] args) throws IOException {
        // 여기서는 거래 내역 조회 인터페이스(/exapi/contract/v1/myTrades)를 예로 들어 서명을 생성하는 방법을 설명합니다.
        String apiKey = "여기에 귀하의 apiKey를 입력하세요";
        String secretKey = "여기에 귀하의 secretKey를 입력하세요";

        String urlPrefix = "https://exapi.doex.com/exapi/contract/v1/myTrades";

        StringBuilder queryParams = new StringBuilder();
        queryParams.append("symbol=BTC-SWAP-USDT&limit=1&fromId=0&toId=0");
        queryParams.append("&timestamp=").append(System.currentTimeMillis());

        String signature = sign(queryParams.toString(), secretKey);
        queryParams.append("&signature=").append(signature);

        URL url = new URL(urlPrefix + "?" + queryParams.toString());
        HttpURLConnection connection = (HttpURLConnection) url.openConnection();
        connection.setRequestMethod("GET");
        connection.setRequestProperty("X-BH-APIKEY", apiKey);

        int responseCode = connection.getResponseCode();
        System.out.println("Response Code: " + responseCode);

        try (BufferedReader in = new BufferedReader(new InputStreamReader(connection.getInputStream()));) {
            String inputLine;
            StringBuilder response = new StringBuilder();

            while ((inputLine = in.readLine()) != null) {
                response.append(inputLine);
            }
            System.out.println("Response Body: " + response.toString());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## 리눅스 쉘

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

