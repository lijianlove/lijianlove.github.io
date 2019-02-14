---
layout:     post
title:      MybatisPlus多数据源问题
subtitle:   MybatisPlus多数据源管理，多数据源切换
date:       2019-01-28
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - mybatis
    - mybatis-plus
---

#### 切换 Connection 的数据源
* mybatis-plus 提供了 DynamicDataSourceContextHolder 基于 ThreadLocal 切换数据源
* DynamicDataSourceAnnotationInterceptor 拦截器默认设置数据源，代码

[![]({{site.url}}/img/20190128/mybatis_mult_datasource.png)]()

* DynamicRoutingDataSource 判断使用哪个数据源

[![]({{site.url}}/img/20190128/mybatis_chose_datasource.png)]()

* 手动切换数据源

[![]({{site.url}}/img/20190128/mybatis_datasource_custome.png)]()

* 注意：多数据源事务问题 Mybatis-Plus 并不会提供支持，也就是说如果方法使用事务，需要自己实现分布式事务

* 该方式是重写了datasource ， 也就是说返回的 connection 对象是指定数据源的，但是mybatis-plus 中ServiceImpl 的批量处理方法并不适合，因为批量处理是基于 SqlSession 的，执行一批任务后 session.flush();所以提供了下面的防范支持基于 SqlSession 的多数据源支持

* 根据 datasource 获取 connection ， 根据 connection.createStatement() 获取到 Statement ， 借助 statement.addBatch 以及 clearBatch ， executeBatch 实现批处理

#### 切换 SqlSession 的数据源

* 先来看 ServiceImpl 中关于批量的方法实现，获取一个 sqlsession , sqlsession.flush();

[![]({{site.url}}/img/20190128/mybatis_sqlsession.png)]()


* ServiceImpl 中关于获取批量 SqlSession 的方法实现

[![]({{site.url}}/img/20190128/mybatis_getsqlsession.png)]()

* 解决思路：重写或者拦截这个方法，通过 SqlSessionFactoryBean 获取到对应的 sqlsessionfactory 从而获取到想要的数据库的 sqlsession

* 核心实现：


```java


import com.baomidou.dynamic.datasource.provider.DynamicDataSourceProvider;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;

import javax.annotation.PostConstruct;
import javax.sql.DataSource;
import java.util.HashMap;
import java.util.Map;

/**
 * @author lijian
 * @date 2019/1/21 下午8:56
 */
@Configuration
public class SqlSessionFactoryBeanConfig {

    @Autowired
    private DynamicDataSourceProvider provider;

    private static volatile Map<String, SqlSessionFactoryBean> sqlSessionFactoryBeanMap = new HashMap<>();

    @PostConstruct
    public void post() {
        Map<String, DataSource> beansOfType = provider.loadDataSources();
        for (String ds : beansOfType.keySet()) {
            DataSource dataSource = beansOfType.get(ds);
            SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
            sqlSessionFactoryBean.setDataSource(dataSource);
            sqlSessionFactoryBeanMap.put(ds, sqlSessionFactoryBean);
        }
    }

}


```

```java

@Component
@Order(-99)
@Aspect
@Slf4j
public class BatchSqlSessionAspect {

    @Pointcut("execution(* com.baomidou.mybatisplus.extension.service..*.*(..))")
    public void addAdvice() {
    }

    @Around("addAdvice()")
    public Object Interceptor(ProceedingJoinPoint pjp) throws Throwable {

        log.info("232");
        try {
            return pjp.proceed();
        } finally {
            TX.clear();
        }
    }

}

```