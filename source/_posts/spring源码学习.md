---
title: SpringBoot 源码学习
date: 2020-10-16 18:02:12
tags:
---
### 前期准备

#### 环境准备

1. Jdk版本 >= 1.8
2. SpringBoot版本 >= 2.0.0

#### Debug

创建最基本的SpringBoot项目，项目中只包含最基本的web依赖即可
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
