访问 Web 资源

目录 访问 Web 资源通过 RestTemplate 访问 Web 资源RestTemplate 的高阶用法简单定制 RestTemplateRestTemplate 支持的 HTTP 库优化底层请求策略通过 WebClient 访问 Web 资源了解 WebClientWebClient 的基本用法SpringBucks 进度小结本章小结SpringBucks 进度小结

---



通过 RestTemplate 访问 Web 资源

-   Spring Boot 中的 RestTemplate
    Spring Boot 中没有自动配置 RestTemplate
    Spring Boot 提供了 RestTemplateBuilder
    RestTemplateBuilder.build()
-   常用方法
    -   GET 请求
        -   getForObject() / getForEntity()
    -   POST 请求
        -   postForObject() / postForEntity()
    -   PUT 请求
        -   put()
    -   DELETE 请求
        -   delete()
-   构造 URI
    -   构造 URI
        -   UriComponentsBuilder
    -   构造相对于当前请求的 URI
        -   ServletUriComponentsBuilder
    -   构造指向 Controller 的 URI
        -   MvcUriComponentsBuilder

---

UriComponenttsBuilder的用法

    URI uri = UriComponentsBuilder.fromUriString("http://localhost:8080/coffee/{id}").build(1);
            ResponseEntity<Coffee> c = restTemplate.getForEntity(uri, Coffee.class);
            log.info("Response status:{},Response Headers:{}", c.getStatusCode(), c.getHeaders().toString());
            log.info("Coffee:{}", c.getBody());



---

RestTemplate 的高阶用法

-   传递 HTTP Header
    -   RestTemplate.exchange()
    -   RequestEntity<T> / ResponseEntity<T>
-   HTTP Header



-   类型转换
    -   JsonSerializer / JsonDeserializer
    -   @JsonComponent
-   解析泛型对象
    -   RestTemplate.exchange()
    -   ParameterizedTypeReference<T>
        
    ---
    使用RequestEntity及ResponseEntity设置请求头以及响应头
        URI nameUri = UriComponentsBuilder.fromUriString("http://localhost:8080/coffee/?name={name}").build("mocha");
                RequestEntity<Void> req = RequestEntity.get(nameUri).accept(MediaType.APPLICATION_XML).build();
                ResponseEntity<String> resp = restTemplate.exchange(req, String.class);
                log.info("Response Status:{},Respense Headers:{}", resp.getStatusCode(), resp.getHeaders());
                log.info("Coffee:{}", resp.getBody());
    使用ParameterizedTypeReference来解析泛型对象，直接使用泛型对象来接收会报类型转换错误，LinkedHashMap can not be casted to target Type
        ParameterizedTypeReference<List<Coffee>> listRespP = new ParameterizedTypeReference<List<Coffee>>() {};
                ResponseEntity<List<Coffee>> listResp = restTemplate.exchange(coffeeUri, HttpMethod.GET, null, listRespP);
                listResp.getBody().forEach(coffee -> log.info("Coffee {}", coffee));
    ---

简单定制 RestTemplate

RestTemplate 支持的 HTTP 库

-   通⽤接口
    -   ClientHttpRequestFactory
-   默认实现
    -   SimpleClientHttpRequestFactory
-   Apache HttpComponents
    -   HttpComponentsClientHttpRequestFactory
-   Netty 支持底层netty
    -   Netty4ClientHttpRequestFactory
-   OkHttp 安卓使用
    -   OkHttp3ClientHttpRequestFactory

优化底层请求策略

-   连接管理
    -   PoolingHttpClientConnectionManager
    -   KeepAlive 策略 ，保持连接时间设置
-   超时设置
    -   connectTimeout / readTimeout
-   SSL校验
-   证书检查策略
-   连接复⽤
    

KeepAlive 默认实现

    org.apache.http.impl.client.DefaultConnectionKeepAliveStrategy  默认为-1，即永久有效

---

自定义ConnectionKeepAliveStrategy

    public class CustomConnectionKeepAliveStrategy implements ConnectionKeepAliveStrategy {
    
        private static final long DEFAULT_SECONDS = 30;
    
        @Override
        public long getKeepAliveDuration(HttpResponse response, HttpContext context) {
            return Arrays.stream(response.getHeaders(HTTP.CONN_KEEP_ALIVE))
                .filter(header -> StringUtils.equalsIgnoreCase(header.getName(), "timeout")
                    && StringUtils.isNumeric(header.getValue()))
                .findFirst()
                .map(header -> NumberUtils.toLong(header.getValue(), DEFAULT_SECONDS))
                .orElse(DEFAULT_SECONDS) * 1000;
        }
    }

初始化HttpComponentsClientHttpRequestFactory，并设置自定义的keep-alive策略

     @Bean
        public HttpComponentsClientHttpRequestFactory requestFactory() {
            PoolingHttpClientConnectionManager connectionManager =
                new PoolingHttpClientConnectionManager(30, TimeUnit.SECONDS);
            connectionManager.setMaxTotal(200);
            connectionManager.setDefaultMaxPerRoute(20);
    
            CloseableHttpClient httpClient = HttpClients.custom()
                .setConnectionManager(connectionManager)
                .evictIdleConnections(30, TimeUnit.SECONDS)
                .disableAutomaticRetries()
                // DefaultConnectionKeepAliveStrategy keepAlive中有值就会取一个值，没有的话就永久有效
                // .setKeepAliveStrategy(DefaultConnectionKeepAliveStrategy.INSTANCE)
                .setKeepAliveStrategy(new CustomConnectionKeepAliveStrategy())
                .build();
            return new HttpComponentsClientHttpRequestFactory(httpClient);
        }

---



通过 WebClient 访问 Web 资源

了解 WebClient

-   WebClient
    -   一个以 Reactive ⽅式处理 HTTP 请求的⾮阻塞式的客户端
-   ⽀持的底层 HTTP 库
    -   Reactor Netty - ReactorClientHttpConnector

-   Jetty ReactiveStream HttpClient - JettyClientHttpConnector

WebClient 的基本用法

-   创建 WebClient
    -   WebClient.create()
    -   WebClient.builder()
-   发起请求
    -   get() / post() / put() / delete() / patch()
-   获得结果
    -   retrieve() / exchange()
-   处理 HTTP Status
    -   onStatus()
-   应答正⽂
    -   bodyToMono() / bodyToFlux()

---



---

SpringBucks 进度小结

本章小结

-   RestTemplate 的各种⽤法
-   RestTemplate 的简单定制
-   WebClient 的基本⽤法

SpringBucks 进度小结

-   增加了 customer-service
    -   通过编码方式查询咖啡
    -   通过编码方式创建订单

---

---


