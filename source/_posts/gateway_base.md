---
title: Gateway 知识点
date: 2024-10-25 14:24:33
layout: post
categories: [Gateway 知识点整理]
tags: [微服务]
comments: true
cover: /img/springcloud.png
description: Gateway 整理知识点
---



# 1.Gateway 网关基础概念

文档地址：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-after-route-predicate-factory

- 什么是网关？理解成火车站的检票口，**统一**去检票
- 网关的优点：统一去进行一些操作，处理一些问题

## 网关作用

- **路由**：转发作用，转发请求到对应的接口（服务器 / 集群）

https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-after-route-predicate-factory

- **负载均衡**：在路由的基础上，随机转发到对应的一个服务器
- **统一鉴权**：判断用户是否有权进行操作
- **统一处理 跨域**：网关统一处理跨域，不用在每个项目中单独处理

https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#global-cors-configuration

- **统一业务处理**：把每个项目中都要做的通用逻辑放到上层（网关），统一处理，比如该项目中的次数统计
- **访问控制**：黑白名单，比如限制 DDOS IP
- **发布控制**：灰度发布，比如上线新接口，先给新接口分配 20% 的流量，老接口 80% ，在来调整。

https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-weight-route-predicate-factory

- **流量染色**：给请求（流量）添加一些标识，一般设置请求头中，添加新的请求头

https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-addrequestheader-gatewayfilter-factory

全局染色：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#default-filters

- **接口保护**：
  1. 限制请求：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#requestheadersize-gatewayfilter-factory
  2. 信息脱敏：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-removerequestheader-gatewayfilter-factory
  3. 降级（熔断）：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#fallback-headers
  4. 限流（学习：令牌桶算法、漏桶算法、RedisLimitHandler）：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-requestratelimiter-gatewayfilter-factory
  5. 超时时间：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#http-timeouts-configuration
  6. 重试（业务保护）：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-retry-gatewayfilter-factory
- **统一文档**：将下游的文档进行聚合，在同一个界面进行查询
- **统一日志**



## 网关类型

- 全局网关（接入层网关）：作用是负载均衡、请求日志等，不和业务逻辑绑定
- 业务网关（微服务网关）：会有一些业务逻辑，作用是将请求转发到不同的 业务 / 项目 / 接口 / 服务

参考文章：https://blog.csdn.net/qq_21040559/article/details/122961395



## 实现

- Nignx（全局网关）、Kong网关（API网关，Kong），编程成本相对较高
- Spring Cloud Gateway（取代了 Zuul）性能高，可以用 Java 代码来写逻辑，适用于学习

参考文章：https://zhuanlan.zhihu.com/p/500587132



## 核心概念

- 路由（根据什么条件，转发请求到哪里）
- 断言（一组规则、条件，用来确定如何转发路由）
- 过滤器：对请求进行一系列的处理，比如 添加请求头、添加请求参数、



## 请求流程

- 客户端发起请求
- Handler Mapping：根据断言，去将请求转发到对应的路由
- Web Handler： 处理请求（一层层过滤器）
- 实际调用服务



## 两种配置方式

- 配置式（方便、规范）
  - 简化版
  - 全称版
- 编程式（更为灵活）



## 断言

- After 在 xx 时间之后
- Before 在 xx 时间之后
- Between 在 xx 时间之间
- 请求类别
- 请求头（包含 Cookie）
- 查询参数
- 客户端地址
- **权重**

```java
spring:
  cloud:
    gateway:
      routes:
        - id: after_route
          uri: https://www.baidu.com
          predicates:
            - After=2017-01-20T17:42:47.789-07:00[America/Denver] # 可以再写个Before
```

> 建议开启日志：
>
> ```yml
> logging:
> level:
>  org:
>    springframework:
>      cloud:
>        gateway: trace
> ```



## 过滤器

- 添加请求头

- 添加请求参数

- 添加响应头

- 降级

  - 引入依赖

  ```xml
    <!-- 降级库、触发降级逻辑 -->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
       </dependency>
  ```

  



# 2.Gateway 基础使用

导入依赖 + 写配置 + 使用（yml / 代码式）

1. 导依赖：

有个**大坑**，就是 nacos 2021 及以后的版本，移除了 Ribbon 默认的负载均衡，必须要导入 loadbalancer 负载均衡依赖，不然能正常启动正常注册但是用的时候，会报找不到服务的错误 503

```xml
   <dependencies>
        <!-- 网关依赖配置管理 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <!-- 负载均衡 -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-loadbalancer</artifactId>
                <version>3.1.7</version>
            </dependency>

            <!-- nacos注册中心 -->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
                <version>2021.0.5.0</version>
            </dependency>
    </dependencies>
```

2. yml 配置

```yml
server:
  port: 10010

spring:
  profiles:
    active: dev
  autoconfigure:
    exclude: org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration # 禁用数据源
  main:
    web-application-type: reactive
  application:
    name: gateway-service # 服务名称
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 # nacos 地址
```

3. 使用

3.1、yml 配置使用：

最好参见官方文档（个人认为 Spring 官网这个文档真不错）：

https://cloud.spring.io/spring-cloud-gateway/reference/html/

```yml
spring:
  cloud:
    gateway:
      default-fi..: 默认过滤规则....
      routes: # 网关路由配置 -- 路由转发
        - id: learn-service8001
          uri: lb://learn-service8001
          predicates: # 断言 ---- 判断请求是否符合规则
            - Path=/8001/**
          fi...: 过滤...
          
        - id: learn-service8002
          uri: lb://learn-service8002
          predicates:
            - Path=/8002/**
```

3.2、代码式过滤器实现：

- 创建类，实现 **GlobalFilter** 接口
- 实现 filter 方法，实现过滤规则

参考代码：

```java
@Component
@Order(-1) // 设置优先级，值越小，优先级越低
public class AuthorzationFilter implements GlobalFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest().getHeaders().get("authorization").get(0);
        // 测 jwt
        if ("admin".equals(token)){
            // 放行
            return chain.filter(exchange);
        }
        // 拦截
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
        return exchange.getResponse().setComplete();
    }
}
```

4. 处理跨域问题：

```yml
spring:
  cloud:
    gateway:
      globalcors: # 全局的跨域处理
        add-to-simple-url-handler-mapping: true # 解决 options 请求被拦截问题
        cors-configurations:
          '[/**]':
            allowed-origins: # 允许哪些网站的跨域请求
              - "http://localhost:8090"
            allowed-methods: # 允许的跨域 ajax 的请求方式
              - "GET"
              - "POST"
              - "PUT"
              - "OPTIONS"
              - "DELETE"
              - "PATCH"
            allowed-headers: "*" # 允许在请求中携带的头信息
            allow-credentials: true # 是否允许携带 cookie
            max-age: 360000 # 这次跨域检测的有效期
```



# 3. 相关的限流算法

## 计数器算法？

**介绍：**略

**问题：**问题在于，设置 1 分钟 100 次访问，我可以直接 **00:59** 和 **01:00** 时候直接各发 100 个请求，直接 2 秒 200 个。



## 漏桶令牌算法？

**介绍：**略

**问题：**无法解决突发流量问题



## 令牌捅算法？

令牌桶算法是一种流量控制算法，用于限制系统的访问频率。该算法允许以固定的速率向”桶“中加入令牌，处理请求时消耗桶中的令牌。当桶中的令牌耗尽时，后续请求会拒绝或延迟处理。

在 Java 中可以使用基于 Guava 的 **`RateLimiter`** 实现令牌桶算法，可以有效控制单用户的访问频率，避免恶意行为。

**令牌桶算法的工作原理**

令牌桶算法的核心思想是通过一个”桶“来控制数据流量的速率：

1. **令牌生成：**系统以固定的速率生成令牌，令牌被放入桶中。生成的速率可以根据需求进行配置，例如每秒生成一定数量的令牌。
2. **令牌存储：**桶中可以存储一定数量的令牌。这个数量被称为”桶容量“或”最大容量“。当桶满时，多余的令牌将会被丢弃。
3. **请求处理：**每当生成一个请求到达系统的时，需要从桶中取出一个令牌。如果桶中有足够的令牌，允许请求通过；如果没有足够的令牌，请求会被拒绝或者等等令牌的生成。
4. **速率控制：**由于令牌是以固定速率生成的，因此系统能够控制请求的速率。例如，如果每秒生成 10 个令牌，并且桶的容量为 100，那么系统每秒最多允许处理 10 个请求，但如果有更多的请求到达，可以在桶中缓存令牌。

**优点：**

- 平滑的流量控制：令牌算法能够平滑处理请求流量，避免了突发流量对系统造成的冲击
- 突发流量处理：由于桶容量可以缓冲突发流量，系统可以在短时间内处理跟多的请求，而不会被立即拒绝
- 灵活性：通过调整令牌的生成速率和桶的大小，可以灵活地控制流量速率和突发流量的处理能力。

**注意事项：**

- 桶容量设置：如果桶的容量设置过小，可能会导致无法处理正常的突发流量；如果设置过大，则可能会积累过多的流量，超出系统处理能力。
- 生成速率调优：令牌生成速率直接影响系统的处理能力。如果速率设置过低，可能无法满足用户的需求；如果速率设置过高，可能会直接导致系统负担过重。

- 时间同步问题：在分布式系统中，时间同步问题可能影响令牌的精确生成，导致限流效果不稳定。

> 扩展知识
>
> **Guava RateLimiter 实现**
>
> 使用 Guava 的 **`RateLimiter`** 实现令牌桶算法时，可以通过以下步骤来限制访问频率：
>
> ```java
> import com.google.common.util.concurrent.RateLimiter;
> 
> public class RateLimitExample {
> 
>     // 创建一个 RateLimiter，设置每秒生成 5 个令牌
>     private static final RateLimiter rateLimiter = RateLimiter.create(5.0);
> 
>     public static void main(String[] args) {
>         // 模拟请求处理
>         for (int i = 0; i < 10; i++) {
>             // 请求获取令牌
>             rateLimiter.acquire();
>             // 处理请求
>             System.out.println("Request " + i + " processed");
>         }
>     }
> }
> ```



# 4.Gateway 编码式

> 写这个的主要思想，不会的就看 Gateway 网关已经实现的现成代码，换个实现逻辑就ok啦

## 自定义路由断言

为什么要自定义捏？当然是配置式不够满足业务

案例代码我写好了，看注释嗷：

```java
package com.linyu.predicates;

import lombok.Getter;
import lombok.Setter;
import org.springframework.cloud.gateway.handler.predicate.AbstractRoutePredicateFactory;
import org.springframework.stereotype.Component;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.server.ServerWebExchange;

import javax.validation.constraints.NotEmpty;
import java.util.Collections;
import java.util.List;
import java.util.function.Predicate;

/**
 * 自定义路由规则
 *
 * 创建参考：
 *   1. 参考现成的断路器： AfterRoutePredicateFactory 这个类
 *   2. 要么继承 AbstractRoutePredicateFactory  , 要么实现 RoutePredicateFactory 接口
 *   3. 开头名字任意，但是必须以 RoutePredicateFactory 后缀结尾
 *
 * 自定义路由断言正常套路：
 *   1. 创建断路器：public class MyRoutePredicateFactory extends AbstractRoutePredicateFactory<MyRoutePredicateFactory.Config>
 *       !!!!! 类名必须为  XXX + RoutePredicateFactory 格式 ！！！！
 *   2. 重写 apply 方法
 *   3. 新建 Config 内部类：用于断言规则，非常重要
 *   4. 空参构造方法，内部调用 upser
 *   5. 重写 apply 方法（真正的去实现路由断言逻辑）,直接创建返回对象，重写 test 方法
 *
 * 使用：
 *   1. yml 配置文件直接写
 *   2. routes: # spring.cloud.gateway 下的啊
 *         - id: pay_routh1 #pay_routh1
 *         uri: lb://cloud-payment-service
 *         predicates:
 *             # - Path=/pay/** # 路径
 *              - My=diamond # 左侧为自定义路由断言的类名（My 为短写，必须在类里面开启 短格式支持，右侧为值）
 *
 * @author 23087
 */
@Component
public class MyRoutePredicateFactory extends  AbstractRoutePredicateFactory<MyRoutePredicateFactory.Config>
{
    public MyRoutePredicateFactory() {
        super(MyRoutePredicateFactory.Config.class);
    }

    public MyRoutePredicateFactory(Class<Config> configClass) {
        super(configClass);
    }

    @Getter
    @Setter
    @Validated
    public static class Config {
        // VIP 用户类型：砖、金、银等用户等级
        @NotEmpty
        private String userType;
    }

    @Override
    public Predicate<ServerWebExchange> apply(MyRoutePredicateFactory.Config config) {
        return new Predicate<ServerWebExchange>(){
            @Override
            public boolean test(ServerWebExchange serverWebExchange) {
                // 检查request的参数里面，userType是否为指定的值，符合配置就通过
                String userType = serverWebExchange.getRequest().getQueryParams().getFirst("userType");
                if (userType == null){
                    return false;
                }
                // 如果说参数存在，就和 config 的数据进行比较
                if (userType.equals(config.getUserType())){
                    return true;
                }
                return false;
            }
        };
    }

    /**
     * 必须开启支持短格式
     *
     *    !!!!! 类名必须为  XXX + RoutePredicateFactory 格式 ！！！！
     *    像这种，yml 配置断路器的时候，可以直接缩写为 XXX 即可
     *    不开启的话必须写全面 XXXRoutePredicateFactory
     * @return
     */
    @Override
    public List<String> shortcutFieldOrder() {
        return Collections.singletonList("userType");
    }

}
```



## 自定义过滤器

**过滤器类型：**

- 全局默认过滤器 Global Filters：
  - gateway 出厂默认已有的，直接用即可，主要作用于 **所有的路由**
  - 不需要再配置文件中配置，作用在所有的路由，**实现 GlobalFilter 接口即可**
- 单一内置过滤器 GatewayFilter：
  - 也可以称为网关过滤器，这种过滤器主要是作用于单一路由或者某个路由分组
- 自定义过滤器

**这里只讲自定义过滤器嗷，gateway内置的过滤器看官方文档**

1. ### 自定义全局过滤器

```java
package com.linyu.gateway;

import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.annotation.Order;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

/**
 * @author 23087
 */
@Component
@Order(-1) // 设置优先级，值越小，优先级越低
public class AuthorzationFilter implements GlobalFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest().getHeaders().get("authorization").get(0);
        // 测 jwt
        if ("admin".equals(token)){
            // 放行
            return chain.filter(exchange);
        }
        // 拦截
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
        return exchange.getResponse().setComplete();
    }
}
```

> 注意：设置过滤器优先级（设置数值 **越低**，优先级 **越高**）：
>
> - 可以选择注解式：@Order(-1)
> - 也可以选择继承接口 Order 实现，返回 数值就ok

2. ### 自定义条件过滤器

案例代码写好了，看注释嗷，都写好了

```java
package com.linyu.filter;

import lombok.Getter;
import lombok.Setter;
import org.springframework.cloud.gateway.filter.GatewayFilter;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.factory.AbstractGatewayFilterFactory;
import org.springframework.http.HttpStatus;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

import java.util.ArrayList;
import java.util.List;

/**
 * 自定义条件构造器：单一内置过滤器GatewayFilter
 *     第一步首先是参考 Gateway 已有的代码了：
 *              1. SetStatusGatewayFilterFactory
 *              2. SetPathGatewayFilterFactory
 *              3. AddResponseHeaderGatewayFilterFactory
 *
 * 自定义条件过滤器步骤：
 *      1. 新建过滤器类，跟那个自定义路由断言一个套路， XXX + GatewayFilterFactory 结尾
 *      2. 继承 AbstractGatewayFilterFactory 类
 *      3. 新建 Config 内部类
 *      4. 重写 apply 方法
 *      5. 重写 shortcutFieldOrder 方法支持短格式配置
 *      6. 空参构造方法，内部调用 super
 *      7. 重写 apply 方法（真正的去实现过滤器的逻辑）
 *
 * yml配置使用：
 *      1. 首先当然是参考现成的啦：直接参考这个 AddResponseHeaderGatewayFilterFactory
 *      spring.cloud.gateway.routes:
 *              - id: learn-service8001
 *                uri: lb://learn-service8001
 *                predicates:
 *                  - Path=/8001/**
 *                filters:
 *                  - AddResponseHeader=XXX,linyu
 *      2. 那我自己的当然就是（下面这里搞一个）：
 *            filters:
 *              - AddResponseHeader=X-Request-xiaoyu,linyu # 请求头 kv，若一头含有多参数则重写一行设置
 *              - My=linyu
 *
 * @author 23087
 */
public class MyGatewayFilterFactory extends AbstractGatewayFilterFactory<MyGatewayFilterFactory.Config> {

    public MyGatewayFilterFactory(){
        super(MyGatewayFilterFactory.Config.class);
    }

    /**
     * 自定义过滤规则，从请求参数上拿嗷
     * @param config
     * @return
     */
    @Override
    public GatewayFilter apply(MyGatewayFilterFactory.Config config) {
        return new GatewayFilter(){
            @Override
            public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
                ServerHttpRequest request = exchange.getRequest();
                System.out.println("进入了自定义网关过滤器MyGatewayFilterFactory，status:" + config.getStatus());
                if (request.getQueryParams().containsKey("linyu")){
                    return chain.filter(exchange);
                }else {
                    exchange.getResponse().setStatusCode(HttpStatus.BAD_REQUEST);
                    return exchange.getResponse().setComplete();
                }
            }
        };
    }

    @Setter
    @Getter
    public static class Config {
        //设定一个状态值/标志位，它等于多少，匹配和才可以访问
        private String status;
    }

    /**
     * 配置支持短格式
     * @return
     */
    @Override
    public List<String> shortcutFieldOrder() {
        List<String> list = new ArrayList<>();
        list.add("status");
        return list;
    }
}
```



# 5.关于 Gateway 网关的一些问题

## 什么是 Spring Cloud Gateway？

是 Spring Cloud 开源的网关组件。

在微服务架构中充当着前端入口点的角色，实现了 **路由转发、请求过滤、负载均衡** 等功能。

主要功能如下：

- **路由转发**：Spring Cloud Gateway 可以根据一开始定义的规则将接收到的请求转发到不同的微服务。这种路由功能使得客户端不需要直接了解各个微服务的地址，就可以实现服务的远程调用，提高了系统的可维护性和灵活性。
- **过滤器机制**：这个主要是借鉴了 Zuul 的过滤器模式，但是在其基础上进行了改进，即支持非阻塞式异步处理。过滤器链运行在请求的处理过程中添加各种预处理和后处理逻辑，比如身份验证、请求转发、限流、熔断、日志记录等功能，即针对服务的功能都可以通过添加过滤器来进行实现。
- **响应式编程**：Spring Cloud Gateway 使用了响应式编程框架 Webflux，充分发挥了响应式编程的优势，在处理大量并发请求的时候，其性能和可伸缩性较好。
- **生态集成良好**：Spring Cloud Gateway 由于其是基于 Spring 体系进行开发的，其与 Spring Cloud 生态兼容以及集成较好，可以结合 Nacos、Eureka 等组件，实现服务注册与发现、远程调用等方面的功能。
- **统一管理**：由于 Spring Cloud Gateway 是系统流量的统一入口，可以实现服务调用的统一管理，简化服务配置。



## Spring Cloud Gateway 的优势？

**高性能与异步处理：**

- **非阻塞架构：**基于 Spring WebFlux 和 Reactor ，Spring Cloud Gateway 使用 **异步非阻塞** 的编程模式，能够在高并发环境下保持较低的资源消耗和更高的吞吐量。
- **适合高并发场景：**与传统的同步阻塞框架相比，Spring Cloud Gateway 更适合处理大量并发请求的场景，如 **电商平台、流量高峰期间的 API 调用等**。

**与 Spring 生态的无缝集成：**

- Spring Cloud 生态：Spring Cloud Gateway 与 Eureka、Consul 等服务注册与发现工具可以无缝集成，实现微服务的 **动态路由**。
- **配置便捷：**支持 **YAML** 文件配置和 Java 编码配置，开发者可以根据需求选择适合的方式配置网关。

**灵活的路由与过滤器机制：**

- **路由机制：**Spring Cloud Gateway 支持通过 **请求路径、请求头、请求参数** 等多种方式进行路由转发，可以实现复杂的路由规则。
- **过滤器链：**支持丰富的内置过滤器（如添加请求头、重写路径、限流等），也支持自定义过滤器。过滤器可以在请求前后进行加工处理，满足多样化的业务需求。



## 为何选择 Gateway 作为网关？

1. 以前用的比较多的是 Spring Cloud 官方开源的 Zuul，其用的比较多的还是 1.0 版本，不过 1.0 版本 Netflix 很早就宣布进入维护状态了，虽然 Zuul 已经出来 2.0 版本，但是 Spring Cloud 官方团队好像没有整合的计划，并且 Netflix 很多相关组件，如 Eureka、Hystrix 都已经进入维护期了，所以不知道前景如何。然后 Spring Cloud 官方自己研发了 Spring Cloud 官方自己研发了 Spring Cloud Gateway，并且大力支持和推荐，所以相比于 Zuul，Gateway 是一个更好的选择。
2. Spring Cloud Gateway 有很多的特性：

- 基于 Spring 5 和 Spring Boot 2 进行构建，与 Spring 生态兼容较好
- 动态路由：可以根据配置条件，如 URL、请求方法等实现动态配置
- 可以对路由指定断言（Predicate）和 过滤器（Filter）
- 集成 Hystrix 断路器功能，可以实现服务熔断
- 集成 Spring Cloud 相关组件，如 Eureka、Nacos、Consule 等实现服务的注册与发现
- 内置了限流模块，可以根据配置实现服务限流



