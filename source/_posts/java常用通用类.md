---
title: java常用通用类
urlname: vsrpqa
date: 2020-06-18 10:45:19 +0800
tags:
  - java工具类
categories: java
---

java工具类：创建文件夹 生成随机数 给对象中的值加双引号
            正则表达式-手机号验证 获取日期

<!--more-->

_**找方法最烦的事情，没有引入包！**_

#### 创建文件夹

```java
import java.io.File;

// 文件夹路径是否存在，不存在创建
public void fileCj(String url) {
    File file = new File(url);
    if (!file.exists() && !file.isDirectory()) {
        file.mkdir();
    }
}
```

#### 生成随机数

```java
import java.util.Random;

/**
* 生成随机数
* @param key 位数
* str_ 中可以自己写要的符号，数字
* @return
*/
public String genRandomNum(int key) {
    int maxNum = 36;
    int i;
    int count = 0;
    char[] str = { 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S',
                  'T', 'U', 'V', 'W', 'X', 'Y', 'Z', '0', '1', '2', '3', '4', '5', '6', '7', '8', '9' };
    StringBuffer pwd = new StringBuffer("");
    Random r = new Random();
    while (count < key) {
        i = Math.abs(r.nextInt(maxNum));
        if (i >= 0 && i < str.length) {
            pwd.append(str[i]);
            count++;
        }
    }
    return pwd.toString();
}
```

#### 给对象中的值加双引号

```java
import java.util.HashMap;
import java.util.Iterator;
import org.json.JSONObject;

/**
* 给对象值加上双引号
* @param javabean
* @return map
*/
	public static HashMap<Object, Object> javaBean(Object javabean) {

		HashMap<Object, Object> java = new HashMap<Object, Object>();
		JSONObject jsonObject = new JSONObject(javabean);
		Iterator<String> it = jsonObject.keys();
		while(it.hasNext()){
			String key = it.next();
			Object value = jsonObject.get(key);
			if (value instanceof String) {
				 java.put(key, value);
			}else {
				 java.put(key, ""+value+"");
			}
		}
		return java;
	}
```

#### 正则表达式-手机号验证

```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;
/**
* 手机号验证
* @param mobile
* @return boolean
*/
public static boolean ValidateMobile(String mobile) {
    final String regex = "^((13[0-9])|(14[5,7,9])|(15([0-3]|[5-9]))|(166)|(17[0,1,3,5,6,7,8])|(18[0-9])|(19[8|9]))\\d{8}$";
    Pattern p = Pattern.compile(regex);
    Matcher m = p.matcher(mobile);
    return m.matches();
}
```

#### 获取日期

```java
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;
import java.util.GregorianCalendar;
/**
* 获取日期
* @param day -1为昨天 0为今天 1为明天 自行按需要输入
* @return
*/
public static String fingTime(int day) {
    Date date = new Date();// 取时间
    Calendar calendar = new GregorianCalendar();
    calendar.setTime(date);
    calendar.add(calendar.DATE, day);// 把日期往后增加一天.整数往后推,负数往前移动
    date = calendar.getTime(); // 这个时间就是日期往后推一天的结果
    SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd 00:00:00");//"yyyy-MM-dd" 设置格式
    String dateString = formatter.format(date);
    return dateString;
}
```
