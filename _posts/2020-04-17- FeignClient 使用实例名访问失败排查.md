---
layout:     post
title:      feign
subtitle:   FeignClient 使用实例名访问失败排查
date:       2020-04-17
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - feign
---

### 现状

```
@FeignClient(value = "jvcloud-saas-udms", fallbackFactory = UdmsClientHihystric.class, configuration = {FeignConfig.class})

```

使用上面的配置调用服务，总会提示未知主机


### 解决
* 首先怀疑 ribbon 依赖失败，因为根据服务名没有替换为对应的实例地址，排查各种开关配置，未发现问题
* 打开 feignclient 源码分析，边看源码分析边跟踪代码，最终成功定位到问题
![feignclient.png](https://i.loli.net/2020/10/30/AmqMfoFySYjVc86.png)
没有修改之前这个Client 对象总为 Client#Default , Client#Default 为正常的url 调用的client , 所以这里应该是一个负载均衡的client，那么为什么client 会是这个呢，首先想到是不是要手动注入一个负载均衡 client ,但是很快否决，因为不能所有的都为负载均衡client，继续跟代码，查看client new 出来的线程堆栈，最终在 FeignConfig 中发现一个配置


```
    @Bean
    @ConditionalOnMissingBean
    public Client feignClient() throws NoSuchAlgorithmException, KeyManagementException {
        SSLContext ctx = SSLContext.getInstance("SSL");
        X509TrustManager tm = new X509TrustManager() {
            @Override
            public void checkClientTrusted(X509Certificate[] chain,
                                           String authType) throws CertificateException {
            }

            @Override
            public void checkServerTrusted(X509Certificate[] chain,
                                           String authType) throws CertificateException {
            }

            @Override
            public X509Certificate[] getAcceptedIssuers() {
                return null;
            }
        };
        ctx.init(null, new TrustManager[]{tm}, null);
        return new Client.Default(ctx.getSocketFactory(),
                (hostname, session) -> {
                    return true;
                });
    }
```

当时这个配置主要是为了关闭 feign 的https 校验的，注意最后一行，写死了返回 Client.Default 

> https://blog.csdn.net/forezp/article/details/83896098