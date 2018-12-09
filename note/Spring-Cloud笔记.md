# SpringCloud笔记



>引言

最近一段时间，微服务大行其道，Spring Cloud是其中的佼佼者，说起微服务，大家会想到下面问题？

+ 什么是微服务？
+ 微服务之间是如何独立通讯的？
+ springCloud和Dubbo有哪些区别？
+ SpringBoot和SpringCloud，请你谈谈对他们的理解
+ 什么是服务熔断？什么是服务降级
+ 微服务的优缺点分别是什么？说下你在项目开发中碰到的坑
+ 你所知道的微服务技术栈有哪些？请列举一二
+ eureka和zookeeper都可以提供服务注册与发现的功能，请说说两个的区别？

<!--more-->

本文使用的是Spring Boot版本2.0.3.RELEASE,Spring Cloud版本为Finchley.RELEASE。

Finchley版本的官方文档如下：http://cloud.spring.io/spring-cloud-static/Finchley.RELEASE/single/spring-cloud.html
## 微服务简介

### 微服务概述

关于微服务，不得不说起业界大牛马丁.福勒，这里给出他的论文：

+ https://martinfowler.com/articles/microservices.html（英文版）
+ http://www.cnblogs.com/liuning8023/p/4493156.html（中文版）

从论文上看，微服务架构是⼀种架构模式，它提倡将单⼀应⽤程序划分成⼀组⼩的服务，服务之间互相协调、互相配合，为⽤户提供最终价值。每个服务运⾏在其独⽴的进程中，服务与服务间采⽤轻量级的通信机制互相协作（通常是基于HTTP协议的RESTful API）。每个服务都围绕着具体业务进⾏构建，并且能够被独⽴的部署到⽣产环境、类⽣产环境等。另外，应当尽量避免统⼀的、集中式的服务管理机制，对具体的⼀个服务⽽⾔，应根据业务上下⽂，选择合适的语⾔、⼯具对其进⾏构建。

总的来说：

微服务化的核心就是将传统的一站式应用，根据业务拆分成一个一个的服务，彻底地去耦合,每一个微服务提供单个业务功能的服务，一个服务做一件事，从技术角度看就是一种小而独立的处理过程，类似进程概念，能够自行单独启动或销毁，拥有自己独立的数据库。

微服务强调的是服务的大小，它关注的是某一个点，是具体解决某一个问题/提供落地对应服务的一个服务应用,狭意的看,可以看作Eclipse里面的一个个微服务工程/或者Module。



### 微服务优缺点

优点：

+ 每个服务足够内聚，足够小，代码容易理解这样能聚焦一个指定的业务功能或业务需求
+ 发简单、开发效率提高，一个服务可能就是专一的只干一件事。
+ 微服务能够被小团队单独开发，这个小团队是2到5人的开发人员组成。
+ 微服务是松耦合的，是有功能意义的服务，无论是在开发阶段或部署阶段都是独立的。
+ 微服务能使用不同的语言开发。
+ 易于和第三方集成，微服务允许容易且灵活的方式集成自动部署，通过持续集成工具，如Jenkins, Hudson, bamboo 。
+ 微服务易于被一个开发人员理解，修改和维护，这样小团队能够更关注自己的工作成果。无需通过合作才能体现价值。
+ 微服务允许你利用融合最新技术。
+ 微服务只是业务逻辑的代码，不会和HTML,CSS 或其他界面组件混合。
+ 每个微服务都有自己的存储能力，可以有自己的数据库。也可以有统一数据库。



缺点：

+ 开发人员要处理分布式系统的复杂性
+ 多服务运维难度，随着服务的增加，运维的压力也在增大
+ 系统部署依赖
+ 服务间通信成本
+ 数据一致性
+ 系统集成测试
+ 性能监控



### 微服务技术栈

| 微服务条目                             | 落地技术                                                     |
| -------------------------------------- | ------------------------------------------------------------ |
| 服务开发                               | Springboot、Spring、SpringMVC                                |
| 服务配置与管理                         | Netflix公司的Archaius、阿里的Diamond等                       |
| 服务注册与发现                         | Eureka、Consul、Zookeeper等                                  |
| 服务调用                               | Rest、RPC、gRPC                                              |
| 服务熔断器                             | Hystrix、Envoy等                                             |
| 负载均衡                               | Ribbon、Nginx等                                              |
| 服务接口调用(客户端调用服务的简化工具) | Feign等                                                      |
| 消息队列                               | Kafka、RabbitMQ、ActiveMQ等                                  |
| 服务配置中心管理                       | SpringCloudConfig、Chef等                                    |
| 服务路由（API网关）                    | Zuul等                                                       |
| 服务监控                               | Zabbix、Nagios、Metrics、Spectator等                         |
| 全链路追踪                             | Zipkin，Brave、Dapper等                                      |
| 服务部署                               | Docker、OpenStack、Kubernetes等                              |
| 数据流操作开发包                       | SpringCloud Stream（封装与Redis,Rabbit、Kafka等发送接收消息） |
| 事件消息总线                           | Spring Cloud Bus                                             |





### 为什么选择SpringCloud作为微服务架构

**1.选型依据？**

整体解决方案和框架成熟度
社区热度
可维护性
学习曲线



**2.当前各大IT公司用的微服务架构有哪些?**
阿里Dubbo/HSF
京东JSF
新浪微博Motan
当当网DubboX
......



**3.各微服务框架对比**

| 功能点/服务框架            | Netflix/SpringCloud                                          | Motan                                                        | gRPC                      | Thrift   | Dubbo/DubboX     |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | :------------------------ | -------- | ---------------- |
| 功能定位                   | 完整的微服务框架                                             | RPC框架，但整合了ZK或Consul，实现集群环境的基本的服务注册/发现 | RPC框架                   | RPC框架  | 服务框架         |
| 支持REST                   | 是 Eibbon支持多种可插拔的序列化选择                          | 否                                                           | 否                        | 否       | 否               |
| 支持RPC                    | 否                                                           | 是                                                           | 是                        | 是       | 是               |
| 支持多语言                 | 是                                                           | 否                                                           | 是                        | 是       | 否               |
| 服务注册/发现              | 是，Eureka服务注册表，karyon服务端框架支持服务自注册和健康检查 | 是（zookerper/consul）                                       | 否                        | 否       | 是               |
| 负载均衡                   | 是（服务端zuul+客户端Ribbon）zuul-服务，动态路由   云端负载均衡 | 是                                                           | 否                        | 否       | 是（客户端）     |
| Eureka（针对中间层服务器） |                                                              |                                                              |                           |          |                  |
| 配置服务                   | Netflix Archaius   Spring Cloud Config Server 集中配置       | 是（zookeeper提供）                                          | 否                        | 否       | 否               |
| 服务调用监控               | 是（zuul）   zuul提供边缘服务，API网关                       | 否                                                           | 否                        | 否       | 否               |
| 高可用/容错                | 是（服务端Hystrix+客户端Ribbon）                             | 是（客户端）                                                 | 否                        | 否       | 是（客户端）     |
| 典型应用案例               | Netflix                                                      | Sina                                                         | Google                    | Facebook |                  |
| 社区活跃程度               | 高                                                           | 一般                                                         | 高                        | 一般     | 已经不维护了     |
| 学习难度                   | 中等                                                         | 低                                                           | 高                        | 高       | 低               |
| 文档丰富度                 | 高                                                           | 一般                                                         | 一般                      | 一般     | 高               |
| 其他                       | Spring Cloud Bus为我们的应用程序带来了更多管理端点           | 支持降级                                                     | Netflix内部在开发集成gRPC | IDL定义  | 实践的公司比较多 |



## SpringCloud入门



### SpringCloud是什么？

**1.官网说明**

![SpringCloud笔记](http://img.bcoder.top/2018.12.08/1.png)



SpringCloud，基于SpringBoot提供了一套微服务解决方案，包括服务注册与发现，配置中心，全链路监控，服务网关，负载均衡，熔断器等组件，除了基于NetFlix的开源组件做高度抽象封装之外，还有一些选型中立的开源组件。

SpringCloud利用SpringBoot的开发便利性巧妙地简化了分布式系统基础设施的开发，SpringCloud为开发人员提供了快速构建分布式系统的一些工具，**包括配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选**、分布式会话等等,它们都可以用SpringBoot的开发风格做到一键启动和部署。

SpringBoot并没有重复制造轮子，它只是将目前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过SpringBoot风格进行再封装屏蔽掉了复杂的配置和实现原理，**最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包**



SpringCloud=分布式微服务架构下的一站式解决方案，是各个微服务架构落地技术的集合体，俗称微服务全家桶



**2.SpringCloud和SpringBoot是什么关系？**

SpringBoot专注于快速方便的开发单个个体微服务。

SpringCloud是关注全局的微服务协调整理治理框架，它将SpringBoot开发的一个个单体微服务整合并管理起来，
为各个微服务之间提供，配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等集成服务

**SpringBoot可以离开SpringCloud独立使用开发项目，但是SpringCloud离不开SpringBoot，属于依赖的关系.**

**SpringBoot专注于快速、方便的开发单个微服务个体，SpringCloud关注全局的服务治理框架。**



**3.Dubbo VS SpringCloud**

| Dubbo        | Spring Cloud |                              |
| ------------ | ------------ | ---------------------------- |
| 服务注册中心 | Zookeeper    | Spring Cloud Netflix Eureka  |
| 服务调用方式 | RPC          | REST API                     |
| 服务网关     | 无           | Spring Cloud Netflix Zuul    |
| 断路器       | 不完善       | Spring Cloud Netflix Hystrix |
| 分布式配置   | 无           | Spring Cloud Config          |
| 服务跟踪     | 无           | Spring Cloud Sleuth          |
| 消息总线     | 无           | Spring Cloud Bus             |
| 数据流       | 无           | Spring Cloud Stream          |
| 批量任务     | 无           | Spring Cloud Task            |
| ……           | ……           | ……                           |

**最大区别：SpringCloud抛弃了Dubbo的RPC通信，采用的是基于HTTP的REST方式。**
严格来说，这两种方式各有优劣。虽然从一定程度上来说，后者牺牲了服务调用的性能，但也避免了上面提到的原生RPC带来的问题。而且REST相比RPC更为灵活，服务提供方和调用方的依赖只依靠一纸契约，不存在代码级别的强依赖，这在强调快速演化的微服务环境下，显得更加合适。



### SpringCloud能干嘛?

+ Distributed/versioned configuration(分布式/版本控制配置)
+ Service registration and discovery(服务注册与发现)
+ Routing(路由)
+ Service-to-service calls(服务到服务的调用)
+ Load balancing(负载均衡配置)
+ Circuit Breakers(断路器)
+ Distributed messaging(分布式消息管理)



**对应的组件？**

+ 服务的注册与发现（Eureka）
+ 服务消费者（rest+Ribbon）
+ 服务消费者（Feign）
+ 断路器（Hystrix）
+ 断路器监控(Hystrix Dashboard)
+ 路由网关(Zuul)
+ 分布式配置中心(Spring Cloud Config)
+ 消息总线(Spring Cloud Bus)
+ 服务链路追踪(Spring Cloud Sleuth)



### SpringCloud怎么玩?



**1.官网**

http://projects.spring.io/spring-cloud/



**2.参考书**

+ https://springcloud.cc/spring-cloud-netflix.html
+ springcloud中国社区：http://springcloud.cn/
+ springcloud中文网：https://springcloud.cc/


3.开源项目（更新实时）

+ https://github.com/ityouknow/spring-cloud-examples
+ https://gitee.com/liuge1988/spring-cloud-demo
+ https://git.oschina.net/forezp/SpringCloudLearning



## Eureka服务注册与发现

### Eureka介绍



Eureka是Netflix的一个子模块，也是核心模块之一。Eureka是一个基于REST的服务，用于定位服务，以实现云端中间层服务发现和故障转移。服务注册与发现对于微服务架构来说是非常重要的，有了服务发现与注册，**只需要使用服务的标识符**，就可以访问到服务，而不需要修改服务调用的配置文件了。**功能类似于dubbo的注册中心，比如Zookeeper。**

Netflix在设计Eureka时遵守的就是AP原则(CAP原则又称CAP定理，指的是在一个分布式系统中，Consistency（一致性）、 Availability（可用性）、Partition tolerance（分区容错性），三者不可兼得)



### Eureka原理

1.Eureka的基本架构

Spring Cloud 封装了 Netflix 公司开发的 Eureka 模块来**实现服务注册和发现(请对比Zookeeper)。**

Eureka 采用了 C-S 的设计架构。Eureka Server 作为服务注册功能的服务器，它是服务注册中心。

而系统中的其他微服务，使用 Eureka 的客户端连接到 Eureka Server并维持心跳连接。这样系统的维护人员就可以通过 Eureka Server 来监控系统中各个微服务是否正常运行。SpringCloud 的一些其他模块（比如Zuul）就可以通过 Eureka Server 来发现系统中的其他微服务，并执行相关的逻辑。



**Eureka 架构：**

![SpringCloud笔记](http://img.bcoder.top/2018.12.08/2.png)

**Dubbo架构：**

![SpringCloud笔记](http://img.bcoder.top/2018.12.08/3.png)

**Eureka包含两个组件：Eureka Server和Eureka Client**
Eureka Server提供服务注册服务,各个节点启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观的看到

EurekaClient是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒）



### Eureka三大角色

+ Eureka Server 提供服务注册和发现
+ Service Provider服务提供方将自身服务注册到Eureka，从而使服务消费方能够找到
+ Service Consumer服务消费方从Eureka获取注册服务列表，从而能够消费服务



### Eureka实战

#### 服务注册中心（server）创建

**1.首先创建一个maven主工程。**

首先创建一个主Maven工程，在其pom文件引入依赖，spring Boot版本为2.0.3.RELEASE，Spring Cloud版本为Finchley.RELEASE。这个pom文件作为父pom文件，起到依赖版本控制的作用，其他module工程继承该pom。这一系列文章全部采用这种模式，其他文章的pom跟这个pom一样。再次说明一下，以后不再重复引入。代码如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>top.bcoder</groupId>
    <artifactId>sc-f-chapter1</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>

    <name>sc-f-chapter1</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
        <relativePath/>
    </parent>

    <modules>
        <module>eureka-server</module>
        <module>service-hi</module>
    </modules>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

**2 然后创建2个model工程**

一个model工程作为服务注册中心，即Eureka Server,另一个作为Eureka Client。

下面以创建server为例子，详细说明创建过程：

右键工程->创建model-> 选择spring initialir 如下图：

![SpringCloud笔记](http://img.bcoder.top/2018.12.08/4.png)

下一步->选择cloud discovery->eureka server ,然后一直下一步就行了。

![SpringCloud笔记](http://img.bcoder.top/2018.12.08/5.png)

创建完后的工程，其pom.xml继承了父pom文件，并引入spring-cloud-starter-netflix-eureka-server的依赖，代码如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>top.bcoder</groupId>
    <artifactId>eureka-server</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>eureka-server</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>top.bcoder</groupId>
        <artifactId>sc-f-chapter1</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>

</project>
```

**3.启动一个服务注册中心**

只需要一个注解@EnableEurekaServer，这个注解需要在springboot工程的启动application类上加：

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run( EurekaServerApplication.class, args );
    }
}
```

**4.appication.yml配置**

eureka是一个高可用的组件，它没有后端缓存，每一个实例注册之后需要向注册中心发送心跳（因此可以在内存中完成），在默认情况下erureka server也是一个eureka client ,必须要指定一个 server。eureka server的配置文件appication.yml：

```yml
erver:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

spring:
  application:
    name: eurka-server
```

通过eureka.client.registerWithEureka：false和fetchRegistry：false来表明自己是一个eureka server.

**5.eureka server界面**

eureka server 是有界面的，启动工程,打开浏览器访问： http://localhost:8761 ,界面如下：

![SpringCloud笔记](http://img.bcoder.top/2018.12.08/6.png)



#### 服务注册中心（client）创建

当client向server注册时，它会提供一些元数据，例如主机和端口，URL，主页等。Eureka server 从每个client实例接收心跳消息。 如果心跳超时，则通常将该实例从注册server中删除。

创建过程同server类似,创建完pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>top.bcoder</groupId>
    <artifactId>service-hi</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>service-hi</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>top.bcoder</groupId>
        <artifactId>sc-f-chapter1</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


</project>
```

通过注解@EnableEurekaClient 表明自己是一个eurekaclient.

```java
@SpringBootApplication
@EnableEurekaClient
@RestController
public class ServiceHiApplication {

    public static void main(String[] args) {
        SpringApplication.run( ServiceHiApplication.class, args );
    }

    @Value("${server.port}")
    String port;

    @RequestMapping("/hi")
    public String home(@RequestParam(value = "name", defaultValue = "forezp") String name) {
        return "hi " + name + " ,i am from port:" + port;
    }

}
```

仅仅@EnableEurekaClient是不够的，还需要在配置文件中注明自己的服务注册中心的地址，application.yml配置文件如下：

```yml
server:
  port: 8762

spring:
  application:
    name: service-hi

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

需要指明spring.application.name,这个很重要，这在以后的服务与服务之间相互调用一般都是根据这个name 。 启动工程，打开http://localhost:8761 ，即eureka server 的网址：

![SpringCloud笔记](http://img.bcoder.top/2018.12.08/7.png)



你会发现一个服务已经注册在服务中了，服务名为SERVICE-HI ,端口为7862

这时打开 http://localhost:8762/hi?name=forezp ，你会在浏览器上看到 :

> hi forezp,i am from port:8762



#### eureka server高可用

Eureka通过运行多个实例，使其更具有高可用性。事实上，这是它默认的熟性，你需要做的就是给对等的实例一个合法的关联serviceurl。

在eureka-server工程中resources文件夹下，创建配置文件application-peer1.yml:

```yaml
server:
  port: 8761

spring:
  profiles: peer1
eureka:
  instance:
    hostname: peer1
  client:
    serviceUrl:
      defaultZone: http://peer2:8769/eureka/
```

并且创建另外一个配置文件application-peer2.yml：

```yaml
server:
  port: 8769

spring:
  profiles: peer2
eureka:
  instance:
    hostname: peer2
  client:
    serviceUrl:
      defaultZone: http://peer1:8761/eureka/
```

这时eureka-server就已经改造完毕.



按照官方文档的指示，需要改变etc/hosts，linux系统通过vim /etc/hosts ,加上：

```
127.0.0.1 peer1
127.0.0.1 peer2
```

这时需要改造下service-hi:

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://peer1:8761/eureka/
server:
  port: 8762
spring:
  application:
    name: service-hi
```

启动eureka-server：

> java -jar eureka-server-0.0.1-SNAPSHOT.jar - -spring.profiles.active=peer1
>
> java -jar eureka-server-0.0.1-SNAPSHOT.jar - -spring.profiles.active=peer2

>

启动service-hi:

> java -jar service-hi-0.0.1-SNAPSHOT.jar

访问：localhost:8761,如图：

![SpringCloud笔记](http://img.bcoder.top/2018.12.08/8.png)

你会发现注册了service-hi，并且有个peer2节点，同理访问localhost:8769你会发现有个peer1节点。

client只向8761注册，但是你打开8769，你也会发现，8769也有 client的注册信息。

个人感受：这是通过看官方文档的写的demo ，但是需要手动改host是不是不符合Spring Cloud 的高上大？

**eureka.instance.preferIpAddress=true**是通过设置ip让eureka让其他服务注册它。也许能通过去改变去通过改变host的方式。

此时的架构图：

![SpringCloud笔记](http://img.bcoder.top/2018.12.08/9.png)

Eureka-eserver peer1 8761,Eureka-eserver peer2 8769相互感应，当有服务注册时，两个Eureka-eserver是对等的，它们都存有相同的信息，这就是通过服务器的冗余来增加可靠性，当有一台服务器宕机了，服务并不会终止，因为另一台服务存有相同的数据。



### Eureka VS Zookeeper

作为服务注册中心，Eureka比Zookeeper好在哪里?
著名的CAP理论指出，一个分布式系统不可能同时满足C(一致性)、A(可用性)和P(分区容错性)。由于分区容错性P在是分布式系统中必须要保证的，因此我们只能在A和C之间进行权衡。
因此,Zookeeper保证的是CP,Eureka则是AP。



**Zookeeper保证CP**
当向注册中心查询服务列表时，我们可以容忍注册中心返回的是几分钟以前的注册信息，但不能接受服务直接down掉不可用。也就是说，服务注册功能对可用性的要求要高于一致性。但是zk会出现这样一种情况，当master节点因为网络故障与其他节点失去联系时，剩余节点会重新进行leader选举。问题在于，选举leader的时间太长，30 ~ 120s, 且选举期间整个zk集群都是不可用的，这就导致在选举期间注册服务瘫痪。在云部署的环境下，因网络问题使得zk集群失去master节点是较大概率会发生的事，虽然服务能够最终恢复，但是漫长的选举时间导致的注册长期不可用是不能容忍的。



**Eureka保证AP**
Eureka看明白了这一点，因此在设计时就优先保证可用性。Eureka各个节点都是平等的，几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。而Eureka的客户端在向某个Eureka注册或时如果发现连接失败，则会自动切换至其它节点，只要有一台Eureka还在，就能保证注册服务可用(保证可用性)，只不过查到的信息可能不是最新的(不保证强一致性)。除此之外，Eureka还有一种自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况： 

1. Eureka不再从注册列表中移除因为长时间没收到心跳而应该过期的服务 
2. Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上(即保证当前节点依然可用) 
3. 当网络稳定时，当前实例新的注册信息会被同步到其它节点中

**因此， Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像zookeeper那样使整个注册服务瘫痪。**



## Ribbon负载均衡

### Ribbon概述

**1.Ribbon是什么?**

Spring Cloud Ribbon是基于Netflix Ribbon实现的一套**客户端负载均衡**的工具。

简单的说，Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法，将Netflix的中间层服务连接在一起。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出Load Balancer（简称LB）后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们也很容易使用Ribbon实现自定义的负载均衡算法。



**2.Ribbon能干吗**

LB，即负载均衡(Load Balance)，在微服务或分布式集群中经常用的一种应用。
负载均衡简单的说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA。
常见的负载均衡有软件Nginx，LVS，硬件 F5等。
相应的在中间件，例如：dubbo和SpringCloud中均给我们提供了负载均衡，SpringCloud的负载均衡算法可以自定义

LB分为两种：

+ **集中式LB**：即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件，如F5, 也可以是软件，如nginx), 由该设施负责把访问请求通过某种策略转发至服务的提供方；

+ **进程内LB：**将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。**Ribbon就属于进程内LB**，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址。



**3.官网资料:**https://github.com/Netflix/ribbon/wiki/Getting-Started



### Ribbon实战

基于上一节的工程，启动eureka-server 工程；启动service-hi工程，它的端口为8762；将service-hi的配置文件的端口改为8763,并启动，这时你会发现：service-hi在eureka-server注册了2个实例，这就相当于一个小的集群。



如何在idea下启动多个实例，请参照这篇文章： https://blog.csdn.net/forezp/article/details/76408139

访问localhost:8761如图所示： 如何一个工程启动多个实例，请看这篇文章:https://blog.csdn.net/forezp/article/details/76408139

![SpringCloud笔记](http://img.bcoder.top/2018.12.08/10.png)



重新新建一个spring-boot工程，取名为：service-ribbon; 在它的pom.xml继承了父pom文件，并引入了以下依赖：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>top.bcoder</groupId>
    <artifactId>service-ribbon</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>service-ribbon</name>
    <description>Demo project for Spring Boot</description>


    <parent>
        <groupId>top.bcoder</groupId>
        <artifactId>sc-f-chapter2</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
        </dependency>
    </dependencies>

 

</project>
```

在工程的配置文件指定服务的注册中心地址为http://localhost:8761/eureka/，程序名称为 service-ribbon，程序端口为8764。配置文件application.yml如下：

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 8764
spring:
  application:
    name: service-ribbon
```

在工程的启动类中,通过@EnableDiscoveryClient向服务中心注册；并且向程序的ioc注入一个bean: restTemplate;并通过@LoadBalanced注解表明这个restRemplate开启负载均衡的功能。

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
public class ServiceRibbonApplication {

    public static void main(String[] args) {
        SpringApplication.run( ServiceRibbonApplication.class, args );
    }

    @Bean
    @LoadBalanced
    RestTemplate restTemplate() {
        return new RestTemplate();
    }

}
```

写一个测试类HelloService，通过之前注入ioc容器的restTemplate来消费service-hi服务的“/hi”接口，在这里我们直接用的程序名替代了具体的url地址，在ribbon中它会根据服务名来选择具体的服务实例，根据服务实例在请求的时候会用具体的url替换掉服务名，代码如下：

```java
@Service
public class HelloService {

    @Autowired
    RestTemplate restTemplate;

    public String hiService(String name) {
        return restTemplate.getForObject("http://SERVICE-HI/hi?name="+name,String.class);
    }


}
```

写一个controller，在controller中用调用HelloService 的方法，代码如下：

```java
@RestController
public class HelloControler {

    @Autowired
    HelloService helloService;

    @GetMapping(value = "/hi")
    public String hi(@RequestParam String name) {
        return helloService.hiService( name );
    }
}
```

在浏览器上多次访问http://localhost:8764/hi?name=forezp，浏览器交替显示：

> hi forezp,i am from port:8762
>
> hi forezp,i am from port:8763

这说明当我们通过调用restTemplate.getForObject(“http://SERVICE-HI/hi?name=”+name,String.class)方法时，已经做了负载均衡，访问了不同的端口的服务实例。



**此时的架构?**

![SpringCloud笔记](http://img.bcoder.top/2018.12.08/11.png)

- 一个服务注册中心，eureka server,端口为8761
- service-hi工程跑了两个实例，端口分别为8762,8763，分别向服务注册中心注册
- sercvice-ribbon端口为8764,向服务注册中心注册
- 当sercvice-ribbon通过restTemplate调用service-hi的hi接口时，因为用ribbon进行了负载均衡，会轮流的调用service-hi：8762和8763 两个端口的hi接口；





### Ribbon核心组件IRule

IRule：根据特定算法中从服务列表中选取一个要访问的服务

+ **RoundRobinRule**：轮询（默认）
+ **RandomRule**：随机
+ **AvailabilityFilteringRule**:  会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务， 还有并发的连接数量超过阈值的服务，然后对剩余的服务列表按照轮询策略进行访问。
+ **WeightedResponseTimeRule**：根据平均响应时间计算所有服务的权重，响应时间越快服务权重越大被选中的概率越高。 刚启动时如果统计信息不足，则使用RoundRobinRule策略，等统计信息足够， 会切换到WeightedResponseTimeRule。
+ **RetryRule**：  先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试，获取可用的服务。
+ **BestAvailableRule**：会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务。
+ **ZoneAvoidanceRule**：默认规则,复合判断server所在区域的性能和server的可用性选择服务器。

## Feign负载均衡

### Feign简介

 Feign是一个声明式的Web服务客户端，使得编写Web服务客户端变得非常容易，
只需要创建一个接口，然后在上面添加注解即可。
参考官网：https://github.com/OpenFeign/feign 

 **Feign能干什么**
Feign旨在使编写Java Http客户端变得更容易。Feign是一个声明式的伪Http客户端，它使得写Http客户端变得更简单。使用Feign，只需要创建一个接口并注解。它具有可插拔的注解特性，可使用Feign 注解和JAX-RS注解。Feign支持可插拔的编码器和解码器。Feign默认集成了Ribbon，并和Eureka结合，默认实现了负载均衡的效果。

前面在使用Ribbon+RestTemplate时，利用RestTemplate对http请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用。所以，Feign在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。在Feign的实现下，我们只需创建一个接口并使用注解的方式来配置它(以前是Dao接口上面标注Mapper注解,现在是一个微服务接口上面标注一个Feign注解即可)，即可完成对服务提供方的接口绑定，简化了使用Spring cloud Ribbon时，自动封装服务调用客户端的开发量。



简而言之：

- Feign 采用的是基于接口的注解
- Feign 整合了ribbon，具有负载均衡的能力
- 整合了Hystrix，具有熔断的能力



### Feign实战



继续用上一节的工程， 启动eureka-server，端口为8761; 启动service-hi 两次，端口分别为8762 、8773.

新建一个spring-boot工程，取名为serice-feign，在它的pom文件引入Feign的起步依赖spring-cloud-starter-feign、Eureka的起步依赖spring-cloud-starter-netflix-eureka-client、Web的起步依赖spring-boot-starter-web，代码如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>top.bcoder</groupId>
    <artifactId>service-feign</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>service-feign</name>
    <description>Demo project for Spring Boot</description>


    <parent>
        <groupId>top.bcoder</groupId>
        <artifactId>sc-f-chapter3</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
    </dependencies>
    
    </project>
```

在工程的配置文件application.yml文件，指定程序名为service-feign，端口号为8765，服务注册地址为http://localhost:8761/eureka/ ，代码如下：

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 8765
spring:
  application:
    name: service-feign
```

在程序的启动类ServiceFeignApplication ，加上@EnableFeignClients注解开启Feign的功能：

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
@EnableFeignClients
public class ServiceFeignApplication {

    public static void main(String[] args) {
        SpringApplication.run( ServiceFeignApplication.class, args );
    }
}
```

定义一个feign接口，通过@ FeignClient（“服务名”），来指定调用哪个服务。比如在代码中调用了service-hi服务的“/hi”接口，代码如下：

```java
@FeignClient(value = "service-hi")
public interface SchedualServiceHi {
    @RequestMapping(value = "/hi",method = RequestMethod.GET)
    String sayHiFromClientOne(@RequestParam(value = "name") String name);
}
```

在Web层的controller层，对外暴露一个”/hi”的API接口，通过上面定义的Feign客户端SchedualServiceHi 来消费服务。代码如下：

```java
@RestController
public class HiController {


    //编译器报错，无视。 因为这个Bean是在程序启动的时候注入的，编译器感知不到，所以报错。
    @Autowired
    SchedualServiceHi schedualServiceHi;

    @GetMapping(value = "/hi")
    public String sayHi(@RequestParam String name) {
        return schedualServiceHi.sayHiFromClientOne( name );
    }
}
```

启动程序，多次访问http://localhost:8765/hi?name=forezp,浏览器交替显示：

> hi forezp,i am from port:8762
>
> hi forezp,i am from port:8763

Feign源码解析：http://blog.csdn.net/forezp/article/details/73480304



## Hystrix断路器



### Hystrix概述

**1.分布式系统面临的问题？**


复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免地失败。这就是服务雪崩。



![SpringCloud笔记](http://img.bcoder.top/2018.12.08/12.png)



**什么是服务雪崩**
多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这就是所谓的“扇出”。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”.

对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源都在几秒钟内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。

在微服务架构中，根据业务来拆分成一个个的服务，服务与服务之间可以相互调用（RPC），在Spring Cloud可以用RestTemplate+Ribbon和Feign来调用。为了保证其高可用，单个服务通常会集群部署。由于网络原因或者自身的原因，服务并不能保证100%可用，如果单个服务出现问题，调用这个服务就会出现线程阻塞，此时若有大量的请求涌入，Servlet容器的线程资源会被消耗完毕，导致服务瘫痪。服务与服务之间的依赖性，故障会传播，会对整个微服务系统造成灾难性的严重后果，这就是服务故障的“雪崩”效应。



**备注：一般情况对于服务依赖的保护主要有3中解决方案：**

（1）熔断模式：这种模式主要是参考电路熔断，如果一条线路电压过高，保险丝会熔断，防止火灾。放到我们的系统中，如果某个目标服务调用慢或者有大量超时，此时，熔断该服务的调用，对于后续调用请求，不在继续调用目标服务，直接返回，快速释放资源。如果目标服务情况好转则恢复调用。

（2）隔离模式：这种模式就像对系统请求按类型划分成一个个小岛的一样，当某个小岛被火少光了，不会影响到其他的小岛。例如可以对不同类型的请求使用线程池来资源隔离，每种类型的请求互不影响，如果一种类型的请求线程资源耗尽，则对后续的该类型请求直接返回，不再调用后续资源。这种模式使用场景非常多，例如将一个服务拆开，对于重要的服务使用单独服务器来部署，再或者公司最近推广的多中心。

（3）限流模式：上述的熔断模式和隔离模式都属于出错后的容错处理机制，而限流模式则可以称为预防模式。限流模式主要是提前对各个类型的请求设置最高的QPS阈值，若高于设置的阈值则对该请求直接返回，不再调用后续资源。这种模式不能解决服务依赖的问题，只能解决系统整体资源分配问题，因为没有被限流的请求依然有可能造成雪崩效应。



**2.Hystrix是什么？**


Hystrix是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。

“断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

官方文档：https://github.com/Netflix/Hystrix/wiki/How-To-Use



**3.Hystrix能干嘛?**

**(1)服务降级**

Hystrix服务降级，其实就是线程池中单个线程障处理，防止单个线程请求时间太长，导致资源长期被占有而得不到释放，从而导致线程池被快速占用完，导致服务崩溃。简单来说，整体资源快不够了，忍痛将某些服务先关掉，待渡过难关，再开启回来。
Hystrix能解决如下问题：
1.请求超时降级，线程资源不足降级，降级之后可以返回自定义数据
2.线程池隔离降级，分布式服务可以针对不同的服务使用不同的线程池，从而互不影响
3.自动触发降级与恢复
4.实现请求缓存和请求合并



**(2)服务熔断**

熔断模式：这种模式主要是参考电路熔断，如果一条线路电压过高，保险丝会熔断，防止火灾。放到我们的系统中，如果某个目标服务调用慢或者有大量超时，此时，熔断该服务的调用，对于后续调用请求，不在继续调用目标服务，直接返回，快速释放资源。如果目标服务情况好转则恢复调用。



**(3)服务限流**

限流模式主要是提前对各个类型的请求设置最高的QPS阈值，若高于设置的阈值则对该请求直接返回，不再调用后续资源。这种模式不能解决服务依赖的问题，只能解决系统整体资源分配问题，因为没有被限流的请求依然有可能造成雪崩效应。



**(4)接近实时的监控**



### Hystrix断路器

#### 在Ribbon使用断路器

改造serice-ribbon 工程的代码，首先在pox.xml文件中加入spring-cloud-starter-netflix-hystrix的起步依赖：

```xml
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
  </dependency>
```

在程序的启动类ServiceRibbonApplication 加@EnableHystrix注解开启Hystrix：

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
@EnableHystrix
public class ServiceRibbonApplication {

    public static void main(String[] args) {
        SpringApplication.run( ServiceRibbonApplication.class, args );
    }

    @Bean
    @LoadBalanced
    RestTemplate restTemplate() {
        return new RestTemplate();
    }

}
```

改造HelloService类，在hiService方法上加上@HystrixCommand注解。该注解对该方法创建了熔断器的功能，并指定了fallbackMethod熔断方法，熔断方法直接返回了一个字符串，字符串为”hi,”+name+”,sorry,error!”，代码如下：

```java
@Service
public class HelloService {

    @Autowired
    RestTemplate restTemplate;

    @HystrixCommand(fallbackMethod = "hiError")
    public String hiService(String name) {
        return restTemplate.getForObject("http://SERVICE-HI/hi?name="+name,String.class);
    }

    public String hiError(String name) {
        return "hi,"+name+",sorry,error!";
    }

}
```

启动：service-ribbon 工程，当我们访问http://localhost:8764/hi?name=forezp,浏览器显示：

> hi forezp,i am from port:8762

此时关闭 service-hi 工程，当我们再访问http://localhost:8764/hi?name=forezp，浏览器会显示：

> hi ,forezp,orry,error!

这就说明当 service-hi 工程不可用的时候，service-ribbon调用 service-hi的API接口时，会执行快速失败，直接返回一组字符串，而不是等待响应超时，这**很好的控制了容器的线程阻塞**。

#### Feign中使用断路器

Feign是自带断路器的，在D版本的Spring Cloud之后，它没有默认打开。需要在配置文件中配置打开它，在配置文件加以下代码：

> feign.hystrix.enabled=true

基于service-feign工程进行改造，只需要在FeignClient的SchedualServiceHi接口的注解中加上fallback的指定类就行了：

```java
@FeignClient(value = "service-hi",fallback = SchedualServiceHiHystric.class)
public interface SchedualServiceHi {
    @RequestMapping(value = "/hi",method = RequestMethod.GET)
    String sayHiFromClientOne(@RequestParam(value = "name") String name);
}
```

SchedualServiceHiHystric需要实现SchedualServiceHi 接口，并注入到Ioc容器中，代码如下：

```java
@Component
public class SchedualServiceHiHystric implements SchedualServiceHi {
    @Override
    public String sayHiFromClientOne(String name) {
        return "sorry "+name;
    }
}
```

启动四servcie-feign工程，浏览器打开http://localhost:8765/hi?name=forezp,注意此时service-hi工程没有启动，网页显示：

> sorry forezp

打开service-hi工程，再次访问，浏览器显示：

> hi forezp,i am from port:8762

这证明断路器起到作用了。



### 服务监控hystrixDashboard

除了隔离依赖服务的调用以外，Hystrix还提供了准实时的调用监控（Hystrix Dashboard），Hystrix会持续地记录所有通过Hystrix发起的请求的执行信息，并以统计报表和图形的形式展示给用户，包括每秒执行多少请求多少成功，多少失败等。Netflix通过hystrix-metrics-event-stream项目实现了对以上指标的监控。Spring Cloud也提供了Hystrix Dashboard的整合，对监控内容转化成可视化界面。



## zuul路由网关



### zuul概述

在微服务架构中，需要几个基础的服务治理组件，包括服务注册与发现、服务消费、负载均衡、断路器、智能路由、配置管理等，由这几个基础组件相互协作，共同组建了一个简单的微服务系统。一个简单的微服务系统如下图：

![SpringCloud笔记](http://img.bcoder.top/2018.12.08/13.png)



Zuul包含了对请求的**路由和过滤**两个最主要的功能：
其中路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础而过滤器功能则负责对请求的处理过程进行干预，是实现请求校验、服务聚合等功能的基础.Zuul和Eureka进行整合，将Zuul自身注册为Eureka服务治理下的应用，同时从Eureka中获得其他微服务的消息，也即以后的访问微服务都是通过Zuul跳转后获得。

注意：Zuul服务最终还是会注册进Eureka



**zuul有以下功能：**

- Authentication
- Insights
- Stress Testing
- Canary Testing
- Dynamic Routing
- Service Migration
- Load Shedding
- Security
- Static Response handling
- Active/Active traffic management



### zuul实战

#### zuul路由

创建service-zuul工程，其pom.xml文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>top.bcoder</groupId>
    <artifactId>service-zuul</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>service-zuul</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>top.bcoder</groupId>
        <artifactId>sc-f-chapter5</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>
    </dependencies>
</project>
```

在其入口applicaton类加上注解@EnableZuulProxy，开启zuul的功能：

```java
@SpringBootApplication
@EnableZuulProxy
@EnableEurekaClient
@EnableDiscoveryClient
public class ServiceZuulApplication {

    public static void main(String[] args) {
        SpringApplication.run( ServiceZuulApplication.class, args );
    }
}
```

加上配置文件application.yml加上以下的配置代码：

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 8769
spring:
  application:
    name: service-zuul
zuul:
  routes:
    api-a:
      path: /api-a/**
      serviceId: service-ribbon
    api-b:
      path: /api-b/**
      serviceId: service-feign
```

首先指定服务注册中心的地址为http://localhost:8761/eureka/，服务的端口为8769，服务名为service-zuul；以/api-a/ 开头的请求都转发给service-ribbon服务；以/api-b/开头的请求都转发给service-feign服务；

依次运行这五个工程;打开浏览器访问：http://localhost:8769/api-a/hi?name=forezp ;浏览器显示：

> hi forezp,i am from port:8762

打开浏览器访问：http://localhost:8769/api-b/hi?name=forezp ;浏览器显示：

> hi forezp,i am from port:8762

这说明zuul起到了路由的作用



#### zuul服务过滤

zuul不仅只是路由，并且还能过滤，做一些安全验证。继续改造工程；

```java
@Component
public class MyFilter extends ZuulFilter {

    private static Logger log = LoggerFactory.getLogger(MyFilter.class);
    @Override
    public String filterType() {
        return "pre";
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

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

- filterType：返回一个字符串代表过滤器的类型，在zuul中定义了四种不同生命周期的过滤器类型，具体如下：
  - pre：路由之前
  - routing：路由之时
  - post： 路由之后
  - error：发送错误调用
- filterOrder：过滤的顺序
- shouldFilter：这里可以写逻辑判断，是否要过滤，本文true,永远过滤。
- run：过滤器的具体逻辑。可用很复杂，包括查sql，nosql去判断该请求到底有没有权限访问。



这时访问：http://localhost:8769/api-a/hi?name=forezp ；网页显示：

> token is empty

访问 http://localhost:8769/api-a/hi?name=forezp&token=22 ； 网页显示：

> hi forezp,i am from port:8762



## SpringCloud Config分布式配置中心

### SpringCloud Config简介

**1.分布式系统面临的配置问题**

微服务意味着要将单体应用中的业务拆分成一个个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以一套集中式的、动态的配置管理设施是必不可少的。SpringCloud提供了ConfigServer来解决这个问题，我们每一个微服务自己带着一个application.yml，上百个配置文件的管理......



**2.SpringCloud Config是什么？**

![SpringCloud笔记](http://img.bcoder.top/2018.12.08/14.png)

SpringCloud Config为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置。



SpringCloud Config为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置。



**3.SpringCloud Config能干嘛？**

集中管理配置文件
不同环境不同配置，动态化的配置更新，分环境部署比如dev/test/prod/beta/release
运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取配置自己的信息
当配置发生变动时，服务不需要重启即可感知到配置的变化并应用新的配置
将配置信息以REST接口的形式暴露



**4.与GitHub整合配置**

由于SpringCloud Config默认使用Git来存储配置文件(也有其它方式,比如支持SVN和本地文件)，但最推荐的还是Git，而且使用的是http/https访问的形式



### SpringCloud Config实战

#### 构建Config Server

父maven工程省略，父pom文件：

```xml 
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>top.bcoder</groupId>
    <artifactId>sc-f-chapter6</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>

    <modules>
        <module>config-server</module>
        <module>config-client</module>
    </modules>

    <name>sc-f-chapter6</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
        <relativePath/>
    </parent>

   
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


</project>
```

创建一个spring-boot项目，取名为config-server,其pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>top.bcoder</groupId>
	<artifactId>config-server</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>config-server</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>top.bcoder</groupId>
		<artifactId>sc-f-chapter6</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-config-server</artifactId>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>


</project>
```

在程序的入口Application类加上@EnableConfigServer注解开启配置服务器的功能，代码如下：

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigServerApplication.class, args);
	}
}
```

需要在程序的配置文件application.properties文件配置以下：

```properties
spring.application.name=config-server
server.port=8888

spring.cloud.config.server.git.uri=https://github.com/forezp/SpringcloudConfig/
spring.cloud.config.server.git.searchPaths=respo
spring.cloud.config.label=master
spring.cloud.config.server.git.username=
spring.cloud.config.server.git.password=
```

- spring.cloud.config.server.git.uri：配置git仓库地址
- spring.cloud.config.server.git.searchPaths：配置仓库路径
- spring.cloud.config.label：配置仓库的分支
- spring.cloud.config.server.git.username：访问git仓库的用户名
- spring.cloud.config.server.git.password：访问git仓库的用户密码



如果Git仓库为公开仓库，可以不填写用户名和密码，如果是私有仓库需要填写，本例子是公开仓库，放心使用。

远程仓库https://github.com/forezp/SpringcloudConfig/ 中有个文件config-client-dev.properties文件中有一个属性：

> foo = foo version 3

启动程序：访问http://localhost:8888/foo/dev



```json
{"name":"foo","profiles":["dev"],"label":"master",
"version":"792ffc77c03f4b138d28e89b576900ac5e01a44b","state":null,"propertySources":[]}
```

证明配置服务中心可以从远程程序获取配置信息。

http请求地址和资源文件映射如下:

- /{application}/{profile}[/{label}]
- /{application}-{profile}.yml
- /{label}/{application}-{profile}.yml
- /{application}-{profile}.properties
- /{label}/{application}-{profile}.properties

#### 构建Config Client

重新创建一个springboot项目，取名为config-client,其pom文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>top.bcoder</groupId>
	<artifactId>config-client</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>config-client</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>top.bcoder</groupId>
		<artifactId>sc-f-chapter6</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>


</project>
```

其配置文件**bootstrap.properties**：

```properties
spring.application.name=config-client
spring.cloud.config.label=master
spring.cloud.config.profile=dev
spring.cloud.config.uri= http://localhost:8888/
server.port=8881
```

- spring.cloud.config.label 指明远程仓库的分支
- spring.cloud.config.profile
  - dev开发环境配置文件
  - test测试环境
  - pro正式环境
- spring.cloud.config.uri= http://localhost:8888/ 指明配置服务中心的网址。



程序的入口类，写一个API接口“／hi”，返回从配置中心读取的foo变量的值，代码如下：

```java
@SpringBootApplication
@RestController
public class ConfigClientApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigClientApplication.class, args);
	}

	@Value("${foo}")
	String foo;
	@RequestMapping(value = "/hi")
	public String hi(){
		return foo;
	}
}
```

打开网址访问：http://localhost:8881/hi，网页显示：

> foo version 3

这就说明，config-client从config-server获取了foo的属性，而config-server是从git仓库读取的,如图：

![SpringCloud笔记](http://img.bcoder.top/2018.12.08/15.png)

#### 高可用的分布式配置中心搭建

上一小节，讲述了一个服务如何从配置中心读取文件，配置中心如何从远程git读取配置文件，当服务实例很多时，都从配置中心读取文件，这时可以考虑将配置中心做成一个微服务，将其集群化，从而达到高可用，架构图如下：

![SpringCloud笔记](http://img.bcoder.top/2018.12.08/16.png)

**1.创建一个eureka-server工程，用作服务注册中心。**

在其pom.xml文件引入Eureka的起步依赖spring-cloud-starter-netflix- eureka-server，代码如下:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>top.bcoder</groupId>
	<artifactId>config-server</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>config-server</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>top.bcoder</groupId>
		<artifactId>sc-f-chapter7</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-config-server</artifactId>
		</dependency>

	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>



</project>
```

在配置文件application.yml上，指定服务端口为8889，加上作为服务注册中心的基本配置，代码如下：

```yaml
erver:
  port: 8889

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

入口类：

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaServerApplication.class, args);
	}
}
```



**2.改造config-server**

在其pom.xml文件加上EurekaClient的起步依赖spring-cloud-starter-netflix-eureka-client，代码如下:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
</dependencies>
```

配置文件application.yml，指定服务注册地址为http://localhost:8889/eureka/，其他配置之前，完整的配置如下：

```properties
spring.application.name=config-server
server.port=8888

spring.cloud.config.server.git.uri=https://github.com/forezp/SpringcloudConfig/
spring.cloud.config.server.git.searchPaths=respo
spring.cloud.config.label=master
spring.cloud.config.server.git.username= your username
spring.cloud.config.server.git.password= your password
eureka.client.serviceUrl.defaultZone=http://localhost:8889/eureka/
```

最后需要在程序的启动类Application加上@EnableEureka的注解。



**3.改造config-client**

将其注册微到服务注册中心，作为Eureka客户端，需要pom文件加上起步依赖spring-cloud-starter-netflix-eureka-client，代码如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
</dependencies>
```

配置文件bootstrap.properties，注意是bootstrap。加上服务注册地址为http://localhost:8889/eureka/

```properties
spring.application.name=config-client
spring.cloud.config.label=master
spring.cloud.config.profile=dev
#spring.cloud.config.uri= http://localhost:8888/

eureka.client.serviceUrl.defaultZone=http://localhost:8889/eureka/
spring.cloud.config.discovery.enabled=true
spring.cloud.config.discovery.serviceId=config-server
server.port=8881
```

- spring.cloud.config.discovery.enabled 是从配置中心读取文件。
- spring.cloud.config.discovery.serviceId 配置中心的servieId，即服务名。

这时发现，在读取配置文件不再写ip地址，而是服务名，这时如果配置服务部署多份，通过负载均衡，从而高可用。

依次启动eureka-servr,config-server,config-client 访问网址：http://localhost:8889/

访问http://localhost:8881/hi，浏览器显示：

> foo version 3

## Spring Cloud Bus消息总线

Spring Cloud Bus 将分布式的节点用轻量的消息代理连接起来。它可以用于广播配置文件的更改或者服务之间的通讯，也可以用于监控。本文要讲述的是用Spring Cloud Bus实现通知微服务架构的配置文件的更改。



### Spring Cloud Bus介绍

Spring Cloud Bus 将分布式的节点用轻量的消息代理连接起来。它可以用于广播配置文件的更改或者服务之间的通讯，也可以用于监控。本文要讲述的是用Spring Cloud Bus实现通知微服务架构的配置文件的更改。



### Spring Cloud Bus实战

本文还是基于上一节文章来实现。按照官方文档，我们只需要在配置文件中配置 spring-cloud-starter-bus-amqp ；这就是说我们需要装rabbitMq，点击[rabbitmq](http://www.rabbitmq.com/)下载。至于怎么使用 rabbitmq，搜索引擎下。

**改造config-client**

在pom文件加上起步依赖spring-cloud-starter-bus-amqp，完整的配置文件如下：

```xml
<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-bus-amqp</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
```

在配置文件application.properties中加上RabbitMq的配置，包括RabbitMq的地址、端口，用户名、密码。并需要加上spring.cloud.bus的三个配置，具体如下：

```properties
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest

spring.cloud.bus.enabled=true
spring.cloud.bus.trace.enabled=true
management.endpoints.web.exposure.include=bus-refresh
```

ConfigClientApplication启动类代码如下：

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
@RestController
@RefreshScope
public class ConfigClientApplication {

	/**
	 * http://localhost:8881/actuator/bus-refresh
	 */

	public static void main(String[] args) {
		SpringApplication.run(ConfigClientApplication.class, args);
	}

	@Value("${foo}")
	String foo;

	@RequestMapping(value = "/hi")
	public String hi(){
		return foo;
	}
}
```

依次启动eureka-server、confg-cserver,启动两个config-client，端口为：8881、8882。

访问http://localhost:8881/hi 或者http://localhost:8882/hi 浏览器显示：

> foo version 3

这时我们去[代码仓库](https://github.com/forezp/SpringcloudConfig/blob/master/respo/config-client-dev.properties)将foo的值改为“foo version 4”，即改变配置文件foo的值。如果是传统的做法，需要重启服务，才能达到配置文件的更新。此时，我们只需要发送post请求：http://localhost:8881/actuator/bus-refresh，你会发现config-client会重新读取配置文件

![SpringCloud笔记](http://img.bcoder.top/2018.12.08/17.png)



重新读取配置文件：

![SpringCloud笔记](http://img.bcoder.top/2018.12.08/18.png)

这时我们再访问http://localhost:8881/hi 或者http://localhost:8882/hi 浏览器显示：

> foo version 4

另外，/actuator/bus-refresh接口可以指定服务，即使用”destination”参数，比如 “/actuator/bus-refresh?destination=customers:**” 即刷新服务名为customers的所有服务。

此时的架构图：

![SpringCloud笔记](http://img.bcoder.top/2018.12.08/19.png)



当git文件更改的时候，通过pc端用post 向端口为8882的config-client发送请求/bus/refresh／；此时8882端口会发送一个消息，由消息总线向其他服务传递，从而使整个微服务集群都达到更新配置文件。

























