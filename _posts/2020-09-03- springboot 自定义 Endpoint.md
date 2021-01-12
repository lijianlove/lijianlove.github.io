---
layout:     post
title:      springboot 自定义 Endpoint
subtitle:   springboot 自定义 Endpoint
date:       2020-09-03
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - springboot
---

#### 依赖
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```
#### Endpoint

```
import com.jovision.jcmp.ChannelPool;
import io.netty.channel.ChannelHandlerContext;
import org.springframework.boot.actuate.endpoint.annotation.Endpoint;
import org.springframework.boot.actuate.endpoint.annotation.ReadOperation;
import org.springframework.boot.actuate.endpoint.web.annotation.WebEndpoint;
import org.springframework.context.annotation.Configuration;

import java.util.Map;

/**
 * @author lijian@jovision.com
 * @date 2020/11/27
 **/
@Configuration
@WebEndpoint(id = "channels")
public class ChannelPoolEndpoint {

    @ReadOperation(produces = "application/json")
    public Map<String, ChannelHandlerContext> chennels() {
        return ChannelPool.getChannelPollV2();
    }
}

```

#### 启用

```
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

#### 访问
> http://127.0.0.1:8080/actuator/channels