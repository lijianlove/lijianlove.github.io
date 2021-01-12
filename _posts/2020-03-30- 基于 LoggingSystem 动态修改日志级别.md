---
layout:     post
title:      log
subtitle:   基于 LoggingSystem 动态修改日志级别
date:       2020-03-30
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - log
---
#### 前提
 LoggingApplicationListener 在系统启动过程中就注册了一个 bean “springBootLoggingSystem”

#### 代码

```

import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.logging.LogLevel;
import org.springframework.boot.logging.LoggerConfiguration;
import org.springframework.boot.logging.LoggingSystem;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
import java.util.List;

/**
 * @author lijian@jovision.com
 * @date 2020/9/27
 **/
@RestController
@Slf4j
@RequestMapping("/log")
public class LogDynamicController {

    @Resource
    private LoggingSystem springBootLoggingSystem;

    @PostMapping(value = "/list", name = "获取配置")
    public List<LoggerConfiguration> list() {
        return springBootLoggingSystem.getLoggerConfigurations();
    }

    @PostMapping(value = "/set", name = "设置配置")
    public void setLevel(@RequestParam("logName") String logName, @RequestParam("Level") String Level) {
        springBootLoggingSystem.setLogLevel(logName, LogLevel.valueOf(Level));
    }
}

```