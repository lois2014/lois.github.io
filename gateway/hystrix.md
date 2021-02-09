# Hystrix配置
完整配置: [Hystrix Configuration](https://github.com/Netflix/Hystrix/wiki/Configuration#coreSize)
#### 线程池配置
##### 默认配置
```yaml
hystrix:
  threadpool:
    default:
      # 核心线程数
      coreSize: 10
      # 最大线程数
      maximumSize: 10
      # 待处理队列数， 队列数要小于 queueSizeRejectionThreshold ， maximumSize才生效
      maxQueueSize: -1
      # 队列满后拒绝
      queueSizeRejectionThreshold: 5
      # 设置为true, maximumSize才生效
      allowMaximumSizeToDivergeFromCoreSize: false

```
###### 配置说明
|  配置 | 描述  | 默认值  | 备注 |
|  :----  | :---- | :----  | :---- |
| coreSize  | 线程核心数 | 10 |  |
| maximumSize  | 最大线程数 | 10 |   |
| allowMaximumSizeToDivergeFromCoreSize | 使maximumSize生效 | false | 1.5.9版本新增属性，只有设置为true，maximumSize才会生效，以前的旧版本coreSize与maximumSize恒相等， |
| maxQueueSize | 最大阻塞队列 | -1 | -1：SynchronousQueue(同步队列), >= 0：LinkedBlockingQueue(阻塞队列) | 
| queueSizeRejectionThreshold | 动态的阻塞队列 | 5 | 只有maxQueueSize >= 0 时才有效，可动态控制阻塞队列，当队列满了，直接拒绝请求，即使阻塞量还没到maxQueueSize |      


###### 配置详解
* hystrix.threadpool.*HystrixThreadPoolKey*  
  - 默认：hystrix.threadpool.default
  - *HystrixThreadPoolKey*是自定义threadpoolkey, 当前gateway不支持线程池自定义配置,可重写HystrixGatewayFilterFactory
  - 设置HystrixCommandGroupKey, 相当于设置*HystrixThreadPoolKey*
```java
public class CustomHystrixGatewayFilterFactory extends AbstractGatewayFilterFactory<CustomHystrixGatewayFilterFactory.Config> {
        ......
        @Override
        public GatewayFilter apply(Config config) {
            if (config.setter == null) {
                //设置groupKey
                HystrixCommandGroupKey groupKey = HystrixCommandGroupKey.Factory.asKey(Optional.of(config.groupKey).orElse(config.name));
                HystrixCommandKey commandKey = HystrixCommandKey.Factory.asKey(config.name);
                config.setter = HystrixObservableCommand.Setter.withGroupKey(groupKey).andCommandKey(commandKey);
            }
            ......
        }

        @Getter
        public static class Config {
            private String groupKey;
            
            public CustomHystrixGatewayFilterFactory.Config setGroupKey(String name) {
                this.groupKey = name;
                return this;
            }
            ......
        }       
        ......
}
```
* 线程数计算公式: rps(Requests per second) * 99%请求的秒数 + 缓冲量

* coreSize: 线程核心数，正常情况下10足够，因为大多数情况下只有1～2个线程在使用

* maximumSize: 1.5.9之前版本是恒等于coreSize, 1.5.9版本及其之后的版本新增allowMaximumSizeToDivergeFromCoreSize，设置为true, 同时 maxQueueSize > 0 && maxQueueSize < queueSizeRejectionThreshold 才生效

* queueSizeRejectionThreshold：这个参数只有maxQueueSize > 0才生效，同样是阻塞队列，maxQueueSize只有重新初始化hystrix线程才能生效，而queueSizeRejectionThreshold动态修改配置即可生效


## Command配置
```yaml
hystrix:
  command:
    fallbackCmd:
      execution:
        isolation:
          strategy: THREAD
          thread:
            timeoutInMilliseconds: 60000
```

|  参数   | 描述  | 默认值  | 备注 |
|  ----  | -----  | ----  |
| execution.isolation.strategy  | 线程池隔离策略 | THREAD | THREAD - 线程，SEMAPHORE - 信号量  |
| execution.isolation.thread.timeoutInMilliseconds|  超时时间,毫秒级  | 3000 | 请求超时即会熔断  |

1. hystrix.command.fallbackCmd: "fallbackCmd"是自定义 *HystrixCommandKey*，可参见[路由配置](route.md)的hystrix的name

2. hystrix.command.*HystrixCommandKey*.execution.isolation.strategy: THREAD - 线程隔离, 通过线程数限制并发量, 默认是THREAD。 SEMAPHORE - 信号量隔离，线程的信号量的数量控制并发量。

  线程隔离需要大量的线程资源支持一定的并发量，而信号量隔离可极大降低线程资源，但是信号量隔离只适合no-network，比如socket访问，请求毫秒以下
  


