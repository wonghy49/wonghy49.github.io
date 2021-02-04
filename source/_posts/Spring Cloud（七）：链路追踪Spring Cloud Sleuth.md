---
title: Spring Cloud（七）：链路追踪Spring Cloud Sleuth
date: 2019-08-21 16:41:24
categories: Spring Cloud
tags: [Spring Cloud]
comments: true
toc: true
---

# 一、简介

Spring Cloud Sleuth 主要功能就是在分布式系统中提供追踪解决方案，并且兼容支持了 zipkin。Sleuth主要目的是去跟进一个请求到底有哪些服务参与，参与的顺序又是怎样的，从而达到每个请求的步骤清晰可见，出了问题，很快定位，从而让我们方便去理清各个微服务间的调用关系。

- 耗时分析: 通过Sleuth可以很方便的了解到每个采样请求的耗时，从而分析出哪些服务调用比较耗时;
- 可视化错误: 对于程序未捕捉的异常，可以通过集成Zipkin服务界面上看到;
- 链路优化: 对于调用比较频繁的服务，可以针对这些服务实施一些优化措施。

![](https://note.youdao.com/yws/api/personal/file/368CF8936D264BA08936AFC51119CA58?method=download&shareKey=a8290104e713f5310ddada5d5caaa648)

spring cloud sleuth可以结合zipkin，将信息发送到zipkin，利用zipkin的存储来存储信息，利用zipkin ui来展示数据。

![](https://note.youdao.com/yws/api/personal/file/198FE9B214F84F2D80D44116215A633D?method=download&shareKey=49391cd409ebba12b6f7cb66e51a14a3)

此图总计7个spans

# 二、基本术语

- Span：基本工作单元，例如，在一个新建的span中发送一个RPC等同于发送一个回应请求给RPC，span通过一个64位ID唯一标识，trace以另一个64位ID表示，span还有其他数据信息，比如摘要、时间戳事件、关键值注释(tags)、span的ID、以及进度ID(通常是IP地址) span在不断的启动和停止，同时记录了时间信息，当你创建了一个span，你必须在未来的某个时刻停止它。

- Trace：一系列spans组成的一个树状结构，例如，如果你正在跑一个分布式大数据工程，你可能需要创建一个trace。

- Annotation：用来及时记录一个事件的存在，一些核心annotations用来定义一个请求的开始和结束     

  - cs - Client Sent -客户端发起一个请求，这个annotion描述了这个span的开始
  - sr - Server Received -服务端获得请求并准备开始处理它，如果将其sr减去cs时间戳便可得到网络延迟
  - ss - Server Sent -注解表明请求处理的完成(当请求返回客户端)，如果ss减去sr时间戳便可得到服务端需要的处理请求时间
  - cr - Client Received -表明span的结束，客户端成功接收到服务端的回复，如果cr减去cs时间戳便可得到客户端从服务端获取回复的所有所需时间 将Span和Trace在一个系统中使用Zipkin注解的过程图形化：

  

# 三、项目构建

## 构建server-zipkin

在spring Cloud为F版本的时候，已经不需要自己构建Zipkin Server了，只需要下载jar即可，下载地址：

https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/

也可以在这里下载：

链接: https://pan.baidu.com/s/1w614Z8gJXHtqLUB6dKWOpQ 密码: 26pf

下载完成jar 包之后，需要运行jar，如下：

```shell
#下载
curl -sSL https://zipkin.io/quickstart.sh | bash -s
java -jar zipkin-server-2.10.1-exec.jar
```

如果用 Docker 的话，使用以下命令：

```text
docker run -d -p 9411:9411 openzipkin/zipkin
```

访问浏览器localhost:9411

## pom依赖

服务提供者、服务消费者端的pom依赖均要加入

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

## application.yml配置文件

```yml
spring:
  sleuth:
    web:
      client:
        enabled: true # web开启sleuth功能
    sampler:
      probability: 1.0 # 将采样比例设置为 1.0，也就是全部都需要。默认是 0.1。当设置为1.0时就是链路数据100%收集到zipkin-server，当设置为0.1时，即10%概率收集链路数据
  zipkin:
    base-url: http://localhost:9411/ # 指定了 Zipkin 服务器的地址
```

# 四、使用rabbitmq进行链路数据收集

> 1、微服务与Zipkin Server解耦，微服务无须知道Zipkin Server的网络地址。
>
> 2、一些场景下，Zipkin Server与微服务网络可能不同，使用HTTP直接收集的方式无法工作，此时可借助消息中间件实现数据收集。

## 4.1、各微服务端（服务提供者、消费者）加入rabbitmq的依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
```

**那怎么把数据传输到 Zipkin服务器呢？**

> 答：利用Zipkin的环境变量，通过环境变量让 Zipkin 从 RabbitMQ 中读取信息

```shell
 java -jar zipkin.jar --RABBIT_URI=amqp://admin:12345@localhost:5672/sleuth --STORAGE_TYPE=elasticsearch --ES_HOSTS=http//:localhost:9200 --ES_HTTP_LOGGING=BASIC
```

> `--RABBIT_URI=amqp://admin:12345@localhost:5672/sleuth` 指定用 RabbitMQ 做数据传输
>
> `--STORAGE_TYPE=elasticsearch --ES_HOSTS=http//:localhost:9200 --ES_HTTP_LOGGING=BASIC` 指定用 Eelasticsearch 做数据传输

```
java -jar .\zipkin-server-2.10.4-exec.jar --zipkin.collector.rabbitmq.addresses=47.110.240.133:5672 --zipkin.collector.rabbitmq.username=root --zipkin.collector.rabbitmq.password=123 --zipkin.collector.rabbitmq.virtual-host=/my_vhost
```

**通过环境变量让 Zipkin 从 RabbitMQ 中读取信息**

| 属性                                         | 环境变量                  | 描述                                                         |
| :------------------------------------------- | :------------------------ | :----------------------------------------------------------- |
| zipkin.collector.rabbitmq.concurrency        | RABBIT_CONCURRENCY        | 并发消费者数量，默认为1                                      |
| zipkin.collector.rabbitmq.connection-timeout | RABBIT_CONNECTION_TIMEOUT | 建立连接时的超时时间，默认为 60000毫秒，即 1 分钟            |
| zipkin.collector.rabbitmq.queue              | RABBIT_QUEUE              | 从中获取 span 信息的队列，默认为 zipkin                      |
| zipkin.collector.rabbitmq.uri                | RABBIT_URI                | 符合 RabbitMQ URI 规范 的 URI，例如amqp://user:pass@host:10000/vhost |

如果设置了 URI，则以下属性将被忽略。

| 属性                                   | 环境变量            | 描述                                                         |
| :------------------------------------- | :------------------ | :----------------------------------------------------------- |
| zipkin.collector.rabbitmq.addresses    | RABBIT_ADDRESSES    | 用逗号分隔的 RabbitMQ 地址列表，例如localhost:5672,localhost:5673 |
| zipkin.collector.rabbitmq.password     | RABBIT_PASSWORD     | 连接到 RabbitMQ 时使用的密码，默认为 guest                   |
| zipkin.collector.rabbitmq.username     | RABBIT_USER         | 连接到 RabbitMQ 时使用的用户名，默认为guest                  |
| zipkin.collector.rabbitmq.virtual-host | RABBIT_VIRTUAL_HOST | 使用的 RabbitMQ virtual host，默认为 /                       |
| zipkin.collector.rabbitmq.use-ssl      | RABBIT_USE_SSL      | 设置为true则用 SSL 的方式与 RabbitMQ 建立链接                |

## 4.2、安装RabbitMQ

- 安装Erlang及配置

首先，您需要安装支持的 Windows 版Erlang。下载并运行Erlang for Windows 安装程序。下载地址http://www.erlang.org/downloads，我是64位的所以下载的64位版本。

设置环境变量，新建ERLANG_HOME；增加Erlang变量到path，%ERLANG_HOME%\bin;

打开cmd命令框，输入erl

- 安装RabbitMQ及配置

设置环境变量，新建RABBITMQ_SERVER；增加RABBITMQ_SERVER变量到path，%RABBITMQ_SERVER%\sbin;

打开cmd命令框，输入rabbitmqctl status

- 激活rabbitmq_management

```powershell
rabbitmq-plugins.bat enable rabbitmq_management
```

> 如果报出"Plugin configuration unchanged" 问题。
>
> 解决方法： 
> 将 C:\Users\Administrator\.erlang.cookie 同步至C:\Windows\System32\config\systemprofile\.erlang.cookie 
>同时删除：C:\Users\Administrator\AppData\Roaming\RabbitMQ目录

- 启动RabbitMQ服务

```shell
#1、启动方式1
rabbitmq-server.bat
#2、启动方式2
net start RabbitMQ
```

- RabbitMQ 测试

  测试地址 <http://localhost:15672/>  
  默认的用户名：guest 
  默认的密码为：guest 


## 4.3、配置文件改造

application.yml 文件的 spring.zipkin.base-url 改为 http://localhost:9411/，即指向一个错误的地址