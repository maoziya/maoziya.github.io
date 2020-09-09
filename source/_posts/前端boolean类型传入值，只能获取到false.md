---
title: 前端boolean类型传入值，只能获取到false
urlname: gnsyem
date: 2020-07-01 15:26:38 +0800
tags:
  - java基础类型
categories: java
---

关于boolean类型的get,set

<!--more-->

**问题：postman 中测试，传入 true/false 时，后端永远只能获取到 false，无法获取 true.**
\*\*
**当你生成 set,get 的时候会出现三种情况: **
第 1 种，属性有 is：一般自动生成都不会出现，除非你设置了

```java
    private boolean isSign;

	public boolean getIsSign() {
		return isSign;
	}
	public void setIsSign(boolean isSign) {
		this.isSign = isSign;
	}
```

第 2 种，属性有 is: 出现了同名的情况 isSign 没有了 get

```java
	private boolean isSign;

	public boolean isSign() {
		return isSign;
	}
	public void setSign(boolean isSign) {
		this.isSign = isSign;
	}
```

第 3 种, 属性没有 is 的---发现自动将 get 变成了 is

```java
	private boolean Sign;

	public boolean isSign() {
		return Sign;
	}
	public void setSign(boolean sign) {
		Sign = sign;
	}
```

原因：

1.如果使用了 boolean，没有带 is 开头，在反射的时候，反射往往找的是 get 方法，所以会出错。 2.如果使用了 boolean，还带了 is 开头，那就更不用说了，一样出错,并且有些框架知道 boolean 会生成 is 或许做了处理，那样上面的（1）或许可以正常使用，但是你如果加了 is 开头，那就 gg 了。

**结论：**

**1.Boolean 类型,生成的 get 方法是 get 开头的(建议使用这个).**
**2.boolean 类型,生成的 get 方法是 is 开头的(用这个最好重写 getXxx()格式的方法).**
**3.boolean 类型的属性值不建议设置为 is 开头，写也行，在生成的 is 方法上加上 get 避免出错**
**
\_**总而言之一句话：最好为正常的 get,set 结构就完事了\*\*\_

参考的大佬文章链接：
[https://blog.csdn.net/java_zhangshuai/article/details/80599377?utm_source=blogxgwz8](https://blog.csdn.net/java_zhangshuai/article/details/80599377?utm_source=blogxgwz8)
[https://blog.csdn.net/justDoItfl/article/details/103119988](https://blog.csdn.net/justDoItfl/article/details/103119988)
