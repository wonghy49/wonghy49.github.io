---
layout: post
title: Spring Boot之基础准备
tags:  [Spring Boot]
categories: [Spring Boot]
author: wonghy
---

## SpringBoot是什么？
- 简化Spring应用开发的一个框架；
- 整个Spring技术栈的大集合；
- J2EEE开发的一站式解决方案
## SpringBoot优缺点？
- 快速创建独立运行的Spring项目和主流框架的继承
- 使用嵌入式的Servlet容器，应用无需打包成war包，直接打包成jar包，使用java -jar命令执行
- starters自动依赖与版本控制
- 大量自动配置，简化开发，也可修改默认值
- 无需配置xml
- 生产环境的运行时应用监控
- 与云计算的天然合成


## SpringApplication的执行流程

**流程图：**
![image](https://note.youdao.com/yws/api/personal/file/F8240B8EF72E4749A998B6A085C8490D?method=download&shareKey=7f2f78e525977804bf8fd2b690929c8e)

**具体步骤：**
1. SpringApplication初始化
2. 




SpringApplicationRunListener
