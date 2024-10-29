---
title: 微服务基础 知识点
date: 2024-10-22 13:36:25
layout: post
categories: [微服务基础 知识点整理]
tags: [微服务]
comments: true
cover: /img/cloud_logo.jpg
description: 微服务基础 整理知识点
---

> 约定 > 配置 > 编码
>

# 1. 认识微服务

## 需要学哪些微服务知识？

![](/img/微服务架构.png )

## 学习路线

![](/img/how_learning.png )

## 分布式架构

**分布式架构：**根据业务功能对系统进行拆分，每个业务模块作为独立项目开发，称为一个服务。

**优点：**

- 降低服务耦合
- 有利于服务升级拓展

**缺点：**

- 架构复杂，开发难度大
- 运维成本高



**微服务架构特征：**

- 单一职责：微服务拆分粒度更小，每一个服务都对应的唯一的业务能力，能做到单一职责，避免重复业务开发。
- 面向服务：微服务对外暴露业务接口。
- 自治：团队独立、技术独立、数据独立、部署独立。
- 隔离性强：服务调用做好隔离、容错、降级、避免出现级联问题。



# 2. 无伤速通版本

> **版本一：**
>
> - Java：Java17+
> - Cloud：2023.0.0
> - Boot：3.2.0
> - Cloud Alibaba：2022.0.0.0-RC2
> - Maven：3.9+
> - Mysql：8.0+







# 服务发现注册

## Nacos 详解

- 单节点启动(bin目录下): **startup.cmd -m standalone**



## Consul 

> - 服务注册 与 发现
> - 分布式配置管理

使用：

1. 去官网下文件：https://www.consul.io/

2. 查看是否适配：cmd 中 consul --version

3. 使用开发者模式启动：

   1. cmd 中 consul agent -dev

   2. 通过以下地址可以访问 Consul 的首页：

      http://localhost:8500

   3. 结果页面

> 在项目中使用：
>
> - 依赖配置
>
>   ```xml
>          <!--SpringCloud consul discovery -->
>           <dependency>
>               <groupId>org.springframework.cloud</groupId>
>               <artifactId>spring-cloud-starter-consul-discovery</artifactId>
>               <exclusions>
>                   <exclusion>
>                       <groupId>commons-logging</groupId>
>                       <artifactId>commons-logging</artifactId>
>                   </exclusion>
>               </exclusions>
>           </dependency>
>   ```
>
> - 启动类加注解
>
>   ```java
>   @EnableDiscoveryClient
>   ```
>
> - yml 配置文件
>
>   ```yml
>   server:
>     port: 80
>   
>   spring:
>     application:
>       name: cloud-consumer-order
>     ####Spring Cloud Consul for Service Discovery
>     cloud:
>       consul:
>         host: localhost
>         port: 8500
>         discovery:
>           service-name: ${spring.application.name}
>           prefer-ip-address: true # 优先使用服务 ip 进行注册 # 有些地方好像可以不写
>   ```
>
> - 调用使用
>
>   ```java
>   public static final String PaymentSrv_URL = "http://cloud-payment-service";
>   ```

注意：consul 天生支持分布式集群

需要加个 负载均衡 注解@LoadBalanced

```java
/**
 * @author 23087
 */
@Configuration
public class RestTemplateConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

}
```



注意（这里没记录，看一下xml）：

- 自动刷新（类似热部署）
- 本地持久化

> 主要用的是 openFeign，Consul 不怎么用来着





