---
title: openFeign（nacos）—— 知识点
date: 2024-10-27 13:20:14
layout: post
categories: [openFeign（nacos） 知识点整理]
tags: [微服务]
comments: true
cover: /img/springcloud.png
description: openFeign（nacos） 整理知识点
---

# 1. 基础概念

## 什么是 Feign

Feign 是一个声明式的 Web 服务客户端

所谓的声明式就是指不需要编写复杂的关于 Http 请求的东西

只需要声明一个一个接口，然后在这个接口上添加一些注解，这些注解包括了请求的方法（如 GET 和 POST）、请求的 URL 等信息

Feign 在运行时通过注解和接口上定义的内容来动态构造和发送 Http 请求

所以使用 Feign，开发者只需要定义服务接口并通过注解指明服务名和参数等信息，Feign 就能自动完成 Http 请求的构建、发送和结果处理

Feign 也是 Spring Cloud Netflix 组件之一，结合 Spirng Cloud 的服务注册和发现、负载均衡等功能，能够让服务间的调用变得更加方便

总体来说，Feign 的主要特点有：

- 声明式的服务客户端，通过 Java 接口和注解构建服务客户端，简化了 Http 调用的使用过程，无需手动构建 HTTP 请求
- 很好地融入了 SpringCloud 生态，可以使用 SpringCloud 负载均衡、服务熔断等能力

> 为什么要有 Feign？
>
> 在项目中，经常需要调用第三方提供的 Http 接口，如 HttpClient等，需要自己手动的去拼 url，当请求多了，也会变得很麻烦
>
> Feign 是一个声明式的 Http 框架，当需要调用 Http 接口时，需要声明一个接口，加一些注解就可以了，组装参数、发送 Http 请求等重复性的工作都交给 Feign来完成



## Feign 的本质：动态代理 + 七大核心组件

Feign 的本质：动态代理 + 七大核心组件

Feign 底层其实是基于 JDK 动态代理来的

**Feign 一次 Http调用过程：**

![](/img/0017-feign_liucheng.png )

- 首先就是前面说的，进入 FeignInvocationHandler，找到方法对应的 SynchronousMethodHandler，调用 invoke 方法实现
- 之后根据 MethodMetadata 和 方法的入参，构建出一个 RequestTemplate，RequestTemplate 封装了 Http 请求的参数，在这个过程中，如果有请求体，那么会通过 Encoder 序列化
- 然后调用 RequestInterceptor，通过 RequestInterceptor 对 RequestTemplate 进行拦截扩展，可以对请求数据再进行修改
- 再然后将 RequestTemplate 转换成 Request，Request 其实跟 RequestTemplate 差不多，也是封装了 Http 请求的参数
- 接下来通过 Client 去根据 Request 中封装的 Http 请求参数，发送 Http 请求，得到响应的 Response
- 最后根据 Decoder，将响应体反序列化成方法返回值类型对象，返回





## Feign 和 OpenFeign 的区别？

Feign 和 OpenFeign 都是用于简化服务之间的 HTTP 调用的工具，让我们可以更加方便地实现服务间的通信

**Feign 最初是由 Netflix 开发的一个声明式 REST 客户端框架**，它的目标是让微服务之间的调用像调用本地方法一样容易

如果我们想调用其它服务的接口，可以创建一个接口，然后通过注解声明所需要调用的方法和路径，Feign 可以自动发送 Http 请求和接收响应，转换为方法返回值

而 **OpenFeign 是 Spring Cloud 在 Feign 的基础上进一步封装的**，它整合了 Spring Cloud 的特性，使得我们可以更加简单地使用 Feign，包括如下几个方面：

- 自动配置：OpenFeign 利用 Spring Boot 的自动装配机制，通过 @FeignClient 和 @ EnableFeignClinets 注解就可以创建一个 Feign 客户端，极大简化了 Feign 客户端的创建和配置过程
- 负载均衡：与 Spring Cloud 等服务发现组件集成，可以轻松实现客户端负载均衡
- Hystrix集成：只需要简单的配置就可以快速集成 Hystrix，提高系统的容错性



# 2. 基础使用

以下代码为 springboot2.7.6版本的 基础代码奥

1. #### 需要的依赖，服务提供者和调用者尽可能版本保证一致：

（springboot3 的依赖版本自己搞定，应该都用最新的就没问题了）

```xml
           <!-- openfeign -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-openfeign</artifactId>
                <version> 3.1.8</version>
            </dependency>
            <!-- 负载均衡（必须要有奥，不然跑不起来） -->
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
            <!-- nacos配置中心 -->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
                <version>2021.0.5.0</version>
            </dependency>
```

可以导入 httpclient5 提升性能：

```xml
            <!-- https://mvnrepository.com/artifact/org.apache.httpcomponents.client5/httpclient5 -->
            <dependency>
                <groupId>org.apache.httpcomponents.client5</groupId>
                <artifactId>httpclient5</artifactId>
                <version>5.2.1</version>
            </dependency>
```

2. #### 配置 yml

```yml
spring:
  profiles:
    active: dev
  application:
    name: learn-service8001 # 应用名称 --- nacos 注册服务名称默认用的就是这个
  cloud:
    nacos: 
      # 服务注册中心
      discovery:
        server-addr: localhost:8848 # 注册中心地址
        cluster-name: HZ # 集群名
        namespace: dev # 命名空间
      # 配置中心
      config:
        namespace: dev # 命名空间
        server-addr: localhost:8848 # 注册中心地址
        file-extension: yaml # 配置文件后缀
        import-check: # 引入检查
          enabled: false
        refresh-enabled: true # 刷新配置
  config: # 配置中心
    import: optional:nacos:${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
```

切换 http 协议及其它配置（原本是不支持线程池的）：

```yml
feign:
  httpclient: # 切换 http 协议
    hc5:
      enabled: true # 这是2.x的写法
    max-connections: 200 # 最大连接数
    max-connections-per-route: 50 # 每个路由的最大连接数
  client:
    config:
      default: # 全局生效，想要局部生效，直接写调用的服务名
#        logger-level: full # 全部日志
        connectTimeout: 60000 # 连接超时
        readTimeout: 60000 # 读取超时
```

3. #### 服务提供者和调用者的启动类加注解：**`@EnableFeignClients`**

（

当然下面抽取公共模块，启动类可能跟公共模块的包路径不太一样

@EnableFeignClients（basePackages="看要扫的包"）

）

4. #### 抽取需要远程调用的接口（公共模块包里写）：

直接粘贴接口的方法和请求方式路径即可

```java
@FeignClient(name = "learn-service8001") // 服务名
public interface LearningServiceApi {

    /**
     *  测试
     * @return
     */
    @PostMapping("/openfign/hc5")
    ResultData openfignHc5();
}
```

5. #### 调用

直接正常调用就行了

```java
@RestController
public class TestController {

    @Autowired
    private LearningServiceApi learningServiceApi;

    @GetMapping("/aaa/test")
    public String test() {
        learningServiceApi.openfignHc5();
        return "通过了";
    }
}
```

6. #### 支持服务降级：

也是在公共模块 common 包里面，api 包下创建一个 fallback 包，里面直接继承接口，实习降级垫底代码，然后在接口那个注解里面，指定降级的实现类就ok了

```java
@Component
public class LearningServiceApiFallBack implements LearningServiceApi {
    @Override
    public ResultData openfignHc5() {
        return ResultData.success("抱歉有点问题，稍后在试~");
    }
}
```

api 接口指定降级实现类

```java
@FeignClient(name = "learn-service8001", fallback = LearningServiceApiFallBack.class) // 服务名
public interface LearningServiceApi {

    /**
     *  测试
     * @return
     */
    @PostMapping("/openfign/hc5")
    ResultData openfignHc5();

}
```

7. #### 服务重试，默认是不会重试的，可以自己配置开启，**但是重试是危险的，建议禁止重试，走降级即可**

8. #### openfeign请求响应压缩

- gzip是一种数据格式，采用用deflate算法压缩数据；gzip是一种流行的数据压缩算法，应用十分广泛，尤其是在Linux平台。

- 当GZIP压缩到一个纯文本数据时，效果是非常明显的，大约可以减少70％以上的数据大小。

- 网络数据经过压缩后实际上降低了网络传输的字节数，最明显的好处就是可以加快网页加载的速度。


```yml
feign:
  compression:
    request:
      enabled: true # 开启请求压缩
      mime-types: text/xml,application/xml,application/json #触发压缩数据格式
      min-request-size: 1 #开启压缩的阈值，请求体大小，单位字节，默认2048，即是2k，这里为了演示效果设置成10字节
    response:
      enabled: true # 开启响应压缩
```

当然服务提供者和调用者都需要开：

```yml
server:
  compression:
    enabled: true #spring项目开启压缩
#    min-response-size: 2KB
  port: 8091
```

9. #### 开启日志（没太大必要）上面也有提及

```yml
feign:
  client:
    config:
      default: # 全局生效，想要局部生效，直接写调用的服务名
#        logger-level: full # 全部日志

```

就不介绍自定义类了

10. #### 切换负载均衡策略

默认的是：轮询

我没试奥，这里我直接找的别人资料

```java
public class DepartConfig {
    @Bean
    public ReactorLoadBalancer<ServiceInstance> randomLoadBalancer(Environment environment,
                                                                   LoadBalancerClientFactory factory) {
        //获取负载均衡客户端名称,即提供者服务名称
        String serverName = environment.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);
        //获取提供者微服务名称可用的实例列表
        ObjectProvider<ServiceInstanceListSupplier> lazyProvider = factory.getLazyProvider(serverName, ServiceInstanceListSupplier.class);
        //从集合lazyProvider中获取指定serverName的实例,随机做负载均衡
        return new RandomLoadBalancer(lazyProvider, serverName);
    }
}
```

- **在 DepartConfig 类中未见`@Configuration`注解,在启动类中需添加**`@LoadBalancerClients(defaultConfiguration = DepartConfig.class)`

> spring-cloud-loadbalancer 存在以下弊端
>
> 负载均衡策略较少
> 仅支持轮询和随机策略,默认是轮询策略
> 更换负载均衡策略方式较为麻烦
> 生产环境下使用的负载均衡器,通常是 dubbo
> dubbo 作为通讯客户端,负载均衡策略可供选择更加多样
> 无论是 openfegin 还是 resttemplate 建立通讯方式均是基于 http 协议进行
> http 协议效率较低,生产环境下使用远程过程调用协议 RPC 效率会更高
> 基于 RPC 通讯协议的框架恰好就有dubbo,此外还有 gRPC 等