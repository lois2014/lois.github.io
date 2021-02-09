 ### RequestSize 配置

gateway http服务是NettyReactiveWebSever, 而gateway不支持netty文件配置

__ netty自定义配置 __

```yaml
    nettyserver:
        # 单位是byte
        max-initial-ling-length: 10485760
        max-http-header-size: 1024000
```
```java
package com.ofashion.gateway.config;

import lombok.Data;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.web.embedded.netty.NettyReactiveWebServerFactory;
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.context.annotation.Configuration;

@Data
@Configuration
public class NettyConfig implements WebServerFactoryCustomizer<NettyReactiveWebServerFactory> {

    @Value("${nettyserver.max-initial-ling-length:102400}")
    private int maxInitialLingLength;

    @Value("${nettyserver.max-http-header-size:102400}")
    private int maxHeaderSize;

    @Override
    public void customize(NettyReactiveWebServerFactory factory) {
        factory.addServerCustomizers(httpServer -> httpServer.httpRequestDecoder(httpRequestDecoderSpec ->
                httpRequestDecoderSpec
                        .maxHeaderSize(maxHeaderSize)
                .maxInitialLineLength(maxInitialLingLength)
        ));
    }
}
 ```