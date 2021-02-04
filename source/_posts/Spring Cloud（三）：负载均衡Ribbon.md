---
title: Spring Cloud（三）：负载均衡Ribbon
date: 2017-03-07 13:25:24
categories: Spring Cloud
tags: [Spring Cloud]
comments: true
toc: true
---

## 什么是Spring Cloud Ribbon?

Spring Cloud Ribbon 是一个基于HTTP 和 TCP 的**客户端负载均衡**工具，它基于Netflix Ribbon实现的，使用了最经典的**Round Robin轮询算法**。通过Spring Cloud的封装，让我们轻松地将面向服务的REST模板请求自动转换成客户端负载均衡的服务调用。
Spring cloud有两种服务调用方式，**一种是Ribbon+restTemplate，另一种是feign**。 
Feign，也是基于Ribbon实现的工具。

## 客户端负载均衡和服务器负载均衡的区别？

**服务端负载均衡**：分为硬件负载均衡和软件负载均衡。硬件负载均衡主要是通过在服务器节点之间安装专门用于负载均衡的设备，如F5等；软件负载均衡是通过在服务器上安装一些具有负载均衡或者模块的软件来完成请求分发工作，如nginx。

它们都会维护一个下挂可用的服务端清单，通过心跳检测来剔除故障的服务端节点以保证清单中都是可以正常访问的服务端节点。当客户端发送请求给负载均衡的设备时，该设备按照某种算法从维护的可用服务端清单中取一台服务端地址，然后进行转发。



**客户端负载均衡**：基于客户端的负载均衡，客户端会有一个服务器地址列表。简单的说就是在客户端程序里面，自己设定一个调度算法，在向服务器发起请求的时候，先执行调度算法计算出向哪台服务器发起请求，然后再发起请求给服务器。 

## Ribbon默认实现的配置bean

![](https://note.youdao.com/yws/api/personal/file/E12F7E13A8664E609D44183A17F7C73B?method=download&shareKey=dbf4fe5746172dc30ffa1b64b6fefe77)

LoadBalancerAutoConfiguration为实现客户端负载均衡器的自动化配置类。

![](https://note.youdao.com/yws/api/personal/file/F9100FADF1294C4BAF8644F45BD65F3A?method=download&shareKey=7d091c2d30ee9e1b4c8bca5307926b2d)

## Ribbon+RestTemplate服务调用

- 首先Ribbon会从 Eureka Client里获取到对应的**服务注册表**，也就知道了所有的服务都部署在了哪些机器上，在监听哪些端口号。

- 然后Ribbon就可以使用默认的**Round Robin算法**，从中选择一台机器

- Feign就会针对这台机器，**构造并发起请求**。

  ![img](https://note.youdao.com/yws/api/personal/file/BB147FB0F608473581B29AFF9276DE5B?method=download&shareKey=95d73d754c350280190bdaecd51257d7)

![](https://note.youdao.com/yws/api/personal/file/733FB485E2814D45BEAF55FF198B77DC?method=download&shareKey=808e601ad4c6bb373493451440ba4a96)

### pom.xml配置

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId >
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId >
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>

```
### 项目准备

创建4个项目/模块

eureka-server 8761

service-hi 8762

service-hi 8763

service-ribbon 8764


```java
//该对象会使用Ribbon的自动化配置
@Bean
@LoadBalanced  //开启客户端负载均衡
RestTemplate restTemplate(){
	return new RestTemplate();
}
```

```xml

```

请求类型：

GET、POST、PUT、DELETE、HEAD请求

## 什么是Feign?

Feign是一个声明式的伪Http客户端，它使得写Http客户端变得更简单。使用Feign，只需要创建一个接口并注解。它具有可插拔的注解特性，可使用Feign 注解和JAX-RS注解。Feign支持可插拔的编码器和解码器。**Feign默认集成了Ribbon**，并和Eureka结合，默认实现了负载均衡的效果。 **动态代理**

只需要创建一个接口并用注解的方式来配置它，即可对服务提供方的接口的绑定

```xml
<!-- Finchley.SR1版本 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableFeignClients
public class EurekaFeignClientApplication {
```

```java
//@ FeignClient（“服务名”），来指定调用哪个服务
//如：调用了service-hi服务的“/hi”接口
@FeignClient(value = "service-hi")
public interface SchedualServiceHi {
    //使用了spring mvc的注解
    @RequestMapping(value = "/hi",method = RequestMethod.GET)
    String sayHiFromClientOne(@RequestParam(value = "name") String name);
}
```

#### 源码分析

@FeignClient注解用于创建声明式API接口，该接口是RESTful风格的。value()和name()是一样的，是被调用的服务的ServiceId，url()直接填写硬编码的Url地址。configuration()指明FeignClient的配置类，默认的配置类为FeignClientsConfiguration类。

#### 工作原理

1. 首先通过@EnableFeignClients注解开启FeignClient的功能。只有这个注解的存在，才能在程序启动时开启对@FeignClient注解的包扫描
2. 根据Feign的规则实现接口，并在接口上加上@FeignClient注解
3. 程序启动后，会进行包扫描，扫描所有的@FeignClient注解的类，并将这些信息注入Ioc容器
4. 当接口的方法被调用时， 通过JDK 的代理来生成具体的RequestTemplate 的对象。
5. 根据RequestTemplate 再生成Http 请求的Request 对象。
6. Request 对象交给Client 去处理， 其中Client 的网络请求框架可以是HttpURLConnection、HttpClient 和OkHttp 。
7. 最后Client 被封装到LoadBalanceClient 类，这个类结合类Ribbon 做到了负载均衡。

![](https://note.youdao.com/yws/api/personal/file/01075C1AB1F545D1A529CD041FA05375?method=download&shareKey=017285d16266320b9dfd3d2f26eb6283)

ps:上图来源于[石杉的架构笔记](https://juejin.im/post/5be13b83f265da6116393fc7#heading-0)