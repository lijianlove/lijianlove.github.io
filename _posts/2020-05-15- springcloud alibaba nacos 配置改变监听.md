---
layout:     post
title:      springcloud alibaba nacos 配置改变监听
subtitle:   springcloud alibaba nacos 配置改变监听
date:       2020-05-15
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - nacos
---

```

import com.alibaba.cloud.nacos.NacosConfigManager;
import com.alibaba.cloud.nacos.NacosPropertySourceRepository;
import com.alibaba.cloud.nacos.client.NacosPropertySource;
import com.alibaba.nacos.api.config.ConfigChangeEvent;
import com.alibaba.nacos.api.config.ConfigService;
import com.alibaba.nacos.client.config.listener.impl.AbstractConfigChangeListener;
import com.google.common.collect.Lists;
import com.jvcloud.saas.udms.common.config.listener.ConfigChangedListener;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.BeansException;
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import java.util.List;

/**
 * @author lijian@jovision.com
 * @date 2020/11/17
 **/
@Slf4j
@Component
public class ConfigChangedCommandLine implements CommandLineRunner, ApplicationContextAware {
    @Resource
    private NacosConfigManager nacosConfigManager;

    private List<ConfigChangedListener> configChangedListenerList = Lists.newArrayList();

    @Override
    public void run(String... args) throws Exception {
        log.info("ConfigChangedCommandLine init configChangedListenerList {}", configChangedListenerList);
        ConfigService configService = nacosConfigManager.getConfigService();
        List<NacosPropertySource> propertySources = NacosPropertySourceRepository.getAll();
        for (NacosPropertySource propertySource : propertySources) {
            if (propertySource.getSource().size() > 0) {
                configService.addListener(propertySource.getDataId(), propertySource.getGroup(), new AbstractConfigChangeListener() {
                    @Override
                    public void receiveConfigChange(ConfigChangeEvent event) {
                        log.info("receiveConfigChange event {}", event);
                        for (ConfigChangedListener listener : configChangedListenerList) {
                            listener.changed(propertySource.getDataId(), propertySource.getGroup(), event);
                        }
                    }
                });
            }
        }

    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        applicationContext.getBeansOfType(ConfigChangedListener.class).forEach((name, bean) -> configChangedListenerList.add(bean));
    }
}


```