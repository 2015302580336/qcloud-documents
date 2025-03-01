## 操作场景

该任务指导您使用 Java 语言，通过应用认证来对您的 API 进行认证管理。

## 操作步骤

1. 在 [API 网关控制台](https://console.cloud.tencent.com/apigateway/index?rid=1)，创建一个 API，选择鉴权类型为“应用认证”（参考 [创建 API 概述](https://cloud.tencent.com/document/product/628/11795)）。
2. 将 API 所在服务发布至发布环境（参考 [服务发布与下线](https://cloud.tencent.com/document/product/628/11809)）。
3. 在控制台 [应用管理](https://console.cloud.tencent.com/apigateway/app) 界面创建应用。
4. 在应用列表中选中已经创建好的应用，单击【绑定API】，选择服务和 API 后单击【提交】，即可将应用与 API 建立绑定关系。
5. 参考 [示例代码](#示例代码)，使用 Java 语言生成签名内容。

## 环境依赖

API 网关提供 JSON 请求方式和 form 请求方式的示例代码，请您根据自己业务的实际情况合理选择。

## 注意事项

- 应用生命周期管理，以及 API 向应用授权、应用绑定 API 等操作请您参考 [应用管理](https://cloud.tencent.com/document/product/628/55087)。
- 应用生成签名过程请您参考 [应用认证方式](https://cloud.tencent.com/document/product/628/55088)。

## 示例代码[](id:示例代码)

### JSON 请求方式示例代码
<dx-codeblock>
:::  java
import org.apache.commons.codec.digest.DigestUtils;

import javax.crypto.Mac;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.net.URI;
import java.net.URL;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.text.SimpleDateFormat;
import java.time.Duration;
import java.util.*;

public class AppAuthJavaDemo {
    private static final String MAC_NAME = "HmacSHA1";
    private static final String ENCODING = "UTF-8";

    private static final HttpClient httpClient = HttpClient.newBuilder()
            .version(HttpClient.Version.HTTP_2)
            .connectTimeout(Duration.ofSeconds(10))
            .build();

    private static String getGMTTime(){
        Calendar cd = Calendar.getInstance();
        SimpleDateFormat sdf = new SimpleDateFormat("EEE, dd MMM yyyy HH:mm:ss 'GMT'", Locale.US);
        sdf.setTimeZone(TimeZone.getTimeZone("GMT"));
        String GMTTime = sdf.format(cd.getTime());
        return GMTTime;
    }

    private static String sortQueryParams(String queryParam){
        // parameters should be in alphabetical order
        if (queryParam == null || queryParam == ""){
            return "";
        }

        String[] queryParams = queryParam.split("&");
        Map<String, String> queryPairs = new TreeMap<>();
        for(String query: queryParams){
            String[] kv = query.split("=");
            queryPairs.put(kv[0], kv[1]);
        }

        StringBuilder sortedParamsBuilder = new StringBuilder();
        Iterator iter = queryPairs.entrySet().iterator();
        while(iter.hasNext()){
            Map.Entry entry = (Map.Entry) iter.next();
            sortedParamsBuilder.append(entry.getKey());
            sortedParamsBuilder.append("=");
            sortedParamsBuilder.append(entry.getValue());
            sortedParamsBuilder.append("&");
        }
        String sortedParams = sortedParamsBuilder.toString();
        sortedParams = sortedParams.substring(0, sortedParams.length() - 1);

        return sortedParams;
    }

    private static byte[] HmacSHA1Encrypt(String encryptText, String encryptKey) throws Exception {
        byte[] data = encryptKey.getBytes(ENCODING);
        SecretKey secretKey = new SecretKeySpec(data, MAC_NAME);
        Mac mac = Mac.getInstance(MAC_NAME);
        mac.init(secretKey);

        byte[] text = encryptText.getBytes(ENCODING);
        return mac.doFinal(text);
    }

    private static String base64Encode(byte[] key) {
        final Base64.Encoder encoder = Base64.getEncoder();
        return encoder.encodeToString(key);
    }

    private static String getMD5(String str) {
        String md5Hex = DigestUtils.md5Hex(str);
        return md5Hex;
    }

    public static void main(String[] args) throws Exception {
        String url = "http://service-xxxxx-xxxxx.bj.apigw.tencentcs.com/appauth?a=1&b=2";
        String host = "http://service-xxxxx-xxxxx.bj.apigw.tencentcs.com";
        String apiAppKey = "<Your AppKey>";
        String apiAppSecret = "<Your AppSecret>";
        String httpMethod = "POST";
        String acceptHeader = "application/json";

        // ContentType and contentMd5 should be empty if request body is not present
        String reqBody = "{\"name\":\"apple\", \"type\":\"fruit\"}";
        String contentType = "application/json";
        String contentMD5 = base64Encode(getMD5(reqBody).getBytes());

        // Parse URL and assemble string to sign
        URL parsedUrl = new URL(url);
        String pathAndParams = parsedUrl.getPath();
        if (parsedUrl.getQuery() != null) {
            pathAndParams = pathAndParams + "?" + sortQueryParams(parsedUrl.getQuery());
        }
        String xDate = getGMTTime();
        String stringToSign = String.format("x-date: %s\n%s\n%s\n%s\n%s\n%s", xDate, httpMethod, acceptHeader, contentType, contentMD5, pathAndParams);

        // Encode string with HMAC and base64
        byte[] hmacStr = HmacSHA1Encrypt(stringToSign, apiAppSecret);
        String signature = base64Encode(hmacStr);
        String authHeader = String.format("hmac id=\"%s\", algorithm=\"hmac-sha1\", headers=\"x-date\", signature=\"%s\"", apiAppKey, signature);

        // Generate request
        HttpRequest request = HttpRequest.newBuilder()
                .POST(HttpRequest.BodyPublishers.ofString(reqBody))
                .uri(URI.create(url))
                .setHeader("Host", host)
                .setHeader("Accept", acceptHeader)
                .setHeader("x-date", xDate)
                .setHeader("Content-Type", contentType)
                .setHeader("Content-MD5", contentMD5)
                .setHeader("Authorization", authHeader)
                .build();

        // Send request
        HttpResponse<String> response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());

        // Receive response
        System.out.println(response.statusCode());
        System.out.println(response.body());
    }
}
:::
</dx-codeblock>



### form 请求方式示例代码

``` java
import javax.crypto.Mac;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.math.BigInteger;
import java.net.URI;
import java.net.URL;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.text.SimpleDateFormat;
import java.time.Duration;
import java.util.*;



public class AppAuthJavaFormDemo {
			private static final String MAC_NAME = "HmacSHA1";
			private static final String ENCODING = "UTF-8";



			private static final HttpClient httpClient = HttpClient.newBuilder()
					.version(HttpClient.Version.HTTP_2)
					.connectTimeout(Duration.ofSeconds(10))
					.build();



			private static String getGMTTime(){
					Calendar cd = Calendar.getInstance();
					SimpleDateFormat sdf = new SimpleDateFormat("EEE, dd MMM yyyy HH:mm:ss 'GMT'", Locale.US);
					sdf.setTimeZone(TimeZone.getTimeZone("GMT"));
					String GMTTime = sdf.format(cd.getTime());
					return GMTTime;
			}



			private static String sortQueryParams(String queryParam){
					// parameters should be in alphabetical order
					if (queryParam == null || queryParam == ""){
						return "";
			}



					String[] queryParams = queryParam.split("&");
					Map<String, String> queryPairs = new TreeMap<>();
					for(String query: queryParams){
						String[] kv = query.split("=");
						queryPairs.put(kv[0], kv[1]);
					}



					StringBuilder sortedParamsBuilder = new StringBuilder();
					Iterator iter = queryPairs.entrySet().iterator();
					while(iter.hasNext()){
						Map.Entry entry = (Map.Entry) iter.next();
						sortedParamsBuilder.append(entry.getKey());
						sortedParamsBuilder.append("=");
						sortedParamsBuilder.append(entry.getValue());
						sortedParamsBuilder.append("&");
					}
					String sortedParams = sortedParamsBuilder.toString();
					sortedParams = sortedParams.substring(0, sortedParams.length() - 1);



					return sortedParams;
			}



			private static byte[] HmacSHA1Encrypt(String encryptText, String encryptKey) throws Exception {
				byte[] data = encryptKey.getBytes(ENCODING);
				SecretKey secretKey = new SecretKeySpec(data, MAC_NAME);
				Mac mac = Mac.getInstance(MAC_NAME);
				mac.init(secretKey);



				byte[] text = encryptText.getBytes(ENCODING);
				return mac.doFinal(text);
			}



			private static String base64Encode(byte[] key) {
				final Base64.Encoder encoder = Base64.getEncoder();
				return encoder.encodeToString(key);
			}



			private static String getMD5(String str) throws NoSuchAlgorithmException {
				MessageDigest md = MessageDigest.getInstance("MD5");
				md.update(str.getBytes());
				return new BigInteger(1, md.digest()).toString(16);
			}



			public static void main(String[] args) throws Exception {
				String url = "http://service-xxxxxx-xxxxxx.bj.apigw.tencentcs.com/appauth?a=1&b=2";
				String host = "http://service-xxxxxx-xxxxxx.bj.apigw.tencentcs.com";
				String apiAppKey = "<Your App Key>";
				String apiAppSecret = "<Your App Secret>";
				String httpMethod = "POST";
				String acceptHeader = "application/json";



				// Parse form data and assemble request body
				Map<String, String> reqBodyMap = new TreeMap<>();
				reqBodyMap.put("type", "fruit");
				reqBodyMap.put("name", "apple");



				StringBuffer reqBodyBuffer = new StringBuffer();
				for (Map.Entry<String, String> e : reqBodyMap.entrySet()) {
					reqBodyBuffer.append(e.getKey());
					reqBodyBuffer.append("=");
					reqBodyBuffer.append(e.getValue());
					reqBodyBuffer.append("&");
				}
				String reqBody = reqBodyBuffer.toString();
				reqBody = reqBody.substring(0, reqBody.length() - 1);



				String contentType = "application/x-www-form-urlencoded";
				String contentMD5 = "";



				// Parse URL and assemble string to sign
				URL parsedUrl = new URL(url);
				String pathAndParams = parsedUrl.getPath();
				if (parsedUrl.getQuery() != null) {
					pathAndParams = pathAndParams + "?" + sortQueryParams(parsedUrl.getQuery());
				}
				if (reqBody != "" && reqBody.length() > 0){
					pathAndParams = pathAndParams + "&" + reqBody;
				}



				String xDate = getGMTTime();
				String stringToSign = String.format("x-date: %s\n%s\n%s\n%s\n%s\n%s", xDate, httpMethod, acceptHeader, contentType, contentMD5, pathAndParams);



				// Encode string with HMAC and base64
				byte[] hmacStr = HmacSHA1Encrypt(stringToSign, apiAppSecret);
				String signature = base64Encode(hmacStr);
				String authHeader = String.format("hmac id=\"%s\", algorithm=\"hmac-sha1\", headers=\"x-date\", signature=\"%s\"", apiAppKey, signature);



				// Generate request
				HttpRequest request = HttpRequest.newBuilder()
						.POST(HttpRequest.BodyPublishers.ofString(reqBody))
						.uri(URI.create(url))
						.setHeader("Host", host)
						.setHeader("Accept", acceptHeader)
						.setHeader("x-date", xDate)
						.setHeader("Content-Type", contentType)
						.setHeader("Content-MD5", contentMD5)
						.setHeader("Authorization", authHeader)
						.build();



				// Send request
				HttpResponse<String> response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());



				// Receive response
				System.out.println(response.statusCode());
				System.out.println(response.body());
			}
}
```
