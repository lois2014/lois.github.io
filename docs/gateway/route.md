#### 路由配置
```java
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
            return builder.routes()
                .route(routeId, r ->
                                r.remoteAddr("0.0.0.0/0")
                                        .and().method(HttpMethod.POST).and()
                                        .readBody(Object.class, readBody -> {
                                            return true;
                                        })
                                        .and()
                                        .path("/exampleclient/**")
                                        .filters(f -> f
                                                .hystrix(config -> {
                                                    config.setFallbackUri("forward:/fallback/serviceFailurePage").setName("fallbackCmd");
                                                })
                                                .retry(retryConfig -> {
                                                    retryConfig.setExceptions(java.io.IOException.class, java.util.concurrent.TimeoutException.class)
                                                            .setRetries(3)
                                                            .setSeries(SERVER_ERROR, CLIENT_ERROR);
                                                })
                                                .stripPrefix(1)
                                        )
                                        .uri("lb://example-client")
                )
                .route(routeId, r ->
                                r.remoteAddr("0.0.0.0/0")
                                        .and()
                                        .path("/exampleclient/**")
                                        .filters(f -> f
                                                .hystrix(config -> {
                                                    config.setFallbackUri("forward:/fallback/serviceFailurePage").setName("fallbackCmd");
                                                })
                                                .retry(retryConfig -> {
                                                    retryConfig.setExceptions(java.io.IOException.class, java.util.concurrent.TimeoutException.class)
                                                            .setRetries(3)
                                                            .setSeries(SERVER_ERROR, CLIENT_ERROR);
                                                })
                                                .stripPrefix(1)
                                        )
                                        .uri("lb://example-client")
                                        .order(5)
                )
                .build();
}
```
下发路由基础配置:

1. routeId配置，自定义就行
2. uri("lb://example-client"), uri匹配，lb://开头是指负载均衡（loadbalance）的协议，后面是注册到注册中心的服务名称：application name, 如果是特定uri,可以直接配置uri， 如：http://localhost:8081
3. order(5) 是指路由的匹配顺序

断言配置：

1. method(HttpMethod.POST), 方法路由断言（Method Route Predicate），只有post请求可匹配
2. path("/exampleclient/**"), 路径路由断言（Path Route Predicate），路径正则匹配， 相当于/exampleclient/a/b  或者 /exampleclient/aa?a=ww等前缀是/exampleclient的路由
3. remoteAddr("0.0.0.0/0")， 限制ip只有配置的ip可访问，即ip白名单。参数是CIDR-notation，这个是ip的归类方法，相当于ip段，可配置多个，用逗号","隔开，如0.0.0.0/0代表所有ip段
4. readBody 是修复post请求获取不到requestBody问题，2.x版本以上应该都已经修复

过滤器配置：

1. stripPrefix(1) /exampleclient/example/getInfo/, gateway下发路由就会变成 /example/getInfo，即去掉路由前缀的一个参数，这样配置路由是为了区分多个系统下的下发路由可以相互不影响
2. retry(config -> {}), 当出现某些异常时的重试机制。
    1. 配置参数如下：
        1. retries: 重试次数, 默认值：3
        2. statuses: http status 如 404， 500， 302等，可参考org.springframework.http.HttpStatus 默认值：无
        3. methods: 请求方式，post，get等，可参考org.springframework.http.HttpMethod， 默认值:GET
        4. exceptions: 抛出的异常， 默认值：IOException ， TimeoutException
        5. series: http状态码系列，如 5xx为一类状态码，可参考org.springframework.http.HttpStatus.Series，默认值： 5XX series
        6. backoff: 两个重试发送中间等待配置，如：
           backoff:
              firstBackoff: 10ms //第一次等待时间
              maxBackoff: 50ms //最大等待时机
              factor: 2 // 指数，等待时间指数级增长  
              basedOnPreviousValue: false //是否基于前面等待的时间
6. histrix(config -> {}), hystrix配置熔断，参考[hystrix](hystrix.md)
    配置如下：
        1. 设置指令名 setName(), 相当于HystrixCommandKey
        2. 设置降级uri setFallbackUri("forward:/exampleFallback"), 只要熔断就会定向到匹配"/exampleFallback"的controller 也可以使用lb://xxxa使用负载

#### 另一种配置方式：
##### 配置文件
```yaml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true                         #根据注册中心的每个服务的serviceId，进行路由转发功能，
          #可以做到负载均衡，限流，熔断
      routes:
        - id: goods-center
          uri: lb://goods-center
          predicates:
            - Path=/goodscenter/**
            - RemoteAddr=${locator.ipWhiteList.goods-center}
          filters:
            - StripPrefix=1
            - name: CustomHystrix
              args:
                name: fallbackCmd
                fallbackUri: forward:/fallback/serviceFailurePage?service=goods-center
                groupKey: goodsGroup
            - name: Retry
              args:
                retries: 3
                series: CLIENT_ERROR,SERVER_ERROR
                exceptions: java.io.IOException,java.util.concurrent.TimeoutException
                methods: GET
        - id: transaction-manager
          uri: lb://transaction-manager
          predicates:
            - Path=/transactionmanager/**
            - RemoteAddr=${locator.ipWhiteList.transaction-manager}
          filters:
            - StripPrefix=1
            - name: CustomHystrix
              args:
                name: fallbackCmd
                fallbackUri: forward:/fallback/serviceFailurePage?service=transaction-manager
                groupKey: transactionManager
            - name: Retry
              args:
                retries: 3
                series: CLIENT_ERROR,SERVER_ERROR
                exceptions: java.io.IOException,java.util.concurrent.TimeoutException
                methods: GET
        - id: coupon-center
          uri: lb://coupon-center
          predicates:
            - Path=/couponcenter/**
            - RemoteAddr=${locator.ipWhiteList.coupon-center}
          filters:
            - StripPrefix=1
            - name: CustomHystrix
              args:
                name: fallbackCmd
                fallbackUri: forward:/fallback/serviceFailurePage?service=coupon-center
                groupKey: couponCenter
            - name: Retry
              args:
                retries: 3
                series: CLIENT_ERROR,SERVER_ERROR
                exceptions: java.io.IOException,java.util.concurrent.TimeoutException
                methods: GET

#自定义: ip白名单
# routeId: ips
locator:
  ipWhiteList:
    goods-center: 0.0.0.0/0 #所有ip
    transaction-manager: 0.0.0.0/0 #所有ip
    coupon-center: 0.0.0.0/0 #所有ip

```
