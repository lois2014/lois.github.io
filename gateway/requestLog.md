# 请求日志
1. 自定义全局过滤器
 ```java
    @Component
    public class CustomGlobalFilter implements GlobalFilter, Ordered {

        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            log.info("custom global filter");
            return chain.filter(exchange);
        }

        @Override
        public int getOrder() {
            return -1;
        }
    }
 ```
 2. 获取请求参数

 ```java
    ServerHttpRequest request = exchange.getRequest();
    Object requestBody = "";
    //post cachedRequestBodyObject
    if (request.getMethod() != null
            && HttpMethod.POST.toString().equals(request.getMethod().toString())) {
        requestBody = exchange.getAttribute("cachedRequestBodyObject");
    }
    String queryString = request.getQueryParams().toString();   
 ```
 3. 获取响应参数
     1. 重新构造response, ServerHttoResonseDecorator
     2. 写入outputStream后要释放原来dataBuffer，不然会造成直接内存溢出问题
     3. 参考自 [Spring Cloud Gateway 内存溢出解决过程](https://blog.csdn.net/live501837145/article/details/99446673)
     
 ```java
 private ServerHttpResponseDecorator handleResponse(ServerWebExchange exchange) {
        log.debug(" ----- response start  ------");
        ServerHttpResponse response = exchange.getResponse();
        DataBufferFactory bufferFactory = response.bufferFactory();
        ServerHttpResponseDecorator decoratedResponse = new ServerHttpResponseDecorator(response) {
            @Override
            public Mono<Void> writeWith(Publisher<? extends DataBuffer> body) {
                if (body instanceof Flux) {
                    Flux<? extends DataBuffer> fluxBody = (Flux<? extends DataBuffer>) body;
                    return super.writeWith(fluxBody.buffer().map(dataBuffers -> {
                        byte[] content = retrieveBytes(dataBuffers);
                        String responseResult = new String(content, Charset.forName("UTF-8"));
                        log.info("Response:[ Status: {} Header: {} ResponseResult: {}] ",
                                this.getStatusCode(), this.getHeaders().toString(), responseResult);
                        return bufferFactory.wrap(content);
                    }));
                }
                return super.writeWith(body);
            }
        };
        log.debug(" ----- response end  ------");
        return decoratedResponse;
    }

    //FIX OutOfDirectMemory
    public byte[] retrieveBytes(List<? extends DataBuffer> dataBuffers) {
        byte[] retrieveBytes = null;
        try (ByteArrayOutputStream outputStream = new ByteArrayOutputStream()) {
            dataBuffers.forEach(i -> processBuffer(i, outputStream));
            retrieveBytes = outputStream.toByteArray();
        } catch (IOException e) {
            log.error("IOException closing ByteArrayOutputStream", e);
        } catch (Exception e) {
            log.error("TaskException processing DataBuffer", e);
        }
        return retrieveBytes;
    }

    private void processBuffer(DataBuffer dataBuffer, ByteArrayOutputStream outputStream) throws RuntimeException {
        int count;
        while ((count = dataBuffer.readableByteCount()) > 0) {
            byte[] array = new byte[count];
            dataBuffer.read(array);
            try {
                outputStream.write(array);
            } catch (IOException e) {
                DataBufferUtils.release(dataBuffer);
                throw new RuntimeException(e);
            }
        }
        DataBufferUtils.release(dataBuffer);
    }    
 ```