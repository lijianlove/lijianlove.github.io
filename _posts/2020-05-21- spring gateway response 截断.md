---
layout:     post
title:      spring gateway response 截断
subtitle:   spring gateway response 截断
date:       2020-05-15
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - geteway
---
### 现状
* 根据接口获取到response ， 根据response 修改一下response 再返回
* 偶现 ExecuteResult<JSONObject> executeResult = JSONObject.parseObject(s, ExecuteResult.class); 这一行报错，显示不是json , 转换错误
```
@Component
@Slf4j
public class PostLoginGatewayFilterFactory extends AbstractGatewayFilterFactory<PostLoginGatewayFilterFactory.Config> {

    @Autowired
    private RedisOperatingService redisOperatingService;
    @Autowired
    private SmsClient smsClient;

    public PostLoginGatewayFilterFactory() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        ModifyResponseGatewayFilter gatewayFilter = new ModifyResponseGatewayFilter();
        gatewayFilter.setFactory(this);
        return gatewayFilter;
    }

    public static class Config {
    }


    public class ModifyResponseGatewayFilter implements GatewayFilter, Ordered {
        private GatewayFilterFactory<PostLoginGatewayFilterFactory.Config> gatewayFilterFactory;

        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            return chain.filter(exchange.mutate()
                    .response(new ModifiedServerHttpResponse(exchange)).build());
        }

        @Override
        public int getOrder() {
            return NettyWriteResponseFilter.WRITE_RESPONSE_FILTER_ORDER - 1;
        }

        public void setFactory(GatewayFilterFactory<PostLoginGatewayFilterFactory.Config> gatewayFilterFactory) {
            this.gatewayFilterFactory = gatewayFilterFactory;
        }
    }


    protected class ModifiedServerHttpResponse extends ServerHttpResponseDecorator {

        private final ServerWebExchange exchange;

        public ModifiedServerHttpResponse(ServerWebExchange exchange) {
            super(exchange.getResponse());
            this.exchange = exchange;
        }

        @SuppressWarnings("unchecked")
        @Override
        public Mono<Void> writeWith(Publisher<? extends DataBuffer> body) {
            if (body instanceof Flux) {
                Flux<? extends DataBuffer> fluxBody = (Flux<? extends DataBuffer>) body;
                return super.writeWith(fluxBody.map(dataBuffer -> {
                    // 注意这里！！！
                    byte[] content = new byte[dataBuffer.readableByteCount()];
                    dataBuffer.read(content);
                    DataBufferUtils.release(dataBuffer);
                    String s = new String(content, Charset.forName("UTF-8"));
                    String url = exchange.getRequest().getURI().getPath();

                    if (url.equals("/v1/auth/login")) {
                        ExecuteResult<JSONObject> executeResult = JSONObject.parseObject(s, ExecuteResult.class);
                        String account = String.valueOf(exchange.getAttributes().get("account"));
                        String tenantId = String.valueOf(exchange.getAttributes().get("tenantId"));

                        if (executeResult.getCode() != BusinessErrorEnum.BASE_SUCCESS.getCode()) {
                            int num = redisOperatingService.addLoginContinuousErrorNum(tenantId, account);

                            //判断账号是否被锁定状态
                            if (num == 5) {
                                // redis 记录用户锁定状态
                                redisOperatingService.setLoginLockState(tenantId, account);
                                redisOperatingService.selLoginUnlockCodeState(tenantId, account);
                                redisOperatingService.deleteLoginContinuousErrorNum(tenantId, account);

                                NotificationDto notificationDto = new NotificationDto();
                                notificationDto.setReceiver(account);
                                notificationDto.setLanguage("zh-CN");
                                notificationDto.setTenantId(tenantId);
                                log.info("send sms , account="+account+" , tenantId="+tenantId);

                                try{
                                    smsClient.notification(notificationDto);
                                }catch (Exception ex){
                                    log.error("账号登录被锁定,发送短信通知失败", ex);
                                }


                                throw new BusinessException(BusinessErrorEnum.AUTH_USER_IS_LOCKED);
                            }
                        }else{
                            redisOperatingService.delLoginUnlockCodeState(tenantId, account);
                        }
                    }

                    if (url.equals("/v1/ums/forget_pwd")) {
                        ExecuteResult<JSONObject> executeResult = JSONObject.parseObject(s, ExecuteResult.class);
                        if (executeResult.getCode() == BusinessErrorEnum.BASE_SUCCESS.getCode()) {

                            String account = String.valueOf(exchange.getAttributes().get("account"));
                            String tenantId = String.valueOf(exchange.getAttributes().get("tenantId"));

                            redisOperatingService.deleteLoginContinuousErrorNum(tenantId, account);
                            redisOperatingService.delLoginLockState(tenantId, account);
                        }
                    }

                    return exchange.getResponse().bufferFactory().wrap(new String(content, Charset.forName("UTF-8")).getBytes());
                }));
            }
            return super.writeWith(body);
        }
    }

}
```

### 原因 
dataBuffer.readableByteCount() 读取的数据不全，response 被截断

### 处理

```
@Component
@Slf4j
public class PostLoginGatewayFilterFactory extends AbstractGatewayFilterFactory<PostLoginGatewayFilterFactory.Config> {

    @Autowired
    private RedisOperatingService redisOperatingService;
    @Autowired
    private SmsClient smsClient;

    public PostLoginGatewayFilterFactory() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        ModifyResponseGatewayFilter gatewayFilter = new ModifyResponseGatewayFilter();
        gatewayFilter.setFactory(this);
        return gatewayFilter;
    }

    public static class Config {
    }


    public class ModifyResponseGatewayFilter implements GatewayFilter, Ordered {
        private GatewayFilterFactory<PostLoginGatewayFilterFactory.Config> gatewayFilterFactory;

        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            return chain.filter(exchange.mutate()
                    .response(new ModifiedServerHttpResponse(exchange)).build());
        }

        @Override
        public int getOrder() {
            return NettyWriteResponseFilter.WRITE_RESPONSE_FILTER_ORDER - 1;
        }

        public void setFactory(GatewayFilterFactory<PostLoginGatewayFilterFactory.Config> gatewayFilterFactory) {
            this.gatewayFilterFactory = gatewayFilterFactory;
        }
    }


    protected class ModifiedServerHttpResponse extends ServerHttpResponseDecorator {

        private final ServerWebExchange exchange;

        public ModifiedServerHttpResponse(ServerWebExchange exchange) {
            super(exchange.getResponse());
            this.exchange = exchange;
        }

        @SuppressWarnings("unchecked")
        @Override
        public Mono<Void> writeWith(Publisher<? extends DataBuffer> body) {
            if (body instanceof Flux) {
                Flux<? extends DataBuffer> fluxBody = (Flux<? extends DataBuffer>) body;
                // 注意这里，修改为了 fluxBody.buffer()
                return super.writeWith(fluxBody.buffer().map(dataBuffer -> {
                // 注意这里，使用 DataBufferFactory 将数据组装再读取
                    DataBufferFactory dataBufferFactory = new DefaultDataBufferFactory();
                    DataBuffer join = dataBufferFactory.join(dataBuffer);
                    byte[] content = new byte[join.readableByteCount()];
                    join.read(content);
                    DataBufferUtils.release(join);
                    String s = new String(content, Charset.forName("UTF-8"));
                    String url = exchange.getRequest().getURI().getPath();

                    if (url.equals("/v1/auth/login")) {
                        ExecuteResult<JSONObject> executeResult = JSONObject.parseObject(s, ExecuteResult.class);
                        String account = String.valueOf(exchange.getAttributes().get("account"));
                        String tenantId = String.valueOf(exchange.getAttributes().get("tenantId"));

                        if (executeResult.getCode() != BusinessErrorEnum.BASE_SUCCESS.getCode()) {
                            int num = redisOperatingService.addLoginContinuousErrorNum(tenantId, account);

                            //判断账号是否被锁定状态
                            if (num == 5) {
                                // redis 记录用户锁定状态
                                redisOperatingService.setLoginLockState(tenantId, account);
                                redisOperatingService.selLoginUnlockCodeState(tenantId, account);
                                redisOperatingService.deleteLoginContinuousErrorNum(tenantId, account);

                                NotificationDto notificationDto = new NotificationDto();
                                notificationDto.setReceiver(account);
                                notificationDto.setLanguage("zh-CN");
                                notificationDto.setTenantId(tenantId);
                                log.info("send sms , account="+account+" , tenantId="+tenantId);

                                try{
                                    smsClient.notification(notificationDto);
                                }catch (Exception ex){
                                    log.error("账号登录被锁定,发送短信通知失败", ex);
                                }


                                throw new BusinessException(BusinessErrorEnum.AUTH_USER_IS_LOCKED);
                            }
                        }else{
                            redisOperatingService.delLoginUnlockCodeState(tenantId, account);
                        }
                    }

                    if (url.equals("/v1/ums/forget_pwd")) {
                        ExecuteResult<JSONObject> executeResult = JSONObject.parseObject(s, ExecuteResult.class);
                        if (executeResult.getCode() == BusinessErrorEnum.BASE_SUCCESS.getCode()) {

                            String account = String.valueOf(exchange.getAttributes().get("account"));
                            String tenantId = String.valueOf(exchange.getAttributes().get("tenantId"));

                            redisOperatingService.deleteLoginContinuousErrorNum(tenantId, account);
                            redisOperatingService.delLoginLockState(tenantId, account);
                        }
                    }

                    return exchange.getResponse().bufferFactory().wrap(new String(content, Charset.forName("UTF-8")).getBytes());
                }));
            }
            return super.writeWith(body);
        }
    }

}

```

### 参考
> https://blog.csdn.net/qq_41463655/article/details/105459544
> https://www.cnblogs.com/garfieldcgf/p/10474898.html