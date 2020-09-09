---
title: 关于@Autowired在Mapper里爆红
urlname: zo4vgq
date: 2020-09-07 17:27:52 +0800
tags:
  - 注解
categories: java
---

关于 Autowired 注解在引入 mapper 接口爆红的原因

<!--more-->

原因：
由于@Autowired 是 Spring 的注解，所以会提示找不到对他的 bean（爆红），原因是你没有显示的将 xxMapper 注入 Spring 容器中去管理。

解决方案：

1. 这种情况只需要在 xxMapper.java 接口上添加@Repository 注解即可，此注解是 Spring 的注解，将当前类注册到 Spring 容器中实例化为一个 bean，所以@Autowired 就能找到此 bean 了。
1. 替换为@Resource 注解，此注解是 JDK 中的注解，不会向@Autowired 那样去 Spring 容器中寻找 bean。
