---
layout: post
title: Spring Boot之配置文件
tags:  [Spring Boot]
categories: [Spring Boot]
author: wonghy
---

#### **1、YAML语法：**



#### **2、配置文件值注入**





#### **3、配置文件占位符**

##### 1、随机数

```xml
${random.value}、${random.int}、${random.long}
${random.int(10)}、$random.int[1024,65536]}
```

##### 2、占位符获取前面配置的值，如果没有可以用 :（冒号） 指定默认值

```properties
person.lastName=hhy
person.dog.name=${person.hello:hello}_dog
```



#### 4、Profile多环境配置

##### 1、多Profile文件

application-{profile}.properties/yml

默认使用application.properties文件

##### 2、yml支持多文档块方式

```yaml
server:
  port: 8081
spring:
  profiles:
    active: prod

---
spring:
  profiles: dev
server:
  port: 8083

---
spring:
  profiles: prod
server:
  port: 8084
```

##### 3、激活使用profile

​	1、在配置文件中指定 spring.profiles.active=dev

​	2、命令行：--spring.profiles.active=dev

​		在run/debug Confgurations中Program arguments中填写--spring.profiles.active=dev

​		在cmd中，java -jar xxxx.jar --spring.profiles.active=dev

​	3、虚拟机参数：

​		在run/debug Confgurations中VM options中填写:

​		 -Dspring.profiles.active=dev



#### 5、配置文件加载位置

Spring boot启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件

file: ./config/  根路径的文件夹

file: ./

classpath: ./config/   类路径的config文件夹

classpath: /

优先级由高到低，高优先级的配置会覆盖低优先级的配置

Spring boot 会从四个位置全部加载主配置文件：**互补配置**



在运维的时候比较方便：

可以通过**spring.config.location**来改变配置

项目已经打包好了，我可以使用**命令行参数**的形式（--spring.config.location=xxxx），启动项目的时候指定配置文件的新位置；指定配置文件和默认加载的配置文件共同起作用形成**互补配置**



#### 6、外部配置加载顺序

**优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置**

**1.命令行参数**
	所有的配置都可以在命令行上进行指定
	java -jar xxx.jar --server.port=8087 --server.context-path=/abc
	多个配置用空格分开  --配置项=值
2.来自java:comp/env的JNDI属性
3.Java系统属性（System.getProperties()）
4.操作系统环境变量
5.RandomValuePropertySource配置的random.*属性值



由**jar包外向jar包内**进行寻找；

==**优先加载带profile**==
**6.jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件**
**7.jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件**

==**再来加载不带profile**==
**8.jar包外部的application.properties或application.yml(不带spring.profile)配置文件**
**9.jar包内部的application.properties或application.yml(不带spring.profile)配置文件**

10.@Configuration注解类上的@PropertySource
11.通过SpringApplication.setDefaultProperties指定的默认属性

所有支持的配置加载来源；
[参考官方文档](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#boot-features-external-config)



#### 7、自动配置原理

[配置文件能配置的属性参照](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#common-application-properties)

1. Spring Boot 启动的时候加载主配置类，开启自动配置功能==@EnableAutoConfiguration==
2. ==@EnableAutoConfiguration==的作用：







精髓：



xxxAutoConfiguration：自动配置类

给容器添加组件

xxxProperties：**封装**配置文件中相关属性（如果我们对自动配置类中哪些属性 不满意，可以通过在配置文件中配置）