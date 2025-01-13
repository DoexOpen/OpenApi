# 接口签名

## 凭证携带
> 所有交易接口: <br/>
> `路径参数`中必须含有 `timestamp` 、`signature`; <br/>
> `header`中必须含有 `X-BH-APIKEY`;

名称| 位置     | 类型        | 是否强制       |描述
------------ |--------|-----------|------------| ------------ | 
`timestamp`  | query  | string    | `YES`      |当前UTC时间戳
`signature`  | query  | string    | `YES`      |签名,参见[请求示例](#请求示例)
`X-BH-APIKEY`| header | string    | `YES`      | API密钥, 请联系客服获取


# 请求示例

## java

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
     * 生成签名
     *
     * @param message 原文
     * @param secret  秘钥
     * @return 密文
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
        // 这里，以查询历史成交接口（/exapi/contract/v1/myTrades）为例，演示如何生成签名
        String apiKey = "替换为您的apiKey";
        String secretKey = "替换为您的secretKey";

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

## linux shell

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