#### readbody -- post方法获取请求参数
源码来着：*org.springframework.cloud.gateway.handler.predicate.ReadBodyPredicateFactory*

> 注意点：
> 1. 只支持post方法
> 2. readbody 默认databuffer maxsize 是 262144b， 所以需要配置spring.codec.max-in-memory-size
以下源码可知：默认是this.messageReaders = HandlerStrategies.withDefaults().messageReaders();
默认的codec的maxSize = 256 * 1024
> 3. 源码查找类顺序大致如下：
> -> org.springframework.web.reactive.function.server.DefaultHandlerStrategiesBuilder
> -> org.springframework.http.codec.support.ServerDefaultCodecsImpl
> -> org.springframework.http.codec.support.BaseDefaultCodecs
> -> org.springframework.core.codec.DataBufferDecoder
> -> org.springframework.core.codec.AbstractDataBufferDecoder
关键源码：
```java

/*
 * Copyright 2013-2019 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.cloud.gateway.handler.predicate;

import java.util.List;
import java.util.Map;
import java.util.function.Predicate;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.reactivestreams.Publisher;
import reactor.core.publisher.Mono;

import org.springframework.cloud.gateway.handler.AsyncPredicate;
import org.springframework.cloud.gateway.support.ServerWebExchangeUtils;
import org.springframework.http.codec.HttpMessageReader;
import org.springframework.web.reactive.function.server.HandlerStrategies;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.server.ServerWebExchange;

/**
 * Predicate that reads the body and applies a user provided predicate to run on the body.
 * The body is cached in memory so that possible subsequent calls to the predicate do not
 * need to deserialize again.
 */
public class ReadBodyPredicateFactory
		extends AbstractRoutePredicateFactory<ReadBodyPredicateFactory.Config> {

	protected static final Log log = LogFactory.getLog(ReadBodyPredicateFactory.class);

	private static final String TEST_ATTRIBUTE = "read_body_predicate_test_attribute";

	private static final String CACHE_REQUEST_BODY_OBJECT_KEY = "cachedRequestBodyObject";

	private final List<HttpMessageReader<?>> messageReaders;

	public ReadBodyPredicateFactory() {
		super(Config.class);
		this.messageReaders = HandlerStrategies.withDefaults().messageReaders();
	}

	public ReadBodyPredicateFactory(List<HttpMessageReader<?>> messageReaders) {
		super(Config.class);
		this.messageReaders = messageReaders;
	}
	// ......
}
	
```

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.core.codec;

import java.util.Map;
import org.reactivestreams.Publisher;
import org.springframework.core.ResolvableType;
import org.springframework.core.io.buffer.DataBuffer;
import org.springframework.core.io.buffer.DataBufferUtils;
import org.springframework.lang.Nullable;
import org.springframework.util.MimeType;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

public abstract class AbstractDataBufferDecoder<T> extends AbstractDecoder<T> {
    private int maxInMemorySize = 262144;

    protected AbstractDataBufferDecoder(MimeType... supportedMimeTypes) {
        super(supportedMimeTypes);
    }

    public void setMaxInMemorySize(int byteCount) {
        this.maxInMemorySize = byteCount;
    }

    public int getMaxInMemorySize() {
        return this.maxInMemorySize;
    }

    public Flux<T> decode(Publisher<DataBuffer> input, ResolvableType elementType, @Nullable MimeType mimeType, @Nullable Map<String, Object> hints) {
        return Flux.from(input).map((buffer) -> {
            return this.decodeDataBuffer(buffer, elementType, mimeType, hints);
        });
    }

    public Mono<T> decodeToMono(Publisher<DataBuffer> input, ResolvableType elementType, @Nullable MimeType mimeType, @Nullable Map<String, Object> hints) {
        return DataBufferUtils.join(input, this.maxInMemorySize).map((buffer) -> {
            return this.decodeDataBuffer(buffer, elementType, mimeType, hints);
        });
    }

    /** @deprecated */
    @Deprecated
    @Nullable
    protected T decodeDataBuffer(DataBuffer buffer, ResolvableType elementType, @Nullable MimeType mimeType, @Nullable Map<String, Object> hints) {
        return this.decode(buffer, elementType, mimeType, hints);
    }
}
```
