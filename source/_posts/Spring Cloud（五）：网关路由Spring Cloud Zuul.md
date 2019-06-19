---
title: Spring Cloud（五）：Spring Cloud Zuul
date: 2017-03-07 13:25:24
categories: Spring Cloud
tags: [Spring Cloud]
comments: true
toc: true
---

# 一、什么是Spring Cloud Zuul？

Zuul作为路由网关组件，将所有服务的API接口统一聚合，并且一起对外暴露。（路由转发和过滤器）外界不用知道内部的各个服务相互调用的复杂性，从而保护了内部微服务单元的API接口，防止被外界直接调用，导致服务的敏感信息暴露。Zuul是Netflix出品的一个基于JVM路由和服务端的负载均衡器

![](https://note.youdao.com/yws/api/personal/file/A987B794E9974665AE54954733F623B7?method=download&shareKey=a7ab1832575b4659f6b5fb8a98ce65f7)

# 二、为什么需要API Gateway

1、简化客户端调用复杂度

​	实现相关的认证逻辑从而简化内部服务之间相互调用的复杂度

2、数据裁剪以及聚合

​	对通用性的响应数据进行裁剪以适应不同客户端的使用需求，可以将多个API调用逻辑进行聚合，减少客户端的请求数，优化客户端用户体验

3、多渠道支持

# 三、工作原理

Zuul是通过Servlet实现的，通过自定义的ZuulServlet（这一点有点像springmvc的dispatchServlet）来对请求进行控制。ZuulServlet 的作用是初始化ZuulFilter，并编排这些ZuulFilter 的执行顺序。Zuul的核心就是一堆的过滤器，可以在Http请求和响应执行一堆过滤器。

**大概流程应该是这样的**：当一个客户端Request 请求进入Zuul 网关服务时，网关先进入“**pre filter**”，进行一系列的验证、操作或者判断。然后给“**routing filter** ”进行路由转发，转发到具体的服务实例进行逻辑处理、返回数据。当具体的服务处理完后，最后由“**post filter** “进行处理， 该类型的处理器处理完之后，将Response 信息返回给客户端。



# 四、准备工作

通过配置文件我们知道，Spring Cloud Zuul 将自己作为一个服务注册到了Eureka。这也就意味着Zuul可以拿到所有注册到Eureka的其他服务的信息。Zuul为这些服务创建了默认的路由规则：**/{servicename}/\****

POM 配置

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka</artifactId>
</dependency>

```

## 4.1、配置文件

```yaml
zuul:
  routes:
    hiapi:  #自己定义的
      path: /hiapi/**    #就可以将指定类型的请求Uri 路由到指定的Serviceld
      serviceid: eureka-client
    ribbonapi:
      path: /ribbonapi/**
      serviceid: eureka-ribbon-client
    feignapi:
      path: /feignapi/**
      serviceid : eureka-feign-client
```

## 4.2、启动类

```java
@EnableZuulProxy
@EnableEurekaClient
@SpringBootApplication
public class ApiGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}
```

## 4.3、网关的默认路由规则

Zuul的路由的默认规则为：`http://ZUUL_HOST:ZUUL_PORT/微服务在Eureka上的serviceId/**`，会被转发到serviceId对应的服务

# 五、服务过滤

Filter是Zuul的核心，用来实现对外服务的控制。Filter的生命周期有4个，分别是“PRE”、“ROUTING”、“POST”、“ERROR”，整个生命周期可以用下图来表示。

- **PRE：** 这种过滤器在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择请求的微服务、记录调试信息等。
- **ROUTING：**这种过滤器将请求路由到微服务。这种过滤器用于构建发送给微服务的请求，并使用Apache HttpClient或Netfilx Ribbon请求微服务。
- **POST：**这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准的HTTP Header、收集统计信息和指标、将响应从微服务发送给客户端等。
- **ERROR：**在其他阶段发生错误时执行该过滤器。 除了默认的过滤器类型，Zuul还允许我们创建自定义的过滤器类型。例如，我们可以定制一种STATIC类型的过滤器，直接在Zuul中生成响应，而不将请求转发到后端的微服务。

## 5.1、Zuul中默认实现的Filter:

| 类型  | 顺序 | 过滤器                  | 功能                       |
| ----- | ---- | ----------------------- | -------------------------- |
| pre   | -3   | ServletDetectionFilter  | 标记处理Servlet的类型      |
| pre   | -2   | Servlet30WrapperFilter  | 包装HttpServletRequest请求 |
| pre   | -1   | FormBodyWrapperFilter   | 包装请求体                 |
| route | 1    | DebugFilter             | 标记调试标志               |
| route | 5    | PreDecorationFilter     | 处理请求上下文供后续使用   |
| route | 10   | RibbonRoutingFilter     | serviceId请求转发          |
| route | 100  | SimpleHostRoutingFilter | url请求转发                |
| route | 500  | SendForwardFilter       | forward请求转发            |
| post  | 0    | SendErrorFilter         | 处理有错误的请求响应       |
| post  | 1000 | SendResponseFilter      | 处理正常的请求响应         |

禁用Filter

```yml
zuul:
	FormBodyWrapperFilter:
		pre:
			disable: true
```

## 5.2、自定义Filter


```java
@Component
public class MyFilter extends ZuulFilter {

    private static Logger log = LoggerFactory.getLogger(MyFilter.class);
    
    /*
    filterType：返回一个字符串代表过滤器的类型，在zuul中定义了四种不同生命周期的过滤器类型，具体如下：
    pre：路由之前
    routing：路由之时
    post： 路由之后
    error：发送错误调用
    */
    @Override
    public String filterType() {
        return "pre";
    }
	
    //filterOrder：过滤的顺序
    @Override
    public int filterOrder() {
        return 0;
    }
	
    //shouldFilter：这里可以写逻辑判断，是否要过滤，本文true,永远过滤。
    @Override
    public boolean shouldFilter() {
        return true;
    }

    //run：过滤器的具体逻辑。可用很复杂，包括查sql，nosql去判断该请求到底有没有权限访问。
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        log.info(String.format("%s >>> %s", request.getMethod(), request.getRequestURL().toString()));
        Object accessToken = request.getParameter("token");
        if(accessToken == null) {
            log.warn("token is empty");
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);
            try {
                ctx.getResponse().getWriter().write("token is empty");
            }catch (Exception e){}

            return null;
        }
        log.info("ok");
        return null;
    }
}

```

5.2、