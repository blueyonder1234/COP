[Chinese Version](/README_zh.md)

# 0x01 COP #

**Cosco shipping lines Open-api Platform**：Currently COP is in **trial operation**!!! Free data service is available during the commissioning period.

COSCO Shipping's Open API is mainly based on container transportation business, extending to the upstream and downstream of the supply chain, front and back end, on the one hand, serving traditional transport customers, customizing information solutions for industry customers, deepening and customer information cooperation, and enhancing service viscosity On the other hand, through the establishment of a rich and comprehensive supply chain and e-commerce API system, and even allow third parties (independent developers, industry solutions providers, customers) based on our API system for customized development, to promote the logistics information platform ecological construction.

## COP Customer Application ##
In order to ensure the security and privacy of user data, COP's customer applications ("Application" or "Consumer") need to go through a certain business application and review process before they can be authorized to access to the COP platform. Each application will be assigned a set of apiKey and secretKey as the identification credentials for application, and developers must keep apiKey and secretKey. ApiKey and secretKey in the production formal environment will serve as **the only credentials** for cop customer applications.



# 0x02 public API Service #
The overall technology is also different depending on the external API needs and patterns. The external API pattern is divided into two types:

## HTTP(S) protocol based ##

Serves synchronous and asynchronous calls:
![Standard or customized synchronous API](https://github.com/cop-cos/COP/blob/master/docs/images/overview_001.png)

![Customized synchronous API](https://github.com/cop-cos/COP/blob/master/docs/images/overview_002.png)

## MQ protocol based ##

Service asynchronous calls, only for deep lyced application scenarios, for MQ's security management, end-to-end MQ protocol network, etc. there are requirements;
![Customized synchronous API](https://github.com/cop-cos/COP/blob/master/docs/images/overview_003.png)



# 0x03 Environment Instructions #

|Environment|Service Address(HTTPS)|
|---| :---: |
|PRODUCTION|https://api.lines.coscoshipping.com/service|
|TEST|https://api-pp.lines.coscoshipping.com/service|

    Note: The URL in all subsequent API manifests refers to the path relative to the service address. 

## PRODUCTION ##

COSCO Shipping COP production formal environment refers to COSCO Shipping COP platform to the real customers, partners and independent software developers of the official production and operation of the environment. The data is real data, the production of formal environment apiKey and secretKey is the only credentials for the customer's application, need to be safely stored, the customer application is responsible for its behavior and data on the COP platform.

## TEST ##

TBD.



# 0x04 Residency and Confirmation #

## Residency - Become a Developer ##

* During trial operation 

Send your application form email to：[COP Administration Team](mailto:MicroService_Mgmt@coscon.com)

Email Subject: COP developer residency application-<Your Company Name>

|Item|Description|
|------------|:-----------------|
|Contact name:	 |    |
|Contact telephone |    |
|Email address:|    |
|Company name:|    |
|My Role:|1. Customer; 2. Agent; 3. Logisitic Partner; 4. Software service vendor; 5. Information Integration Vendor;|
|Usage:|    |

* During official operation: **TODO**

## Confirmation ##
[COP Administration Team](mailto:MicroService_Mgmt@coscon.com)The review will be evaluated according to the residency application and the review will be fed back within 15 working days.

## Issuance of Crenditials ##
The COP Platform administration team (mailto:MicroService_Mgmt@coscon.com) will send your credentials ApiKey/SecretKey to your email address after your residency application confirmed. In the event of a SecretKey leak, be sure to contact the COP Platform Operations Team (mailto:MicroService_Mgmt@coscon.com) ASAP.

## Limitations and constraints
Based on the anti-Deny-of-Service, performance and API features, the size of each http request's body (HTTP Request Body) must not exceed 1MB.


# 0x05 Security #

## About SSL Certificate##

You may experience https/ssl certificate trust issues while you are using it. It is recommended that you download a server-side certificate file through your browser and load it into a trusted certificate library.
```
keytool -import -trustcacerts -alias cop -keystore "%JAVA_HOME%/JRE/LIB/SECURITY/CACERTS" -file ./api.lines.coscoshipping.com.cer -storepass changeit
```

## Hmac Auth ##

The COP platform publishes a pair of **App Key** and **Secret Key** for each Application to identify Application. The COP platform assigns access to the API based on the application and business requirements.


**Hmac Auth system uses API Key, Secret Key, abstract and other technologies, for the user access to URI address and request message for service-side verification, high security, performance overhead slightly higher.

### Java Sample 1 ###
[Hmac](https://github.com/cop-cos/COP/blob/master/openapi-client-pure/src/main/java/com/coscon/oaclient/pure/HmacPureExecutor.java) 

* Setup ApiKey and SecretKey

```java
    //com.coscon.oaclient.pure.HmacPureExecutor
    hmacPureExecutor = new HmacPureExecutor();
    hmacPureExecutor.setApiKey("YOUR_API_KEY");
    hmacPureExecutor.setSecretKey("YOUR_SECRET_KEY");
```

* Setup HTTP Headers
  
```java
    Map<String, String> headers = getHmacPureExecutor().buildHmacHeaders(request.getRequestLine().toString(), httpContent);
    if(headers!=null) {
        for(Entry<String, String> e:headers.entrySet()) {
            request.addHeader(e.getKey(), e.getValue());
        }
    }
```

### Java Sample 2 - HttpClient ### 
 [HttpClient Sample](https://github.com/cop-cos/COP/blob/master/openapi-client-httpclient/src/test/java/com/coscon/openapi/client/httpclient/CargoTrackingTestcase.java)
 
    com.coscon.openapi.client.httpclient.CargoTrackingTestcase

* Setup ApiKey and SecretKey

```java
    /*com.coscon.openapi.client.httpclient.AbstractOpenapiTestcase#setUp*/
    hmacPureExecutor = new HmacPureExecutor();
    hmacPureExecutor.setApiKey("YOUR_API_KEY");
    hmacPureExecutor.setSecretKey("YOUR_SECRET_KEY");
```

* HttpClientBuilder - add an HttpRequestInterceptor to handle HTTP(S) Security purpose
  
```java
    HttpClientBuilder builder = HttpClientBuilder.create();
    builder.addInterceptorLast(new HttpRequestInterceptor() {

        @Override
        public void process(HttpRequest request, HttpContext context) throws HttpException, IOException {
            if(!match(request, hostPatterns)) {
                return;
            }
            byte[] httpContent = new byte[0];
            if (request instanceof HttpEntityEnclosingRequest) {
				HttpEntity entity = ((HttpEntityEnclosingRequest) request).getEntity();
				if(entity != null) {
					httpContent = IOUtils.toByteArray(entity.getContent());
				}
            }
            try {
                Map<String, String> headers = getHmacPureExecutor().buildHmacHeaders(request.getRequestLine().toString(), httpContent);
                if(headers!=null) {
                    for(Entry<String, String> e:headers.entrySet()) {
                        request.addHeader(e.getKey(), e.getValue());
                    }
                }
            } catch (OpenClientSecurityException e) {
                e.printStackTrace();
            }
        }
    });
```



# 0x06 Global Codes #
Detail:[Global Codes](https://github.com/cop-cos/COP/blob/master/GlobalCodes.md)



# 0x07 API List #

| Customer Group           | Module           | Service        | Document | Traffic Control Policy|
| ------------ | ------------ | ---------- | :-------: | :-------: |
| 公共组   | 公共查询        | 货物跟踪 |[说明 doc](https://github.com/cop-cos/COP/blob/master/docs/info-cargotracking.md)| **账号级别**，每天至多1000次，每月至多30000次|
|      |         | 船期查询 |[说明 doc](https://github.com/cop-cos/COP/blob/master/docs/info-schedule.md)| **账号级别**，每天至多1000次，每月至多30000次|
|      | 内贸服务        | 订舱确认书下载 |[说明 doc](https://github.com/cop-cos/COP/blob/master/docs/dts/bookingConfirmDownload.md)| **账号级别**，每天至多1000次，每月至多30000次|
|      |         | 签收单链接查询 |[说明 doc](https://github.com/cop-cos/COP/blob/master/docs/dts/cargoReceiptLink.md)| **账号级别**，每天至多1000次，每月至多30000次|
|      |         | 运单下载 |[说明 doc](https://github.com/cop-cos/COP/blob/master/docs/dts/waybillDownload.md)| **账号级别**，每天至多1000次，每月至多30000次|
|      |         | 订单信息查询 |[说明 doc](https://github.com/cop-cos/COP/blob/master/docs/dts/waybillInfo.md)| **账号级别**，每天至多1000次，每月至多30000次|
|      |         | 自动派车 |[说明 doc](https://github.com/cop-cos/COP/blob/master/docs/dts/autoDispatchToFleet.md)| **账号级别**，每天至多1000次，每月至多30000次|
|      |         | 提箱校验码 |[说明 doc](https://github.com/cop-cos/COP/blob/master/docs/dts/pickupItemCheckCode.md)| **账号级别**，每天至多1000次，每月至多30000次|


# 0x08 Agreements #
When you apply for residency, you are deemed to have agreed to the following agreements:

|Agreements|
|---|
| [中远海运集运开放平台合作伙伴开发协议.docx](https://github.com/cop-cos/COP/blob/master/docs/agreements/中远海运集运开放平台合作伙伴开发协议.docx)|
| [中远海运集运开放平台合作伙伴应用安全规范.docx](https://github.com/cop-cos/COP/blob/master/docs/agreements/中远海运集运开放平台合作伙伴应用安全规范.docx)|
| [中远海运集运开放平台运营规则.docx](https://github.com/cop-cos/COP/blob/master/docs/agreements/中远海运集运开放平台运营规则.docx)|
