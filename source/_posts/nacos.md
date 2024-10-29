---
title: Nacos 知识点
date: 2024-11-01 19:00:00
layout: post
categories: [Nacos 知识点整理]
tags: [微服务]
comments: true
cover: /img/0023-nacos_logo.png
description: Nacos 整理知识点
---

> 官网文档：https://nacos.io/zh-cn/docs/architecture.html
>

# 1. 基础概念

1. **Nacos全称：**Dynamic Naming and  Configuration Service  （翻译：动态命名和配置服务）
2. **Nacos是什么：**一个 `更易于构建云原生应用` 的**动态服务发现**、**配置管理**和**服务管理平台**。
3. Nacos ---->  注册中心 + 配置中心的组合
4. 等价于：
   - Nacos = Eureka + Config + Bus
   - Nacos = **Spring Cloud Consul**
5. 能干什么：
   - 替代 Eureka / Consul 做服务注册中心
   - 替代 （Config + Bus）/ Consul 做服务配置中心和满足动态刷新广播通知
6. 各种注册中心比较：

| **服务注册与发现框架** | **CPA模型**                                                  | **控制台管理** | **社区活跃度**    |
| ---------------------- | ------------------------------------------------------------ | -------------- | ----------------- |
| Eureka                 | AP                                                           | 支持           | 低（2.x版本闭源） |
| Zookeeper              | CP                                                           | 不支持         | 中                |
| Consul                 | CP                                                           | 支持           | 高                |
| Nacos                  | AP（Nacos默认是**AP**模式，但也可以调整切换为**CP**，我们一般用默认AP即可） | 支持           | 高                |

> ## **CAP 模型介绍：**
>
> ​		**CAP**定理指出，分布式系统无法同时保证**一致性（Consistency）**、**可用性（Availability）**和**分区容错性（Partition tolerance）**。在分区通信可能失败的情况下，通常需要在一致性与可用性之间做出选择。当追求一致性时，可能导致部分节点不可用；而保证可用性则可能牺牲一致性。CAP是理解分布式系统设计基础的重要理论。
>
> **分布式系统的三个指标：**
>
> ![](/img/0018-CAP.png )
>
> - **分区容错（Partition tolerance）：**大多数分布式系统都分布在多个子网络。每个子网络就叫做一个区（partition）。分区容错的意思是，区间通信可能失败。（比如，一台服务器放在中国，另一台服务器放在美国，这就是两个区，它们之间可能无法通信） -------- CAP理论要求 P 总是同时成立的
> - **一致性（Consistency）：**写操作之后的读操作，必须返回该值。
> - **可用性（Availability）：**只要收到用户的请求，服务器就必须给出回应。
>
> 
>
> #### Consistency 和 Availability 的矛盾，为什么 一致性 和 可用性 不能同时成立？
>
> 因为通信可能失败（即：分区容错性），自己理解嗷，总结来说 CAP 只能同时成立两个，但是分布式架构中，要求 P 必须成立



# 2. 简单使用

## Nacos Discovery 服务注册中心

相应的依赖：

- spring-cloud-starter-alibaba-nacos-discovery：实现服务注册于发现
- spring-cloud-starter-alibaba-nacos-config：实现配置的动态变更

！！！注意：2021 及以后的版本，nacos 的 **负载均衡** 使用必须用 **Loadbalancer** 而非曾今内置的 Ribbon 

```xml
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
```

8. yml 配置：

```yml
server:
  port: 9001

spring:
  application:
    name: nacos-payment-provider # nacos 默认注册进去的服务名是这个
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #配置Nacos地址
```

启动类加注解：@EnableDiscoveryClient         //  但是好像也可以不加

负载均衡实现，看之前的 openFeign 的文章，里面有 **Loadbalancer** 实现负载均衡 和 切换负载均衡规则



## Nacos Config 服务配置中心

1. **依赖(版本可以自己测一下)：**

！！！必须有一个 **bootstrap** 依赖，作为nacos 依赖提前加载配置（ `application.yml ` 之前）

注册中心的依赖当然也要导一下了

```xml
   		<!-- 负载均衡 -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-loadbalancer</artifactId>
                <version>3.1.7</version>
            </dependency>
 	    <!--bootstrap-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bootstrap</artifactId>
        </dependency>
        <!--nacos-config-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
     <!--nacos-discovery-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
```

- Nacos同Consul一样，在项目初始化时，要保证先从配置中心进行配置拉取，
- 拉取配置之后，才能保证项目的正常启动，为了满足动态刷新和全局广播通知
- springboot中配置文件的加载是存在优先级顺序的，bootstrap优先级高于application



2. **yml 配置：**

- **bootstrap.yml ：**

```yml
# nacos配置
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置

# nacos端配置文件DataId的命名规则是：
# ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}
# 本案例的DataID是:nacos-config-client-dev.yaml
```

- **application.yml :**

```yml
server:
  port: 3377

spring:
  profiles:
    active: dev # 表示开发环境
       #active: prod # 表示生产环境
       #active: test # 表示测试环境
```



3. **主启动类：**@EnableDiscoveryClient       -----> 注解必须加嗷



4. **支持动态刷新**

一：**注解形式支持动态刷新（写个配置类）：** ----> Spring Cloud 原生注解 @RefreshScope 实现配置自动更新（原生的，当然是 Consul 那个）

抓到配置文件的配置

- 利用注解实现动态刷新配置：@RefreshScope //在控制器类加入注解使当前类下的配置支持Nacos的动态刷新功能。

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @auther zzyy
 * @create 2023-11-27 15:51
 */
@RestController
@RefreshScope //在控制器类加入@RefreshScope注解使当前类下的配置支持Nacos的动态刷新功能。
public class NacosConfigClientController
{
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    public String getConfigInfo() {
        return configInfo;
    }
}
```

二：**配置方式支持动态刷新（推荐）**

Nacos 好像是自带配置刷新的



5. **Nacos 中的匹配规则：**

- 设置 DataId 理论（Nacos中的dataid组成格式及SpringBoot配置文件中的匹配规则）：

最终公式：${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}

- **prefix** ：默认为 spring .application.name 值
- **spring.profiles.active**：即为当前环境对应的 profile，可以通过配置项 spring.profile.active 来配置
- **file-extension**：为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置
- 总结：

![](/img/0019-nacos配置中心指向配置说明.png )

- 创建配置的话，直接在控制台创建即可，记得遵循配置规则



6. **历史配置：**

- Nacos会记录配置文件的历史版本默认保留30天，此外还有一键回滚功能，回滚操作将会触发配置更新
- 控制台操作：直接看 **配置管理** 里面的历史版本，可以直接查询



## Nacos 数据模型之 Namespace-Group-DataId

> **DataId**：${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
>
> 当然你 Namespace 和 Group 设置的话，yml 必须去指定，不然用的就是默认的

- **多环境多项目管理：**
  - 直接就是看文档：https://nacos.io/zh-cn/docs/architecture.html
- **Namespace + Group + DataId 三者关系？为什么这么设计？** 
  - **Namespace**：命名空间（代码 yml 不设置的话默认是 `public` ），可以用于环境隔离
  - **Group**：分组（控制台创建配置的时候，可以选分组，不设置的为默认分组：`DEFAULT_GROUP` ）
  - **DataId**：个人感觉，就是配置的标识
  - **Server**：你注册的服务（微服务）呗

![](/img/0020-nacos数据模型.png )

> **三者说明：**
>
> ![](/img/0021-nacos.png )
>
> - 是什么？
>
> ---> 类似 Java 里面的 package 名和类名，最外层的 `Namespace` 是可以用于区分部署环境的，`Group` 和 `DataID` 逻辑上区分两个目标对象
>
> - 默认值：
>
> ---> **默认情况：Namespace=public 、 Group=DEFAULT_GROUP**
>
> Nacos 默认的命名空间是 public，**Namespace** 主要用来**实现隔离。**
>
> 比如，现在有三个环境：`开发dev`、`测试test`、`生产prod` ，可以创建三个 Namespace，不同的 Namespace 之间是隔离的。Group 默认是 DEFAULT_GROUP，Group 可以把不同的微服务划分到同一个分组里面去。
>
> - Server 就是微服务：
>
>   一个 Service 可以包含一个或多个 Cluster（集群），Nacos 默认 Cluster 是 DEFAULT，Cluster 是对指定微服务的一个虚拟划分。
>
> ![](/img/0022-nacos服务领域模型.png )



## 配置的优先级

> bootstrap.yml：用这玩意需要导依赖嗷（ spring-cloud-starter-bootstrap ）

- **本地配置文件：** bootstrap.yml 先加载 application.yml后加载，application 也不会覆盖 bootstrap，而 application.yml 里面的内容可以动态替换。

- **远程和本地配置文件的优先级：**远程项目应用名配置文件 > 远程扩展配置文件 > 远程共享配置文件 > 本地配置文件。

**Nacos 配置优先级：**

Spring Cloud Alibaba Nacos Config 提供了三种配置能力从 Nacos 拉取相关的配置

- 通过 spring.cloud.nacos.config.shared-configs[n].data-id 方式支持多个共享 dataId 配置
- 通过 spring.cloud.nacos.config.extension-configs[n]-data-id 方式支持多个扩展 dataId 配置
- 通过内部相关规则（应用名、应用名+Profile）自动生成相关的 dataId 配置

当三种方式同时使用时，它们的优先级关系：A < B < C，同理，若在这三种方式中配置了同样的参数时，则会使用 C 配置的参数值



## Nacos 配置文件加密：Jasypt

> 为什么要加密？
>
> - 在使用配置中心的时候，像账号、密码、密钥等信息，可能会经常的改动，如果你不加密，nacos 被攻破了，怎么办捏？
> - 当然也可以选择，将这些敏感的配置信息，放到环境变量里，直接从环境变量里去读，虽然也方便，但是管理的话，你还要管理环境变量里面的配置。

1. ### 当然了，Nacos 本身肯定是提供了加密的：基于 SPI 的插件机制实现

插件的依赖包：

```xml
<dependency>
  <groupId>com.alibaba.nacos</groupId>
  <artifactId>nacos-aes-encryption-plugin</artifactId>
  <version>${nacos-aes-encryption-plugin.version}</version>
</dependency>
```

之后如果想对配置加密，需要创建名称为 `cipher-[加密算法名称]-dataId`这种规则的配置文件，例如`cipher-aes-application-dev.yml` ，之后不管你在配置中写上什么内容，都会被加密；在程序中读出来的都是被加密过的，需要你**调用插件提供的解密方法解密**，或者**自定义加解密方法**。

但是，哥们这也太变态了吧，全加密？那岂不是我无论读取什么数据都要经过一次解密？有些不重要的信息，就算公开也无所谓，那我加密加什么哎，加了个寂寞？

并且，还要升级到 2.x 版本才能使用，都做成插件了，还要区分版本哎~

所以，当然有更好的肯定用更好的咯，对官方这种敷衍的行为做出强烈的谴责~



2. ### Jasypt 加密

Jasypt 其实是一个专门用于加解密的库，对于像 Nacos 配置文件、本地配置文件等配置信息的加解密就是一个顺手的事儿；加密就很简单了，太多开源的包了，你也可以自己写，so easzy啦

像我读取配置的时候，可以采用以下方式：

```java
@Value("${aestest.appKey}")
private String appKey;
```

现成的 jasypt 直接用

- 引入依赖：

```xml
<dependency>
  <groupId>com.github.ulisesbocchio</groupId>
  <artifactId>jasypt-spring-boot-starter</artifactId>
  <version>3.0.5</version>
</dependency>
```

- 在配置文件中进行相关配置：

```yml
jasypt:
  encryptor:
    password: hello
    algorithm: PBEWithMD5AndDES
```

`algorithm`是加密算法，官方默认的加密算法是 `PBEWITHHMACSHA512ANDAES_256`，但是如果你用的是 JDK1.8，还用不了这个算法，JDK9以上才支持，所以可以把这个算法改成`PBEWithMD5AndDES`。

`password`是加解密的时候用到的密码，这个配置是不建议放到Nacos的，可以放到环境变量中，这样一来，就只有这一个参数放到环境变量了。

- 生成加密字符串：

Jasypt 默认用 `Enc(内容)`这样的格式来表示这是加密的配置，当然你可以通过配置来修改前缀和后缀，比如改成 `JASYPT[内容]`这种形式，其中`内容`部分是加密后的。

加密串可以这样生成：

**引入加密类：**

```java
@Autowired
private StringEncryptor encryptor;
```

**生成加密内容：**

```java
@GetMapping("/encrypt")
public String encrypt(String content) {
  return "ENC(" + encryptor.encrypt(content) + ")";
}
```

- 最后将生成的加密串保存到 Nacos 或本地配置中，例如下面这样

```yml
aestest:
  appKey: ENC(GT2vTn1+SdeFu90xH/vgw3uYTNyV5PGp)
```

- 直接使用`@Value`注解获取就行，和不加密的用法一模一样

```java
@Value("${aestest.appKey}")
private String appKey;
```

> 原理
> 加密的原理没啥好说的，上面用的PBEWithMD5AndDES就是DES加密算法。
>
> 而@Value注解直接拿到解密后的值，其实是实现了BeanFactoryPostProcessor接口，相当于利用 Spring Boot 的加载机制做了一个filter，在filter中查找 @Value注解，并且内容是以 Jasypt 指定的前后缀的配置项（例如ENC()），将找到的内容进行解密，再赋值解密后的值。



# 3. Nacos Server 的运行启动

> #### **Nacos 有两种运行模式：**
>
> - standalone：单机运行模式（使用默认 Nacos 内置的一个嵌入式数据库 derby）
> - cluster：集群运行模式（使用 mysql）

## standalone （单节点）模式

此模式一般用于 demo 和 测试，不用改任何配置，直接敲以下命令执行

```powershell
sh bin/startup.sh -m standalone
```

Windows 的话就是

```cmd
cmd bin/startup.cmd -m standalone
```

默认控制台地址： http://cdh1:8848/nacos 

默认账号和密码为：nacos nacos



## cluster （集群）模式

测试环境，可以先用 standalone 模式撸起来，享受 coding 的快感，但是，生产环境可以使用 cluster 模式。

**cluster 模式需要依赖 MySQL，然后改两个配置文件：**

```
conf/cluster.conf
conf/application.properties
```

如下：

-  cluster.conf，填入要运行 Nacos Server 机器的 ip：
  - 192.168.100.155
  - 192.168.100.156

- 修改NACOS_PATH/conf/application.properties，加入 MySQL 配置

```properties
db.num=1
db.url.0=jdbc:mysql://localhost:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=1234
```

创建一个名为nacos_config的 database，将NACOS_PATH/conf/nacos-mysql.sql中的表结构导入刚才创建的库中，这几张表的用途就自己研究



## Nacos Server 的配置数据是存在哪里呢？

不做其它配置只有两个地方： 内存Derby（nacos内置的） / 本地数据库

 有两个坑：Nacos Server 的数据源是用 Derby 还是 MySQL 完全是由其运行模式决定的

- standalone 的话仅会使用 Derby，即使在 application.properties 里边配置 MySQL 也照样无视；

`单节点用于测试：单节点启动的持久化配置在内置Derby数据库 ----> NACOS_PATH/data，有个derby-data目录（这玩意我也看到 sql 文件了，好像也能用 mysql 搞？没试过，但是正常都是集群模式，单节点用于测试）`

- cluster 模式会自动使用 MySQL，这时候如果没有 MySQL 的配置，是会报错的（**注意：mysql8.0 以上好像是不支持的**）。

`集群模式：当然是用mysql数据库了，主要是统一各个节点的配置，NACOS_PATH/conf/nacos-mysql.sql中的表结构导入创建的库，研究研究，到底是配置了什么东西 `

（

**为什么要这样做？？？**

- 防止 Nacos 宕机或重启后数据丢失,Nacos 支持将数据统一持久化到数据库 

- Mysql(在不配置Nacos持久化到Mysql时,默认 Nacos 内置了一个嵌入式数据库derby,将一些数据保存到了内置的数据库上,多台 Nacos 就会出现多个内置数据库)。

）

> 下面是 集群模式持久化存储的 sql 文件中的表结构（AI解析的，可以了解一下）：
>
> - `config_info` 表
>
> #### 表描述
> 存储配置信息的基本数据。
>
> | 字段名             | 类型         | 是否为空 | 默认值   | 备注                            |
> | ------------------ | ------------ | -------- | -------- | ------------------------------- |
> | id                 | bigint(20)   | 否       | 自增     | 主键，唯一标识每条记录          |
> | data_id            | varchar(255) | 否       | 无       | 配置项的唯一标识符              |
> | group_id           | varchar(255) | 是       | NULL     | 配置项所属的组                  |
> | content            | longtext     | 否       | 无       | 配置项的内容                    |
> | md5                | varchar(32)  | 是       | NULL     | 配置内容的 MD5 值，用于快速校验 |
> | gmt_create         | datetime     | 否       | 当前时间 | 配置项的创建时间                |
> | gmt_modified       | datetime     | 否       | 当前时间 | 配置项的最后修改时间            |
> | src_user           | text         | 是       | NULL     | 创建或修改配置项的用户          |
> | src_ip             | varchar(50)  | 是       | NULL     | 创建或修改配置项的 IP 地址      |
> | app_name           | varchar(128) | 是       | NULL     | 应用名称                        |
> | tenant_id          | varchar(128) | 是       | 空字符串 | 租户字段，用于多租户支持        |
> | c_desc             | varchar(256) | 是       | NULL     | 配置描述                        |
> | c_use              | varchar(64)  | 是       | NULL     | 配置用途                        |
> | effect             | varchar(64)  | 是       | NULL     | 配置生效范围                    |
> | type               | varchar(64)  | 是       | NULL     | 配置类型                        |
> | c_schema           | text         | 是       | NULL     | 配置的 JSON Schema              |
> | encrypted_data_key | text         | 否       | 无       | 加密数据的密钥                  |
>
> - 2. `config_info_aggr` 表
>
> #### 表描述
> 存储聚合配置信息。
>
> | 字段名       | 类型         | 是否为空 | 默认值   | 备注                     |
> | ------------ | ------------ | -------- | -------- | ------------------------ |
> | id           | bigint(20)   | 否       | 自增     | 主键，唯一标识每条记录   |
> | data_id      | varchar(255) | 否       | 无       | 配置项的唯一标识符       |
> | group_id     | varchar(255) | 否       | 无       | 配置项所属的组           |
> | datum_id     | varchar(255) | 否       | 无       | 聚合配置项的唯一标识符   |
> | content      | longtext     | 否       | 无       | 聚合配置项的内容         |
> | gmt_modified | datetime     | 否       | 无       | 聚合配置项的最后修改时间 |
> | app_name     | varchar(128) | 是       | NULL     | 应用名称                 |
> | tenant_id    | varchar(128) | 是       | 空字符串 | 租户字段，用于多租户支持 |
>
> - 3. `config_info_beta` 表
>
> 存储 Beta 版本的配置信息。
>
> | 字段名             | 类型          | 是否为空 | 默认值   | 备注                            |
> | ------------------ | ------------- | -------- | -------- | ------------------------------- |
> | id                 | bigint(20)    | 否       | 自增     | 主键，唯一标识每条记录          |
> | data_id            | varchar(255)  | 否       | 无       | 配置项的唯一标识符              |
> | group_id           | varchar(128)  | 否       | 无       | 配置项所属的组                  |
> | app_name           | varchar(128)  | 是       | NULL     | 应用名称                        |
> | content            | longtext      | 否       | 无       | 配置项的内容                    |
> | beta_ips           | varchar(1024) | 是       | NULL     | Beta 版本的 IP 地址列表         |
> | md5                | varchar(32)   | 是       | NULL     | 配置内容的 MD5 值，用于快速校验 |
> | gmt_create         | datetime      | 否       | 当前时间 | 配置项的创建时间                |
> | gmt_modified       | datetime      | 否       | 当前时间 | 配置项的最后修改时间            |
> | src_user           | text          | 是       | NULL     | 创建或修改配置项的用户          |
> | src_ip             | varchar(50)   | 是       | NULL     | 创建或修改配置项的 IP 地址      |
> | tenant_id          | varchar(128)  | 是       | 空字符串 | 租户字段，用于多租户支持        |
> | encrypted_data_key | text          | 否       | 无       | 加密数据的密钥                  |
>
> - 4. `config_info_tag` 表
>
> 存储带有标签的配置信息。
>
> | 字段名       | 类型         | 是否为空 | 默认值   | 备注                            |
> | ------------ | ------------ | -------- | -------- | ------------------------------- |
> | id           | bigint(20)   | 否       | 自增     | 主键，唯一标识每条记录          |
> | data_id      | varchar(255) | 否       | 无       | 配置项的唯一标识符              |
> | group_id     | varchar(128) | 否       | 无       | 配置项所属的组                  |
> | tenant_id    | varchar(128) | 是       | 空字符串 | 租户字段，用于多租户支持        |
> | tag_id       | varchar(128) | 否       | 无       | 标签的唯一标识符                |
> | app_name     | varchar(128) | 是       | NULL     | 应用名称                        |
> | content      | longtext     | 否       | 无       | 配置项的内容                    |
> | md5          | varchar(32)  | 是       | NULL     | 配置内容的 MD5 值，用于快速校验 |
> | gmt_create   | datetime     | 否       | 当前时间 | 配置项的创建时间                |
> | gmt_modified | datetime     | 否       | 当前时间 | 配置项的最后修改时间            |
> | src_user     | text         | 是       | NULL     | 创建或修改配置项的用户          |
> | src_ip       | varchar(50)  | 是       | NULL     | 创建或修改配置项的 IP 地址      |
>
> - 5. `config_tags_relation` 表
>
> 存储配置项和标签的关系。
>
> | 字段名    | 类型         | 是否为空 | 默认值   | 备注                     |
> | --------- | ------------ | -------- | -------- | ------------------------ |
> | id        | bigint(20)   | 否       | 无       | 配置项的唯一标识符       |
> | tag_name  | varchar(128) | 否       | 无       | 标签名称                 |
> | tag_type  | varchar(64)  | 是       | NULL     | 标签类型                 |
> | data_id   | varchar(255) | 否       | 无       | 配置项的唯一标识符       |
> | group_id  | varchar(128) | 否       | 无       | 配置项所属的组           |
> | tenant_id | varchar(128) | 是       | 空字符串 | 租户字段，用于多租户支持 |
> | nid       | bigint(20)   | 否       | 自增     | 主键，唯一标识每条记录   |
>
> - 6. `group_capacity` 表
>
> 存储各 Group 的容量信息。
>
> | 字段名            | 类型                | 是否为空 | 默认值   | 备注                                                      |
> | ----------------- | ------------------- | -------- | -------- | --------------------------------------------------------- |
> | id                | bigint(20) unsigned | 否       | 自增     | 主键，唯一标识每条记录                                    |
> | group_id          | varchar(128)        | 否       | 空字符串 | Group ID，空字符表示整个集群                              |
> | quota             | int(10) unsigned    | 否       | 0        | 配额，0表示使用默认值                                     |
> | usage             | int(10) unsigned    | 否       | 0        | 使用量                                                    |
> | max_size          | int(10) unsigned    | 否       | 0        | 单个配置大小上限，单位为字节，0表示使用默认值             |
> | max_aggr_count    | int(10) unsigned    | 否       | 0        | 聚合子配置最大个数，0表示使用默认值                       |
> | max_aggr_size     | int(10) unsigned    | 否       | 0        | 单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值 |
> | max_history_count | int(10) unsigned    | 否       | 0        | 最大变更历史数量                                          |
> | gmt_create        | datetime            | 否       | 当前时间 | 创建时间                                                  |
> | gmt_modified      | datetime            | 否       | 当前时间 | 修改时间                                                  |
>
> - 7. `his_config_info` 表
>
> 存储配置信息的历史记录。
>
> | 字段名             | 类型                | 是否为空 | 默认值   | 备注                            |
> | ------------------ | ------------------- | -------- | -------- | ------------------------------- |
> | id                 | bigint(20) unsigned | 否       | 无       | 配置项的唯一标识符              |
> | nid                | bigint(20) unsigned | 否       | 自增     | 主键，唯一标识每条记录          |
> | data_id            | varchar(255)        | 否       | 无       | 配置项的唯一标识符              |
> | group_id           | varchar(128)        | 否       | 无       | 配置项所属的组                  |
> | app_name           | varchar(128)        | 是       | NULL     | 应用名称                        |
> | content            | longtext            | 否       | 无       | 配置项的内容                    |
> | md5                | varchar(32)         | 是       | NULL     | 配置内容的 MD5 值，用于快速校验 |
> | gmt_create         | datetime            | 否       | 当前时间 | 配置项的创建时间                |
> | gmt_modified       | datetime            | 否       | 当前时间 | 配置项的最后修改时间            |
> | src_user           | text                | 是       | NULL     | 创建或修改配置项的用户          |
> | src_ip             | varchar(50)         | 是       | NULL     | 创建或修改配置项的 IP 地址      |
> | op_type            | char(10)            | 是       | NULL     | 操作类型                        |
> | tenant_id          | varchar(128)        | 是       | 空字符串 | 租户字段，用于多租户支持        |
> | encrypted_data_key | text                | 否       | 无       | 加密数据的密钥                  |
>
> - 8. `tenant_capacity` 表
>
> 存储租户的容量信息。
>
> | 字段名            | 类型                | 是否为空 | 默认值   | 备注                                                      |
> | ----------------- | ------------------- | -------- | -------- | --------------------------------------------------------- |
> | id                | bigint(20) unsigned | 否       | 自增     | 主键，唯一标识每条记录                                    |
> | tenant_id         | varchar(128)        | 否       | 空字符串 | 租户 ID                                                   |
> | quota             | int(10) unsigned    | 否       | 0        | 配额，0表示使用默认值                                     |
> | usage             | int(10) unsigned    | 否       | 0        | 使用量                                                    |
> | max_size          | int(10) unsigned    | 否       | 0        | 单个配置大小上限，单位为字节，0表示使用默认值             |
> | max_aggr_count    | int(10) unsigned    | 否       | 0        | 聚合子配置最大个数                                        |
> | max_aggr_size     | int(10) unsigned    | 否       | 0        | 单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值 |
> | max_history_count | int(10) unsigned    | 否       | 0        | 最大变更历史数量                                          |
> | gmt_create        | datetime            | 否       | 当前时间 | 创建时间                                                  |
> | gmt_modified      | datetime            | 否       | 当前时间 | 修改时间                                                  |
>
> - 9. `tenant_info` 表
>
> 存储租户信息。
>
> | 字段名        | 类型         | 是否为空 | 默认值   | 备注                   |
> | ------------- | ------------ | -------- | -------- | ---------------------- |
> | id            | bigint(20)   | 否       | 自增     | 主键，唯一标识每条记录 |
> | kp            | varchar(128) | 否       | 无       | 租户的唯一标识符       |
> | tenant_id     | varchar(128) | 是       | 空字符串 | 租户 ID                |
> | tenant_name   | varchar(128) | 是       | 空字符串 | 租户名称               |
> | tenant_desc   | varchar(256) | 是       | NULL     | 租户描述               |
> | create_source | varchar(32)  | 是       | NULL     | 创建来源               |
> | gmt_create    | bigint(20)   | 否       | 无       | 创建时间，时间戳格式   |
> | gmt_modified  | bigint(20)   | 否       | 无       | 修改时间，时间戳格式   |
>
> - 10. `users` 表
>
> 存储用户信息。
>
> | 字段名   | 类型         | 是否为空 | 默认值 | 备注         |
> | -------- | ------------ | -------- | ------ | ------------ |
> | username | varchar(50)  | 否       | 无     | 用户名，主键 |
> | password | varchar(500) | 否       | 无     | 密码         |
> | enabled  | boolean      | 否       | 无     | 用户是否启用 |
>
> - 11. `roles` 表
>
> 存储用户角色信息。
>
> | 字段名   | 类型        | 是否为空 | 默认值 | 备注   |
> | -------- | ----------- | -------- | ------ | ------ |
> | username | varchar(50) | 否       | 无     | 用户名 |
> | role     | varchar(50) | 否       | 无     | 角色   |
>
> - 12. `permissions` 表
>
> 存储角色权限信息。
>
> | 字段名   | 类型         | 是否为空 | 默认值 | 备注                      |
> | -------- | ------------ | -------- | ------ | ------------------------- |
> | role     | varchar(50)  | 否       | 无     | 角色                      |
> | resource | varchar(255) | 否       | 无     | 资源路径                  |
> | action   | varchar(8)   | 否       | 无     | 动作，如 `GET`, `POST` 等 |
>
> 
>
> 
>
> 

