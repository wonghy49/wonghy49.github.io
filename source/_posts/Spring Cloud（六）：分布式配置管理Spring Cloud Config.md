---
title: Spring Cloud（六）：Spring Cloud Config
date: 2017-06-18 13:25:24
categories: Spring Cloud
tags: [Spring Cloud]
comments: true
toc: true
---




# 一、简介

- 提供服务端和客户端支持
- 集中管理个环境的配置文件
- 配置文件修改之后，可以快速生效
- 可以进行版本控制（使用git）
- 支持大的并发查询
- 支持各种语言

Spring Cloud Config项目是一个解决分布式系统的配置管理方案。它包含了Client和Server两个部分，server提供配置文件的存储、以接口的形式将配置文件的内容提供出去，client通过接口获取数据、并依据此数据初始化自己的应用。

# 二、准备工作

在github上面创建文件夹config-repo，然后往里面添加dev/sit/prod三个配置文件

```
config-dev.properties
config-sit.properties
config-prod.properties
```

## **server端**

### 1、添加依赖

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-config-server</artifactId>
	</dependency>
</dependencies>

```

### 2、配置文件

为了支持版本控制功能，推荐使用git

```yml
server:
  port: 8001
spring:
  application:
    name: spring-cloud-config-server
  cloud:
    config:
      server:
        git:
          # 配置git仓库的地址
          uri: https://github.com/ityouknow/spring-cloud-starter/     
          # git仓库地址下的相对地址，可以配置多个，用,分割。
          search-paths: config-repo      
          # git仓库的账号
          username: 
          # git仓库的密码
          password:                                             

```

- 本地存储配置的方式：只需要设置为`spring.profiles.active=native`，Config Server 默认从应用的src/main/resource目录中去检索配置文件
- 外部目录存储：`spring.cloud.config.server.native.searchLocations=file:E:/properties/`属性来指定配置文件的位置

### 3、启动类

```java
//加入注解@EnableConfigServer
@EnableConfigServer
@SpringBootApplication
public class ConfigServerApplication {
	public static void main(String[] args) {
		SpringApplication.run(ConfigServerApplication.class, args);
	}
}
```

### 4、测试server端是否可以访问配置信息

直接访问`http://localhost:8001/config/dev`，访问config-dev.properties配置文件

返回的信息包含了配置文件的位置、版本、配置文件的名称以及配置文件中的具体内容

```
{
    "name": "config", 
    "profiles": [
        "dev"
    ], 
    "label": null, 
    "version": null, 
    "state": null, 
    "propertySources": [
        {
            "name": "git地址/config-repo/config-dev.properties", 
            "source": {
                "hello": "hello im dev"
            }
        }
    ]
}
```

如果访问的URL为`http://localhost:8001/config-dev.properties`，则直接放回配置文件的内容

> 仓库中的配置文件会被转换成web接口，访问可以参照以下的规则：
>
> - /{application}/{profile}[/{label}]
> - /{application}-{profile}.yml
> - /{label}/{application}-{profile}.yml
> - /{application}-{profile}.properties
> - /{label}/{application}-{profile}.properties
>
> 以config-dev.properties为例子，它的application是config，profile是dev。client会根据填写的参数来选择读取对应的配置。

## **client端**

### 1、添加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```



### 2、配置文件

配置两个配置文件，application.properties和bootstrap.properties

application.properties如下：

```properties
spring.application.name=spring-cloud-config-client
server.port=8002
```

bootstrap.properties如下：

```properties
#对应{application}部分
spring.cloud.config.name=config
#对应{profile}部分
spring.cloud.config.profile=dev
#配置中心的具体地址
spring.cloud.config.uri=http://localhost:8001/
#对应git的分支。如果配置中心使用的是本地存储，则该参数无用
spring.cloud.config.label=master
```

> 特别注意：上面这些与spring-cloud相关的属性必须配置在bootstrap.properties中，config部分内容才能被正确加载。因为config的相关配置会先于application.properties，而bootstrap.properties的加载也是先于application.properties。

### 3、启动类

只需要用`@SpringBootApplication`就可以了

### 4、client端测试

```java
@RestController
class HelloController {
    //使用@Value注解来获取server端参数的值
    @Value("${hello}")
    private String hello;

    @RequestMapping("/hello")
    public String from() {
        return this.hello;
    }
}
```

### 5、更新配置文件——自动/手动通知

**如果修改git上面的配置文件后，再次访问浏览器URL后，获取的信息还是旧的参数？**

**答**：因为spring boot 项目只有启动的时候才会获取配置文件的值，修改github信息后，client端并没有再次获取

**解决办法**：每个客户端通过POST方法触发各自的`/refresh`。

- 添加依赖

增加了`spring-boot-starter-actuator`包，`spring-boot-starter-actuator`是一套监控的功能，可以监控程序在运行时状态，其中就包括`/refresh`的功能。

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

- 开启更新机制

  需要给加载变量的类上面加载`@RefreshScope`，在客户端执行`/refresh`的时候就会更新此类下面的变量值。

  ```java
  @RestController
  // 使用该注解的类，会在接到SpringCloud配置中心配置刷新的时候，自动将新的配置更新到该类对应的字段中。
  @RefreshScope 
  class HelloController {
  
      @Value("${hello}")
      private String hello;
  
      @RequestMapping("/hello")
      public String from() {
          return this.hello;
      }
  }
  ```

- 测试（手动更新）

  springboot 1.5.X 以上默认开通了安全认证，所以需要在配置文件application.properties添加以下配置*

  ```
  management.security.enabled=false
  ```

  再以post请求的方式来访问`http://localhost:8002/refresh` 就会更新修改后的配置文件。

  在win上面打开cmd执行`curl -X POST http://localhost:8002/refresh`，返回`["hello"]`表示已经更新

- webhook（自动更新）在github上操作

  WebHook是当某个事件发生时，通过发送http post请求的方式来通知信息接收方。Webhook来监测你在Github.com上的各种事件，最常见的莫过于push事件。如果你设置了一个监测push事件的Webhook，那么每当你的这个项目有了任何提交，这个Webhook都会被触发，这时Github就会发送一个HTTP
  POST请求到你配置好的地址。

-  Spring Cloud Bus 消息总线


# 三、配置中心服务化和高可用

## server端改造

1、添加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```

2、配置文件

将server端加到注册中心里

```yml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8000/eureka/   ## 注册中心eurka地址
```

## client端改造

1、添加依赖：加入eureka的依赖

2、配置文件

bootstrap.properties文件

去掉`spring.cloud.config.uri`直接指向server端地址的配置

```properties
#开启Config服务发现支持
spring.cloud.config.discovery.enabled=true
#指定server端的name,也就是server端spring.application.name的值
spring.cloud.config.discovery.serviceId=spring-cloud-config-server
#指向注册中心的地址
eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/

```

# 四、消息总线Spring Cloud Bus

**方式一**：某个微服务承担配置刷新的职责

![](https://note.youdao.com/yws/api/personal/file/4B0BADFF99E648469498FAF62BEAB72E?method=download&shareKey=df3a7ed59b3088ba7bc8b856d8840f64)

- 1、提交代码触发post给客户端A发送bus/refresh
- 2、客户端A接收到请求从Server端更新配置并且发送给Spring Cloud Bus
- 3、Spring Cloud bus接到消息并通知给其它客户端
- 4、其它客户端接收到通知，请求Server端获取最新配置
- 5、全部客户端均获取到最新的配置

存在问题：

1、打破了微服务的职责单一性。微服务本身是业务模块，它本不应该承担配置刷新的职责。

2、破坏了微服务各节点的对等性。

3、有一定的局限性。WebHook的配置随着承担刷新配置的微服务节点发生改变。

**方式二**：配置中心Server端承担起配置刷新的职责

![](https://note.youdao.com/yws/api/personal/file/62E22B57EC1B4B84AE79D7D2B359F8FA?method=download&shareKey=dad74118dd83ef1095bd4d7ea0ea09d7)

- 1、提交代码触发post请求给bus/refresh
- 2、server端接收到请求并发送给Spring Cloud Bus
- 3、Spring Cloud bus接到消息并通知给其它客户端
- 4、其它客户端接收到通知，请求Server端获取最新配置
- 5、全部客户端均获取到最新的配置

## client端改造（方式一）

### 1、添加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

### 2、配置文件

```properties
## 刷新时，关闭安全验证
management.security.enabled=false
## 开启消息跟踪
spring.cloud.bus.trace.enabled=true

spring.rabbitmq.host=192.168.9.89
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin
spring.rabbitmq.password=123456

```

### 3、测试

webhook向客户端发送一个/bus/refresh的POST请求，然后再依次访问其他的客户端，均能拿到最新的配置文件的信息

## server端改进版本（方式二）

### 1、添加依赖

```xml
<!--增加对消息总线的支持-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>

```

### 2、配置文件

```properties
server:
  port: 8001
spring:
  application:
    name: spring-cloud-config-server
  cloud:
    config:
      server:
        git:
          # 配置git仓库的地址
          uri: https://github.com/ityouknow/spring-cloud-starter/     
          # git仓库地址下的相对地址，可以配置多个，用,分割。
          search-paths: config-repo     
          # git仓库的账号
          username: username
          # git仓库的密码
          password: password                                    
  rabbitmq:
    host: 192.168.0.6
    port: 5672
    username: admin
    password: 123456

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8000/eureka/   ## 注册中心eurka地址


management:
  security:
     enabled: false
```

### 3、测试

webhook向服务端发送一个/bus/refresh的POST请求，然后再依次访问其他的客户端，均能拿到最新的配置文件的信息

```xml
<!-- 模拟webhook触发server端的bus/refresh -->
curl -X POST http://localhost:8001/bus/refresh
```

