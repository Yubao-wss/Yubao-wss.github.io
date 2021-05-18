---
layout: post
title: SpringCloud
categories:
  - ACM
  - Template
tags: Posts
date: 2021-04-26 14:00:00
---

# SpringCloud

- 用于实现“微服务”架构，以及解决其问题的技术生态

## 微服务

- 简而言之，即模块化开发方式，把之前all in one（一站式）的系统，垂直拆分为若干模块
- 以更好的利用服务器资源，以及更合理的组织开发活动，最终为用户提供更多的价值

### 微服务架构需要解决的问题

1. 多个服务负载均衡的问题
2. 服务之间通信的问题
3. 服务崩溃的解决方法

## SpringCloud常见解决方案

- Spring Cloud NetFlix
- Apache Dubbo Zookeeper
- Spring Cloud Alibaba

### 技术栈一览

| 服务条目            | 落地技术                           |
| ------------------- | ---------------------------------- |
| 服务开发            | SpringBoot、SpringMVC              |
| 服务配置与管理      | Netfix的Archaius、阿里的Diamond    |
| 服务注册与发现      | Eureka、Consul、Zookeeper          |
| 服务调用            | Rest、RPC、gRPC                    |
| 服务熔断器          | Hystrix、Envoy                     |
| 负载均衡            | Ribbon、Nginx                      |
| 服务接口调用        | Feign                              |
| 消息队列            | Kafka、RabbitMQ、ActiveMQ          |
| 服务配置中心管理    | SpringCloudConfig、Chef            |
| 服务路由（API网关） | Zuul                               |
| 服务监控            | Zabbix、Nagios、Metrics、Specataor |
| 全链路追踪          | Zipkin、Brave、Dapper              |
| 服务部署            | Docker、OpenStack、Kubernetes      |
| 数据流操作开发包    | SpringCloud Stream                 |
| 事件消息总线        | SpringCloud Bus                    |

### Spring Cloud NetFlix

- 一个一站式解决方案
- 使用api网关、zuul组件
- 服务之间使用Http通信
- 使用Eureka实现服务注册与发现
- 利用Hystrix实现熔断机制

#### NetFlix项目架构图

