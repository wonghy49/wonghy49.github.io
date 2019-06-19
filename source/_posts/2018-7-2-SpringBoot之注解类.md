---
layout: post
title: Spring Boot之注解类
tags:  [Spring Boot]
categories: [Spring Boot]
author: wonghy
---

#### 	1、@ConfigurationProperties和@Value的区别

 ```java
例子：
@Component
//@ConfigurationProperties(prefix = "author")
public class User {
    private Integer id;

    private String name;

    private Integer age;
    
/*
    <bean class="User">
        <property name="name" value="字面量/${key}从环境变量、配置文件中获取/#{spEL}"></property>
    </bean>
*/
 ```
@Value用法：等同于上面的<property>中的value赋值

| 区别 |  @ConfigurationProperties    |   @Value   |
| :---: | :--: | :--: |
| 功能 |   批量注入配置文件中的属性   |  一个个指定    |
|  松散绑定（松散语法）     |   支持   |  不支持    |
|   SpEL    |  不支持    |   支持   |
|   JSR303数据校验（@Email等）   |   支持   |   不支持   |
如果说，只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value
如果说，专门编写一个javaBean来和配置文件进行映射，使用@ConfigurationProperties



#### 2、@PropertySource&@ImportSource
@PropertySource:加载指定的配置文件

```java
@Component
//@ConfigurationProperties(prefix = "author")
@PropertySource(value = {"classpath:person.properties"})
public class User {
    private Integer id;

    private String name;

    private Integer age;
```
@ImportSource：导入Spring的配置文件，让配置文件里面的内容生效

在Spring Boot中，对于我们自己编写的配置文件，是不能自动识别的；

想让Spring的配置文件生效，加载进去；需要将**@ImportSource**加载配置类上。

```java
@ImportSource(locations = {"classpath:beans.xml"})
导入Spring的配置文件让其生效
```
SpringBoot推荐给容器中添加组件的方式：推荐全注解
1、配置类，使用@Configuration设置为配置类
2、使用@Bean给容器添加组件
```java
@Configuration
public class MyConfig {
    //将方法中的返回值添加到容器中，容器中这种组件的id为方法名
    @Bean
    public HelloServiceImpl helloService(){
        System.out.println("配置类@Bean给容器中添加组件");
        return new HelloServiceImpl();
    }

}
```



3、