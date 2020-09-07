---
title: JSON格式介绍
urlname: orinxb
date: 2020-06-09 10:17:59 +0800
tags: []
categories: []
---

JSON 格式：
第一种，对象格式：
{"a":"1","b":"2"}
第二种，对象数组格式：
[{"a":"1","b":"2"},{"a":"1","b":"2"}]
作用：
使用 ajax 进行前后台数据交换
移动端与服务端的数据交换，统一数据规范

**下面是用与转换 json 格式的 java 类库：**

### com.alibaba.fastjson

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>x.x.x</version>
</dependency>
<!--其中x.x.x是版本号，根据需要使用特定版本，建议使用最新版本。-->
```

### org.codehaus.jackson

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>x.x.x</version>
</dependency>
<!--Spring boot默认使用Jackson core包主要用于解析json，还有annotations,databind-->
```

JSON 主流类库 Gson，FastJson，Jackson，Json-lib
性能分析：[https://blog.csdn.net/jiyueqianxue/article/details/83377181](https://blog.csdn.net/jiyueqianxue/article/details/83377181)

### _fastjson 简单介绍：_

\_
fastjson 是阿里巴巴的开源 JSON 解析库，它可以解析 JSON 格式的字符串，支持将 Java Bean 序列化为 JSON 字符串，也可以从 JSON 字符串反序列化到 JavaBean。
优点： **速度快** 使用广泛 测试完备 使用简单

_**常用方法：**_
\_
String(json 对象)-->JSON 对象
JSONObject jsonObj = JSON.parseObject(str); 取 jsonObj.get("xx") 存 jsonObj.put(key,value);

String(json 对象)-->Java 对象
Model model = JSON.parseObject(str, Model.class);

String(json 数组格式)-->JSONArray  
JSONArray jsonArr = JSON.parseArray(str);

Object、JSONObject、JSONArray、JavaBean、数组、List、Set、Map 都可以通过这种方式转 String
String str= JSON.toJSONString(object);

java 对象-->json 对象
JSONObject obj = (JSONObject) JSON.toJSON(javabean);

### jackson 简单介绍***：***

1. jackson-core，核心包，提供基于"流模式"解析的相关 API，它包括 JsonPaser 和 JsonGenerator。 Jackson 内部实现正是通过高性能的流模式 API 的 JsonGenerator 和 JsonParser 来生成和解析 json。
1. jackson-annotations，注解包，提供标准注解功能。
1. jackson-databind ，数据绑定包， 提供基于"对象绑定" 解析的相关 API （ ObjectMapper ） 和"树模型" 解析的相关 API （JsonNode）；基于"对象绑定" 解析的 API 和"树模型"解析的 API 依赖基于"流模式"解析的 API。

时间类型格式化： SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
json 转 String：
ObjectMapper mapper = new ObjectMapper();
String result = mapper.writeValueAsString(list);
writeValueAsString(list)中的 list 可换成其他对象，如 javabean，数组，集合等，result 即为 json 字符串

jackson 处理 json：[https://blog.csdn.net/psh18513234633/article/details/88599509](https://blog.csdn.net/psh18513234633/article/details/88599509)
