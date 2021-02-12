#### databuffer maxsize 配置
```yaml
spring:
  codec:
    max-in-memory-size: 2MB # dataBuffer maxSize default 262144 Bytes
```

> 参考：
> 1. [spring issue](https://github.com/spring-projects/spring-boot/issues/24417)
>
> 2. org.springframework.cloud.gateway.handler.predicate.ReadBodyPredicateFactory

> fix: org.springframework.core.io.buffer.DataBufferLimitException: Exceeded limit on max bytes to buffer : 262144
> [详解](readbody.md)

