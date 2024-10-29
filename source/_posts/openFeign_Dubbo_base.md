---
title: 微服务的远程调用
date: 2024-10-20 12:05:28
layout: post
categories: [opneFeign vs dubbo dubbo知识点整理]
tags: [微服务]
comments: true
cover: /img/feign_dubbo_logo.jpg
description: 微服务Dubbo简单 整理知识点
---





> ### 微服务远程调用 Dubbo 和 openFeign 怎么选择？优劣？
>
> **计算机网络七层模型中，越往下数据传输越稳定。**
>
> ![](/img/dubbo_feign_network.png )
>
> 我认为二者各有优缺点：
>
> - 首先 Dubbo 是基于 TCP 进行数据传输，处于网络模型更底层，所以数据传输相对更加稳定。它还是一个相对独立的 RPC 框架，提供完整的服务治理解决方案，适用于大型的分布式项目中.
> - openFeign 是基于 HTTP 进行数据传输,属于应用层,相对来讲数据传输受网络等其他因素影响较大一些.它是 springcloud 生态中的一部分.更适用于构建轻量级的微服务.
> - dubbo 虽然支持多种协议,但是需要显示的定义接口和实现类,配置各种参数.
> - openfeign 则更加关注于 RESTful 风格的接口调用,更加简单容易操作.





# 1. 两者的使用 Demo

## OpenFeign 的使用

JDK17 + Springboot3 的奥，不一样的话依赖版本需要改改

下面代码案例：

> 前提准备：依赖（openFeign + nacos注册中心 + 负载均衡） + 配置yml
>
> （注册中心可以换，随便了）
>
> 依赖（生产者和消费者都需要，版本也保证一致最好）：
>
> ```xml
>      <!-- openfeign -->
>         <dependency>
>             <groupId>org.springframework.cloud</groupId>
>             <artifactId>spring-cloud-starter-openfeign</artifactId>
>             <version>4.1.3</version>
>         </dependency>
>         <!-- 负载均衡 -->
>         <dependency>
>             <groupId>org.springframework.cloud</groupId>
>             <artifactId>spring-cloud-starter-loadbalancer</artifactId>
>             <version>4.1.0</version>
>         </dependency>
> 
>         <!-- nacos -->
>         <dependency>
>             <groupId>com.alibaba.cloud</groupId>
>             <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
>             <version>2022.0.0.0-RC2</version>
>         </dependency>
> ```
>
> 配置yml（服务提供者和服务消费者都要配都要配）：
>
> ```yml
> spring:
>   application:
>     name: service-provider # 服务名称（可以用服务模块名命名）
>   cloud:
>     nacos: 
>       discovery:
>         server-addr: localhost:8848 # 注册中心地址
> ```



1. 消费者和生产者启动类都要加注解：@EnableFeignClients
2. 定义生产者接口：

```java
@RestController
@RequestMapping("/order")
public class TbOrderController {

    @Autowired
    private ITbOrderService tbOrderService;
    /**
     * 服务提供测试
     * @return
     */
    @GetMapping("/test")
    public ResultData providerTest() {
        // 测试查询所有信息返回
        return ResultData.success(tbOrderService.getAll());
    }
}
```

3. 抽取公共模块定义 api：

需要请求方式和请求方法（参数）：

**@FeignClient(value = "service-provider")  // 必须加这个注解，里面的值是提供服务的服务名，用于到注册中心中找**

```java
@FeignClient(value = "service-provider")
public interface OrderFeignApi {

    @GetMapping("/order/test")
    public ResultData providerTest();

}
```

写实现类用于请求**失败降级**：

也要在 api 接口注解配置一下 @FeignClient(value = "service-provider")  这个里面，好像是，fallback，值是全类名.class（导包形式也ok）

```java
@Component
public class OrderFeignApiFailBack implements OrderFeignApi {
    @Override
    public ResultData providerTest() {
        return ResultData.fail(ReturnCodeEnum.RC500.getCode(),"对方服务宕机或不可用，FallBack服务降级o(╥﹏╥)o");
    }
}
```

4. 消费者直接调用：

```java
    @Autowired
    private OrderFeignApi orderFeignApi ;


    @GetMapping("/getorder")
    public ResultData getOrder(){
        ResultData resultData = orderFeignApi.providerTest();
        System.out.println("============== 远程调用成功 ==================");
        return ResultData.success(resultData.getData());
    }
```



## Dubbo 的使用

代码案例：

首先还是导入依赖（dubbo 、nacos 官方文档用的是zookeeper但是整合的时候优点问题） + 配置yml

依赖（这里最好用 java8 奥）：

```xml
       <!-- dubbo依赖管理 -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo</artifactId>
            <version>3.0.9</version>
        </dependency>
        <!-- nacos 依赖配置管理 -->
        <dependency>
            <groupId>com.alibaba.nacos</groupId>
            <artifactId>nacos-client</artifactId>
            <version>2.1.0</version>
        </dependency>
```

配置yml：

```yml
dubbo:
  application:
    name: dubbo-springboot-demo-provider
  protocol:
    name: dubbo
    port: -1
  registry:
    id: nacos-registry
    address: nacos://localhost:8848
```

1. 服务提供者和消费者都需要在**启动类**上加：**@EnableDubbo** 

2. 抽取远程调用服务接口（都写在一个包奥）：

```java
public interface TicketService {

    /**
     * 展示所有上线的电影票
     *
     * @return
     */
    List<Ticket> listAllOnlineTicket();
}
```

3. 实现远程调用接口（推荐 impl 包里面建一个 inner 包存放）：

```java
@DubboService
public class DubboTicketServiceImpl implements TicketService {

    @Override
    public List<Ticket> listAllOnlineTicket() {
        // 此处就不连数据库了，直接做一个假的数据返回
        Ticket ticket1 = new Ticket(1L, "票1", new BigDecimal(100));
        Ticket ticket2 = new Ticket(2L, "票2", new BigDecimal(200));
        Ticket ticket3 = new Ticket(3L, "票3", new BigDecimal(300));
        List<Ticket> list = Arrays.asList(ticket1, ticket2, ticket3);
        return list;
    }
}
```

4. 使用 （@DubboReference // 使用这个注解注入。注入后直接调用）：

```java
@Service
public class OrderServiceImpl implements OrderService {

    @DubboReference
    private TicketService ticketService;
    
    @Override
    public List<Ticket> listAllOnlineTicket() {
        return ticketService.listAllOnlineTicket();
    }
}
```

5. 服务降级：

// 在 远程调用接口 api 包下建一个 mock 包（跟 openFeign 类似）

```java
@Component
public class TicketServiceMock implements TicketService {

    @Override
    public List<Ticket> listAllOnlineTicket() {
        System.out.println("===== 降级处理 =====");
        return new ArrayList<>();
    }
}
```

```java
// 重点注意，需要在使用这个接口服务的时候配置上 mock 值为全类名
    @DubboReference(mock = "com.linyu.apiservice.mock.TicketServiceMock")
    private TicketService ticketService;
```



# 2. Dubbo 知识点

## Dubbo 的服务调用流程是怎样的?

宏观上可以分为以下几步:

1. **服务注册**：服务提供者启动时, 将自身的服务信息(如接口、方法、版本、地址等)注册到注册中心。
2. **服务订阅**：服务消费者在启动的时候，向注册中心订阅自己所需的服务。
3. **服务发现**：注册中心接收到服务消费者的订阅请求后，将服务提供者的地址信息推送给服务消费者。
4. **服务调用**：消费者根据注册中心返回的地址信息，通过负载均衡机制选择一个服务提供者进行远程调用。
5. **结果返回**：服务提供者执行请求逻辑后，将结果返回给服务消费者。

> 扩展知识：
>
> - **服务监控**：通过监控服务的调用过程和结果，囧事发现问题和瓶颈。
> - **容错机制**：包括失败重试、请求超时、断路器等机制，保障服务稳定性。
> - **智能路由**：根据服务提供者的状态和网络情况，自动调整路由策略，确保调用的高效和稳定性。



## Dubbo 支持哪些序列化方式？

包括但不限于以下：

- Hessian：**`Dubbo 默认的序列化方式`**，是一种二进制序列化协议。它的优点是速度快且序列化后的体积小。Hessian 适用于大多数场景，但如果需要更好的性能，可以选择其他的方式。
- JSON：JSON 是一种基于文本的序列化形式，具有良好的可读性和跨语言性，但它的序列化和反序列化速度较慢，序列化后的数据量较大。JSON 适用于需要与前端进行数据交换的场景，因为前端通常用 JavaScript 处理。
- Java 序列化：Java 自带的序列化机制，优化是使用简单，和 Java 语言紧密集成。缺点是序列化的速度较慢，序列化后的数据体积较大，因此并不常用于高性能需求场景。
- Kryo：一种高性能的序列化库，适用于速度和体积要求较高的场景。Kryo 的缺点是它的 API 较为复杂，使用起来需要一定的学习成本。
- Protobuf：是 Google 出品的一种高效可扩展的通信协议。它的优点是性能卓越，适合用在对性能要求极高且需要跨语言支持的场景。缺点是学习成本较高，需要定义 .proto 文件。
- FST（Fast-Serialization）：FST 也是一种高性能的序列化方式，和 Kryo 类似，速度快且序列化后的数据较小。适用于对性能有较高要求的场景。



## Dubbo 如何优化序列化性能？

优化 Dubbo 的序列化性能，可以从以下几个方面入手：

1. 选择更高效的序列化框架：选择性能更优的序列化框架，比如 Kryo、Protobuf、Hessian 等。
2. 避免不必要的序列化：只序列化必要的字段，避免序列化大小的对象或冗余数据。
3. 配置和调优参数：调整序列化框架的一些参数，比如缓冲区大小、缓存对象等。
4. 使用缓存机制：将已序列化的数据进行缓存，以减少重复序列化的开销。

> 另外可以参考的优化手段：
>
> - 压缩传输：对于大数据，可以在序列化后对其进行压缩，然后再进行网络传输。这样可以在减少数据量的同时加快传输速度。
> - 异步处理：在高并发场景下，通过异步序列化与反序列化操作，可以减少主线程的阻塞，提升系统的整体吞吐量。



## Dubbo 支持哪些负载均衡策略？

Dubbo 是 Apache 提供的一个高性能、轻量级的 Java RPC 框架。它支持多种负载均衡策略，每种策略都有其特定的使用场景和优劣。以下是 Dubbo 支持的主要负载均衡策略：

1. 随机（Random）：相当于随机选择一个服务提供者，适用于调用数比较均匀的场景。
2. 轮询（Round Robin）：按顺序轮询选择提供者。适用于请求量大且稳定的场景。
3. 最少活跃调用数（Least Active）：优先选择当前活跃调用数最少的提供者，能够有效避免请求集中到某个服务提供者。
4. 一致性哈希（Consistent Hash）：对于相同参数的请求总是发到同一提供者。适用于缓存等对请求的一致性要求高的场景。

> 扩展知识：
>
> 实现原理和使用场景。
>
> 1. 随机（Random）：这些策略实现的相对简单，在请求时随机选择一个 provider。它适用于负载较为均衡，且对性能要求不特别苛刻的场景。
> 2. 轮询（Round Robin）：在轮询策略中，Dubbo 会为每个请求按顺序选择不同的提供者。这种方式能够比较均匀地分发请求，但在负载特别大时，可能造成某些瞬间的过载。
> 3. 最少活跃调用数（Least Active）：这种策略在选择时会优先考虑当前处理请求数最少的提供者，是一种比较智能的负载均衡方法。它特别适合用于请求量比较大的场景，能够有效减缓热点问题，提高系统的稳定性。
> 4. 一致性哈希（Consistent Hash）：一致性哈希的核心思想是，将请求根据一定的规则（如参数哈希值）映射到固定节点上。这种策略特别适用于分布式缓存场景，比如在 Redis Client 或者 分布式文件系统中，因为它能够在提供者节点数量变化时，仍然保持相对高的请求命中率
>
> 此外,Dubbo 还允许用户通过实现 LoadBalance 接口自定义负载均衡策略,以应对更加复杂和特殊的业务需求.这些提供了很大的灵活性,可以根据具体的业务场景进行优化调整.



## Dubbo 和 Spring Cloud Gateway 有什么区别？

Dubbo 是一个 RPC （远程过程调用）框架，主要用于服务之间的通信。它提供高性能的 RPC 调用、负载均衡、服务发现、服务注册、服务治理等功能。

适用于需要高性能 RPC 调用的分布式系统，常用于内部服务通信.

Spring Cloud Gateway 是一个 API 网关,用于处理外部客户端请求并将其路由到后端服务.它提供请求路由、负载均衡、协议转换、安全管理、流量控制、日志和监控等功能。

适用于微服务架构中的统一入口管理，常用于外部请求的入口层。

所以说它们不是一个层级的东西。



# 3. 个人面经整理

## 你是如何在项目中使用 Dubbo RPC 框架的？

在使用 Dubbo 到项目前，我是先去 Dubbo 的官方文档，按照快速启动文档跑通了基础的 RPC 调用的 Demo，明确了注册中心、Maven 包等各依赖的版本号。

先在本地启动 Nacos 注册中心，然后在服务提供者和服务调用者项目引入 Dubbo 依赖（尽量引入相同的依赖和配置）、编写 Nacos 的连接配置、并且在项目启动类型通过 @EnableDubbo 注解开启 Dubbo 支持

编写服务提供者和服务调用客户端类，分别加上 @DubboService 和 @DubboReference 注解

并且编写了降级垫底实现类，在使用 @DubboReference 注解注入的时候，配置了 @DubboReference(mock = "降级实现类全类名")

有优先启动服务提供者项目，在 Nacos 控制台观察到服务注册信息，再启动服务调用者项目。



## Dubbo vs Spring Cloud

都能完全兼容 Spring 体系的应用开发模式。

两者虽然有很多相似之处，但由于它们在诞生背景与架构设计上的巨大差异，两者在性能、适用的微服务集群规模、生产稳定性保障、服务治理等方面都有很大差异。

**Spring Cloud 的优势在于：**

- 同样支持 Spring 开发体系的情况下，Spring Cloud 得到更多的原生支持。
- 对一些常用的微服务模式做了抽象如服务发现、动态配置、异步消息等，同时包括一些批处理任务、定时任务、持久化数据访问等领域也有涉猎。
- 基于 HTTP 的通信模式，加上相对比较完善的入门文档和演示 demo 和 starters，让开发者在第一感觉上更易于上手。

**Spring Cloud 的问题：**

- 只提供抽象模式的定义不提供官方稳定实现，开发者只能寻求类似 Netfix、Alibaba、Azure 等不同厂商的实现套件，而每个厂商支持的完善度、稳定性、活跃度各异。
- 有微服务全家桶却不是能拿来就用的全家桶，demo 上手容易，但落地推广与长期使用的成本非常高
- 欠缺服务治理能力，尤其是流量管控方面如负载均衡、流量路由方面能力都比较弱。
- 编程模型与通信协议绑定 HTTP，在性能，与其他 RPC 体系互通上存在障碍
- 总体架构与实现只适用于微服务集群时间，当集群规模增长后就会遇到地址推送效率、内存占用等各种瓶颈问题，但此时迁移到其他体系却很难实现
- 很多微服务实践场景的问题需要用户独自解决，比如优雅停机、启动预热、服务测试，再比如双注册、双订阅、延迟注册、服务按分组隔离、集群容错等。

而以上这些点，都是 Dubbo 的优势所在：

- 完全支持 Spring & Spring Boot 开发模式，同时再服务发现、动态配置等基础模式上提供与 Spring Cloud 对等的能力
- 是企业级微服务实践方案的整体输出，Dubbo 考虑到企业微服务实践中会遇到的各种问题如优雅下线、多注册中心、流量管理等，因此其在生产环境的长期维护成本更低
- 在通信协议和编码上选择更灵活，包括 rpc 通信层协议如 HTTP 、HTTP/2（Triple、gRPC）、TCP 二进制协议、rest等，序列化编码协议 Protobuf、JSON、Hessian2等，支持单端口多协议
- Dubbo 从设计上突出服务服务治理能力，如权重动态调整、标签路由等，支持 Proxyless 等多种模式接入 Service Mesh 体系 高性能的 RPC 协议编码与实现。
- Dubbo 是在超大规模服务集群实践场景下开发的框架，可以做到百万实例规模的集群水平扩容，应对集群增长带来的各种问题
- Dubbo 提供 Java 外的多语言实现，使得构建多语言异构的微服务体系成为可能