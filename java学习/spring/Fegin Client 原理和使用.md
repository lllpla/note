# Fegin Client 原理和使用

## 一、原理

Feign是一个java到http的客户端绑定器，灵感来源于Retrofit和JAXRS-2.0以及WebSocket。Feign的第一个目标是降低将Denominator无变化的绑定到HTTP APIs的复杂性，而不考虑Restfulness。

Feign使用Jersy和CXF等工具为Rest或者SOAP服务编写JAVA客户端。此外，Feign允许您在ApacheHC等http库之上编写自己的代码。Feign以最小的开销将代码链接到HttpAPIs，并通过可定制的解码器和错误处理（可以写入任何基于文本的httpAPIs）将代码连接到httpAPIs。

Feign通过将注解处理为模板化请求来工作。参数在输出之前直接应用于这些模板。尽管Feign仅限于支持基于文本的APIs，但它极大地简化了系统方面，例如重放请求。此外，Feign使得对转换进行单元测试变得简单。

Feign10.x及以上版本是在java8上构建的。对于需要使用jdk6的用户请使用Feign9.x

## 二、处理过程图

![title](https://raw.githubusercontent.com/lllpla/img/master/gitnote/2020/06/21/1592704630569-1592704630642.png)

## 三、HTTP CLIENT依赖

feign 在默认情况下使用 JDK 原生的 URLConnection 发送HTTP请求。(没有连接池，保持长连接) 。

可以通过修改 client 依赖换用底层的 client，不同的 http client 对请求的支持可能有差异。具体使用示例如下:

```yaml
feign: 
  httpclient:
    enable: false
  okhttp:
    enable: true
```

和

```xml
<!-- Support PATCH Method-->
<dependency>    
  <groupId>org.apache.httpcomponents</groupId>    
  <artifactId>httpclient</artifactId> 
</dependency>
      
<!-- Do not support PATCH Method -->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-okhttp</artifactId>
</dependency>
```

## 四、Http Client 配置

- okhttp 配置源码

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnMissingBean(okhttp3.OkHttpClient.class)
public class OkHttpFeignConfiguration {

 private okhttp3.OkHttpClient okHttpClient;
  
 @Bean
 @ConditionalOnMissingBean(ConnectionPool.class)
 public ConnectionPool httpClientConnectionPool(
   FeignHttpClientProperties httpClientProperties,
   OkHttpClientConnectionPoolFactory connectionPoolFactory) {
  Integer maxTotalConnections = httpClientProperties.getMaxConnections();
  Long timeToLive = httpClientProperties.getTimeToLive();
  TimeUnit ttlUnit = httpClientProperties.getTimeToLiveUnit();
  return connectionPoolFactory.create(maxTotalConnections, timeToLive, ttlUnit);
 }

 @Bean
 public okhttp3.OkHttpClient client(OkHttpClientFactory httpClientFactory,
   ConnectionPool connectionPool,
   FeignHttpClientProperties httpClientProperties) {
  Boolean followRedirects = httpClientProperties.isFollowRedirects();
  Integer connectTimeout = httpClientProperties.getConnectionTimeout();
  this.okHttpClient = httpClientFactory
    .createBuilder(httpClientProperties.isDisableSslValidation())
    .connectTimeout(connectTimeout, TimeUnit.MILLISECONDS)
    .followRedirects(followRedirects).connectionPool(connectionPool).build();
  return this.okHttpClient;
 }

 @PreDestroy
 public void destroy() {
  if (this.okHttpClient != null) {
   this.okHttpClient.dispatcher().executorService().shutdown();
   this.okHttpClient.connectionPool().evictAll();
  }
 }
}
```

* HttpClient 配置源码

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnMissingBean(CloseableHttpClient.class)
public class HttpClientFeignConfiguration {

 private final Timer connectionManagerTimer = new Timer(
   "FeignApacheHttpClientConfiguration.connectionManagerTimer", true);

 private CloseableHttpClient httpClient;

 @Autowired(required = false)
 private RegistryBuilder registryBuilder;

 @Bean
 @ConditionalOnMissingBean(HttpClientConnectionManager.class)
 public HttpClientConnectionManager connectionManager(
   ApacheHttpClientConnectionManagerFactory connectionManagerFactory,
   FeignHttpClientProperties httpClientProperties) {
  final HttpClientConnectionManager connectionManager = connectionManagerFactory
    .newConnectionManager(httpClientProperties.isDisableSslValidation(),
      httpClientProperties.getMaxConnections(),
      httpClientProperties.getMaxConnectionsPerRoute(),
      httpClientProperties.getTimeToLive(),
      httpClientProperties.getTimeToLiveUnit(), this.registryBuilder);
  this.connectionManagerTimer.schedule(new TimerTask() {
   @Override
   public void run() {
    connectionManager.closeExpiredConnections();
   }
  }, 30000, httpClientProperties.getConnectionTimerRepeat());
  return connectionManager;
 }

 @Bean
 @ConditionalOnProperty(value = "feign.compression.response.enabled",
   havingValue = "true")
 public CloseableHttpClient customHttpClient(
   HttpClientConnectionManager httpClientConnectionManager,
   FeignHttpClientProperties httpClientProperties) {
  HttpClientBuilder builder = HttpClientBuilder.create().disableCookieManagement()
    .useSystemProperties();
  this.httpClient = createClient(builder, httpClientConnectionManager,
    httpClientProperties);
  return this.httpClient;
 }

 @Bean
 @ConditionalOnProperty(value = "feign.compression.response.enabled",
   havingValue = "false", matchIfMissing = true)
 public CloseableHttpClient httpClient(ApacheHttpClientFactory httpClientFactory,
   HttpClientConnectionManager httpClientConnectionManager,
   FeignHttpClientProperties httpClientProperties) {
  this.httpClient = createClient(httpClientFactory.createBuilder(),
    httpClientConnectionManager, httpClientProperties);
  return this.httpClient;
 }

 private CloseableHttpClient createClient(HttpClientBuilder builder,
   HttpClientConnectionManager httpClientConnectionManager,
   FeignHttpClientProperties httpClientProperties) {
  RequestConfig defaultRequestConfig = RequestConfig.custom()
    .setConnectTimeout(httpClientProperties.getConnectionTimeout())
    .setRedirectsEnabled(httpClientProperties.isFollowRedirects()).build();
  CloseableHttpClient httpClient = builder
    .setDefaultRequestConfig(defaultRequestConfig)
    .setConnectionManager(httpClientConnectionManager).build();
  return httpClient;
 }

 @PreDestroy
 public void destroy() throws Exception {
  this.connectionManagerTimer.cancel();
  if (this.httpClient != null) {
   this.httpClient.close();
  }
 }
}
```

- HttpClient 配置属性

```java

```