---
layout:     post
title:      Jackson Boolean 类型传值数字转换问题
subtitle:   Jackson Boolean 类型传值数字转换问题
date:       2020-08-20
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - Jackson
---


### 现象
后端定义了一个 Boolean 类型的字段，前端给这个字段传值 123，后端竟然可以接受成功（接受到的是 true）

### 代码跟踪

![1.png](https://i.loli.net/2020/12/03/sipk4zvhUuPGfnr.png)

![2.png](https://i.loli.net/2020/12/03/ucqVoiWBfZlJenk.png)

![3.png](https://i.loli.net/2020/12/03/G8pT3qjQODUFrt4.png)

![4.png](https://i.loli.net/2020/12/03/rVywPGAUcD1EJlz.png)

### 解决

```

import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.core.JsonToken;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.JsonDeserializer;

import java.io.IOException;

/**
 * NumberDeserializers.BooleanDeserializer
 *
 * @author lijian@jovision.com
 * @date 2020/12/3
 **/
public class BooleanDeserializer extends JsonDeserializer<Boolean> {

    final protected Class<?> _valueClass = Boolean.class;

    @Override
    public Boolean deserialize(JsonParser jp, DeserializationContext ctxt) throws IOException,
            JsonProcessingException {
        return _parseBooleanPrimitive2(jp, ctxt);
    }

    protected final boolean _parseBooleanPrimitive2(JsonParser jp, DeserializationContext ctxt)
            throws IOException, JsonProcessingException {
        JsonToken t = jp.getCurrentToken();
        if (t == JsonToken.VALUE_TRUE) {
            return true;
        }
        if (t == JsonToken.VALUE_FALSE) {
            return false;
        }
        if (t == JsonToken.VALUE_STRING) {
            String text = jp.getText().trim();
            if ("true".equals(text)) {
                return true;
            }
            if ("false".equals(text) || text.length() == 0) {
                return Boolean.FALSE;
            }
        }
        throw ctxt.mappingException(_valueClass);
    }
}
```

```
    @NotNull(message = "alarmSoundEnable must not null")
    @JsonDeserialize(contentUsing = BooleanDeserializer.class)
    private Boolean alarmSoundEnable;
```