---
title: Spring Cloud（一）：浅谈微服务
date: 2017-03-07 13:25:24
categories: Spring Cloud
tags: [Spring Cloud]
comments: true
toc: true
---

## 什么是微服务架构呢？

​	微服务是系统架构上的一种设计风格，它是将一个原本独立的系统拆分成多个小型服务，这些小型服务都在各自独立的进程中运行，服务之间通过基于HTTP的RESTful API进行通信协作。被拆分成的每一个小型服务都是围绕着系统中某一项或者一些耦合度较高业务功能进行构建，并且每个服务都维护自身的数据存储、业务开发、自动化测试以及独立的部署机制。

## Spring Cloud 简介

Spring Cloud 是一个基于Spring Boot 实现的微服务架构开发工具。它为微服务架构中涉及的配置管理、服务治理、断路器、智能路由、微代理、控制总线、全局锁、决策竞选、分布式会话和集群状态管理等操作提供一种简单的开发方式。

## Spring Cloud的优势

微服务的框架那么多比如：dubbo、Kubernetes，为什么就要使用Spring Cloud的呢？ 

- 产出于spring大家族，spring在企业级开发框架中无人能敌，来头很大，可以保证后续的更新、完善。
- 有Spring Boot 这个独立干将可以省很多事，大大小小的活Spring Boot都搞的挺不错。
- 作为一个微服务治理的大家伙，考虑的很全面，几乎服务治理的方方面面都考虑到了，方便开发开箱即用。
- Spring Cloud 活跃度很高，教程很丰富，遇到问题很容易找到解决方案
- 轻轻松松几行代码就完成了熔断、均衡负责、服务中心的各种平台功能

## 核心成员

### Spring Cloud Netflix

这可是个大boss，地位仅次于老大，老大各项服务依赖与它，与各种Netflix OSS组件集成，组成微服务的核心，它的小弟主要有Eureka, Hystrix, Zuul, Archaius… 太多了

- **Netflix Eureka** 
- **Netflix Hystrix** 
- **Netflix Zuul** 
- **Netflix Archaius** 

### Spring Cloud Config

### Spring Cloud Bus

### Spring Cloud for Cloud Foundry

### Spring Cloud Cluster

### Spring Cloud Consul