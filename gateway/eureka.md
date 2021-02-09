# 注册中心配置

```yaml
    eureka:
      client:
        serviceUrl:
          defaultZone: http://${peer1.host}:${peer1.port}/eureka/, http://${peer2.host}:${peer2.port}/eureka/
        enabled: true
      instance:
        prefer-ip-address: true
```
### 解析

1. serviceUrl.defaultZone -- 注册中心地址
2. instance.prefer-ip-address=true -- 用当前ip注册，默认是hostname注册，原因是无法通过hostname访问其他机器，用ip注册可以通过内网访问其他机器