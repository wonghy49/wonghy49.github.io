---
title: Spring Cloud（四）：熔断器Hystrix
date: 2017-03-07 13:25:24
categories: Spring Cloud
tags: [Spring Cloud]
comments: true
toc: true
---

## 什么是熔断器Hystrix？

在分布式系统中，服务与服务之间的依赖错综复杂， 一种不可避免的情况就是某些服务会
出现故障，导致依赖于它们的其他服务出现远程调度的线程阻塞。Hystrix 是Netflix 公司开
源的一个项目，它提供了熔断器功能，能够阻止分布式系统中出现联动故障。Hystrix 是通过
隔离服务的访问点阻止联动故障的，并提供了故障的解决方案，从而提高了整个分布式系统
的弹性。

出现问题的场景：由于服务的依赖性，会导致依赖于该故障服务的其他服务也处于线程阻
塞状态，最终导致这些服务的线程资源消耗殆尽， 直到不可用，从而导致整个问服务系统都不
可用，即雪崩效应。

#### 设计原则

- 防止单个服务的故障耗尽整个服务的Servlet 容器（例如Tomcat ）的线程资源。
- 快速失败机制，如果某个服务出现了故障，则调用该服务的请求快速失败，而不是线
  程等待。
- 提供**回退（ fallback ）**方案，在请求发生故障时，提供设定好的回退方案。
- 使用**熔断机制**，防止故障扩散到其他服务。
- 提供熔断器的监控组件**Hystrix Dashboard**，可以实时监控熔断器的状态。

#### 工作机制

![](https://note.youdao.com/yws/api/personal/file/BE8B3CE6A57649218A457044B47C1460?method=download&shareKey=c3d4b8a099ed8899cd3cd72b0299c54c)

1. 创建HystrixCommand或HystrixObservableCommand对象

2. 命令行执行（命令行模式）

   > - HystrixCommand实现了execute（）同步执行、queue（）异步执行
   >
   > `R value = command.execute();`
   >
   > `Future<R> fValue = command.queue();`
   >
   > - HystrixObservableCommand()实现了observe（）返回Hot Observable、toObservable（）返回Cold Observable
   >
   > `Observable<R>  ohValue = command.observe();`
   >
   > `Observable<R>  ohValue = command.toObservable();`

3. 结果是否被缓存

   若当前命令的请求缓存功能是被启用，并且该命令缓存命中，那么缓存的结果会立即以Observable对象的形式返回。

4. 断路器是否打开

   在命令结果没有缓存命中时，Hystrix在执行命令前需要检查短路器是否为打开状态：

   > - 断路器打开时，Hystrix不执行命令，转接到fallback处理逻辑，跳到第8步。
   > - 断路器关闭时，Hustrix跳到第5步，检查是否有可用资源来执行命令

5. 线程池/请求队列/信号量是否占满

   如果线程池/请求队列/信号量都占满了，Hystrix不会执行命令，转接到fallback处理逻辑，跳到第8步。

   ==注意：==这里的线程池是指每个依赖服务的专有线程池。Hystrix采用了“舱壁模式”来隔离每个依赖的服务

6. HystrixObservableCommand.construct()或HystrixCommand.run()

   Hystrix根据我们编写的方法来决定采取什么的方式去请求依赖服务

   > HystrixCommand.run()返回一个单一的结果，或者抛出异常
   >
   > HystrixObservableCommand.construct()返回一个Observable对象来发射多个结果或者通过onError发送错误通知

7. 计算断路器的健康度

   Hystrix会将“成功”、“失败”、“拒绝”、“超时”等信息报告给断路器，断路器维护一组计数器来统计这些数据。

8. fallback处理（==服务降级==）

   需要实现一个通用的响应结果，并且该结果的处理逻辑是从缓存或者根据一些静态逻辑来获取，而不是依赖网络请求获取。（也可以包含网络请求，该请求要包装在HystrixCommand或HystrixObservableCommand中，从而形成了级联的降级策略，最终结果还是一个稳定的返回结果的处理逻辑）

9. 返回成功的响应

#### 依赖隔离

优势点：

- 应用自身得到完全保护，不会受不可控的依赖服务影响。即便给依赖服务分配的线程池被填满，也不会影响应用自身的其余部分
- 可以有效降低接入新服务的风险。如果新服务接入后运行不稳定或存在问题，完全不会影响应用其他的请求。
- 依赖的服务从失效恢复正常后，它的线程池会被清理并能够马上恢复健康的服务。
- 依赖的服务出现配置错误的时候，线程池会快速反映出问题
- 每个专有线程池提供内置的并发实现

#### 在ribbon使用断路器

```xml
    <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
```

```java
@EnableEurekaClient
@EnableHystrix
@SpringBootApplication
public class EurekaClientRibbonApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaClientRibbonApplication.class, args);
    }

}
```

```java
	@RequestMapping("getBook")
    @HystrixCommand(fallbackMethod = "getBookError")
    public String getBook(@RequestParam String name) {
        logger.info("name:", name);
        return restTemplate.getForObject("http://eureka-client/getBook?name="+name,String.class);
    }

    public String getBookError(String name) {
        return "The name of the book is " + name + ", get Book Error!!";
    }
```

#### 在Feign使用断路器

Feign是自带断路器的，在D版本的Spring Cloud中，它没有默认打开。需要在配置文件中配置打开它，在配置文件加以下代码：

> feign.hystrix.enabled=true

@FeignClient中加入fallback，fallback引用的方法是继承了该接口的实现类

```java
@FeignClient(value = "eureka-client", fallback = BookFeignHystrixServiceImpl.class)
public interface BookFeignService {

    //此类中的方法和远程服务中contoller(即跟服务eureka-client)中的方法名和参数需保持一致
    @RequestMapping("getBook")
    public String getBook1(@RequestParam String name);
}

@Component
public class BookFeignHystrixServiceImpl implements BookFeignService{

    @Override
    public String getBook1(String name) {
        return "sorry, get book name:" + name;
    }
}
```



#### Hystrix DashBoard

提供了数据监控和友好的图形化展示界面

![](https://note.youdao.com/yws/api/personal/file/5325550E788F43E19E3095000725B73B?method=download&shareKey=119adac5a686d3159617a8723df26aac)

支持三种不同的监控方式：

- 默认的集群监控：通过URL http://turbine-hostname:port/turbine.stream开启，实现对默认集群的监控
- 指定的集群监控：通过URL http://turbine-hostname:port/turbine.stream?cluster=[clusterName]开启，实现对clusterName集群的监控
- 单体应用的监控：通过URL http://turbine-hostname:port/hystrix.stream开启，实现对具体单个服务实例的监控

如果是监控多个集群应用，方法有点不一样，主要是消费者的配置文件要加点东西。跟着我看一下吧

**turbine配置文件**：

```yaml
turbine:
  aggregator:
    cluster-config: mycluster   #自定义集群名称
  app-config: eureka-feign-client    #,eureka-ribbon-client
  cluster-name-expression: metadata['cluster']   #固定的
  combine-host-port: true
  instanceUrlSuffix: /hystrix.stream
```

消费者的配置文件：

```yaml
instance:
	metadata-map:
		cluster: mycluster
```

**ribbon和Feign配置**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
```
新版本的Spring Cloud 因为没有该/hystrix.stream的访问路径，所以需要配置一下

```java
@Configuration
public class ServletConfig {
    @Bean
    public ServletRegistrationBean getServlet() {
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
}
```



#### Turbine监控

用这个Turbine的想法很简单，一个词叫做聚合。把多实例放在一起来统计、管理、监控

![](https://note.youdao.com/yws/api/personal/file/A9865FFDCAFC42F0A085B3AF1BC471F8?method=download&shareKey=457b23f146ae621a6232468c56e360d0)

- pom.xml

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
</dependency>
```

- 在启动类上面加注解@EnableTurbine开启Turbine

- 配置文件

  ```yaml
  turbine:
    aggregator:
      cluster-config: default 
    app-config: eureka-ribbon-client,eureka-feign-client
    cluster-name-expression: new String("default")
    combine-host-port: true
    instanceUrlSuffix: /hystrix.stream
  ```

  > aggregator.cluster-config：指定聚合哪些集群，多个使用","分割，默认为default。可使用http://.../turbine.stream?cluster={clusterConfig之一}访问
>
  > app-config：需要收集监控信息的服务名
>
  > cluster-name-expression：指定集群名称为default，如果为default时，turbine.aggregator.clusterConfig可以不写，因为默认就是default
>
  > combine-host-port：设置为true，可以让一台主机上的服务通过主机名和端口号组合来区分。默认是通过host来区分不同的服务的。

- 启动Turbine

  重启工程一切正常，打开 <http://localhost:port/hystrix> 然后输入监控地址，因为前面我们使用的是default cluster所以这里输入：<http://localhost:port/turbine.stream>

  点击 Monitor Stream 按钮后，需要访问 <http://localhost:8764/testRibbon?name=rocye> 和 <http://localhost:port/testRibbon?name=rocye> 可以多刷新几次，可以看到如下效果

- 仪表盘参数详细说明

  ![](https://note.youdao.com/yws/api/personal/file/C73E4EBB5727418E882320996021F3CB?method=download&shareKey=5208d35b9af6bb5075a219aa664be07e)

#### Turbine Stream



#### 遇到的问题：

1. Unable to connect to Command Metric Stream

解决方案：

```java
@Configuration
public class DashBoardConfig {

    @Bean
    public ServletRegistrationBean servletRegistrationBean() {
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
}
```

2. Turbine404问题

   > ```
   > com.netflix.turbine.monitor.instance.InstanceMonitor$MisconfiguredHostException:
   > [{"timestamp":"2018-06-28T08:01:38.908+0000","status":404,"error":"Not Found","message":"No message available","path":"/actuator/hystrix.stream"}]
   > ```

解决办法：

```yaml
turbine:
  #aggregator:
    #cluster-config: default
  app-config: eureka-feign-client    #,eureka-ribbon-client
  cluster-name-expression: new String("default") #metadata['cluster']
  combine-host-port: true
  instanceUrlSuffix: /hystrix.stream   #一定要加这个
```

