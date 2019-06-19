---
title: Spring Cloud（二）：服务治理Eureka
date: 2017-03-07 13:25:24
categories: Spring Cloud
tags: [Spring Cloud]
comments: true
toc: true
---

## 什么是Spring Cloud Eureka？

Spring Cloud Eureka 是 Spring Cloud Netflix微服务套件中的一部分，是在Eureka开源组件进一步封装。服务中心又称注册中心，管理各种服务功能包括服务的注册、发现、熔断、负载、降级等，比如dubbo admin后台的各种功能。 

Eureka 是一个基于 REST 的服务，主要在 AWS 云中使用, 定位服务来进行中间层服务器的负载均衡和故障转移。 

Eureka的基本架构有三种角色。

- **Register Service:服务注册中心**，它是一个Eureka Server，提供服务注册和发现的功能
  - 失效剔除

  <u>Eureka Server在启动的时候会创建一个定时任务，默认每隔30秒将当前清单中超时90秒没有续约的服务剔除出去。</u>

  - 自我保护

  <u>服务注册到Eureka Server后，会维护一个心跳连接。Eureka Server在运行期间，会统计心跳失败的比例在15分钟内是否低于85%，如果出现低于的情况，Eureka Server 会将当前的实例注册信息保护起来，让这些实例不会过期。</u>

  ```txt
  EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.
  ```

  

- **Provider Service:服务提供者**，它是一个Eureka Client，提供服务
  - 服务注册

  <u>“服务提供者”启动的时候发送REST请求将自己注册到Eureka Server，同时带上自身的元数据信息。Eureka Server接收到REST请求后，将元数据存储到一个双层结构Map中，第一层的key是服务名，第二层key是具体服务的实例名。</u>

  - 服务同步

  <u>由于服务注册中心之间因互相注册为服务，当服务提供者发送注册请求到一个服务注册中心，它会将该请求转发给集群中相连的其他注册中心，从而实现注册中心之间的服务同步。</u>

  - 服务续约

  <u>注册完服务之后，服务提供者会维护一个心跳用来持续告诉Eureka Server，我还活着，防止Eureka Server 剔除任务将服务实例从服务列表中排除出去</u>。

- **Comsumer Service:服务消费者**，它是一个Eureka Client，消费服务
  - 获取服务

  <u>启动服务消费者，它发送一个REST请求给服务注册中心，获取注册服务清单。Eureka Server会维护一份只读的服务清单返回给客户端，同时清单会每隔30秒更新一次。</u>

  - 服务调用

  <u>服务消费者在获取服务清单后，通过服务名可以获取具体服务的实例名和实例的元数据信息。（每个服务客户端需要注册到一个Zone中，每个客户端对应一个Region和一个Zone。在服务调用的时候，优先访问同处一个Zone的服务提供方，如果访问不到，就访问其他Zone）</u>

  - 服务下线

  <u>当服务实例进行正常的关闭操作时，它会触发一个服务下线的REST请求给Eureka Server，服务端接收到请求后，将该服务状态设置为下线（DOWN），并把下线的事件传播出去。</u>

服务流程是先启动注册中心，服务提供者生产服务并注册到服务中心中，消费者从服务中心中获取服务并执行 

## 什么是服务治理？

服务治理是微服务架构中最为核心和基础的模块，它主要用来实现各个微服务实例的自动化注册与发现。

![1534400474739](https://note.youdao.com/yws/api/personal/file/C8F0E49F8A384402B440C63AC6F5D81D?method=download&shareKey=59c1b82b4243a9f5bd812f8f8b6bca5e)

- 服务注册

在服务治理的框架中，通常会构建一个服务注册中心，每个服务单元向注册中心登记自己提供的服务，将主机与端口好、版本号、通信协议等一些附加信息告知注册中心，注册中心按服务名分类组织服务名单。服务注册中心还需要以==心跳的方式==去检测清单中的服务是否可用，若不可用需要从服务清单中剔除掉。

- 服务发现

服务间的调用是通过向服务名发起请求调用实现。调用方向服务注册中心咨询服务，并获得所有服务的实例清单，以实现对具体服务实例的访问。当服务发起调用的时候，就从这份清单用某种轮询策略去除一个位置来进行服务调用。

## 环境搭建

搭建服务注册中心

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>

 <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-dependencies</artifactId>
     <version>Finchley.SR1</version>   <!--这个springboot2.x以上，springcloud指定的版本-->
     <type>pom</type>
     <scope>import</scope>
</dependency>
```

```yaml
server:
  port: 8761
eureka:
  instance:
    hostname: localhost
  client:
  	#代表不向注册中心注册自己
    register-with-eureka: false   
    #注册中心是维护服务实例，不需要去检索服务
    fetch-registry: false   
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/


```

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {}
```

客户端

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```yaml
server:
  port: 8762
spring:
  application:
    name: eureka-client
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

```java
@SpringBootApplication
@EnableEurekaClient
public class EurekaClientApplication{}
```

## 高可用注册中心（集群）

#### 服务端

application.yml：

```yaml
---
spring:
  profiles: peer1
  application:
    name: eureka-ha
server:
  port: 8761
eureka:
  instance:
    hostname: peer1
  client:
    service-url:
      defaultZone: http://peer2:8762/eureka/

---
spring:
  profiles: peer2
  application:
      name: eureka-ha
server:
  port: 8762
eureka:
  instance:
    hostname: peer2
  client:
    service-url:
      defaultZone: http://peer1:8761/eureka/
```

电脑配置：

因为是在本地搭建Eureka Server 集群，所以需要修改本地的host o Windows 系统的电脑
在C: /windows/system32/drivers/etc/hosts 中修改， Mac 系统的电脑通过终端vim/etc/hosts 进行编
辑修改

```
127.0.0.1 peer1
127.0.0.1 peer2
```

启动peer1,peer2：

```shell
java -jar xx.jar --spring.profiles.active=peer1
java -jar xx.jar --spring.profiles.active=peer2
```



