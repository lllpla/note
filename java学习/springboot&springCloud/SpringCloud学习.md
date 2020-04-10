# 一、什么是Spring Cloud

Spring Cloud是一系列框架的有序集合。 利用Springboot的开发便利性简化了分布式系统基础设置的开发。它将各家公司开发比较成熟的服务框架组合起来，通过Springboot风格进行再封装屏蔽掉了复杂的配置和实现原理。

 

# 二、核心成员

## Netflix Eureka



 **服务中心**，云端服务发现，基于Rest的服务。用于定位服务、注册服务，以实现云端中间层服务发现和故障转移。

> 任何小弟需要其它小弟支持什么都需要从这里来拿，同样的你有什么独门武功的都赶紧过报道，方便以后其它小弟来调用；它的好处是你不需要直接找各种什么小弟支持，只需要到服务中心来领取，也不需要知道提供支持的其它小弟在哪里，还是几个小弟来支持的，反正拿来用就行，服务中心来保证稳定性和质量

## Netflix Hystrix



**熔断器**，容错管理工具。通过熔断机制控制服务和第三方库的节点，从而对延迟和故障提供更强大的容错。

> 比如突然某个小弟生病了，但是你还需要它的支持，然后调用之后它半天没有响应，你却不知道，一直在等等这个响应；有可能别的小弟也正在调用你的武功绝技，那么当请求多之后，就会发生严重的阻塞影响老大的整体计划。这个时候Hystrix就派上用场了，当Hystrix发现某个小弟不在状态不稳定立马马上让它下线，让其它小弟来顶上来，或者给你说不用等了这个小弟今天肯定不行，该干嘛赶紧干嘛去别在这排队了。

## Netflix Zuul

动态路由、监控，弹性、安全。

> Zuul相当于是设备和Netflix流应用的Web网站后端所有请求的前门。当其他门派来找大哥办事的时候一定要先经过zuul，看下有没有带刀子什么的给拦截回去，或者需要找哪个小弟，就直接给带过去。

## Netflix Archaius

**配置管理API**，包含一系列配置管理API，提供动态类型化属性、线程安全配置操作、沦胥框架、回调机制等功能。可以实现动态配置。

原理是每隔60s（默认，可修改）从配置源读取一次内容，这样修改了配置文件就不需要重启服务就可以使修改后的内容生效。

## Spring Cloud Config

**配置中心**,配置管理工具包，让你可以把配置放到远程服务器，集中化管理集群配置，目前支持本地存储、Git以及Subversion。

## Spring Cloud Bus

**事件、消息总线**，用于在集群中传播状态变化，可与Spring Cloud Config联合实现热部署。

## Spring Cloud for Cloud Foundry

Cloud Foundry是VMware推出的业界第一个开源PaaS云平台，它支持多种框架、语言、运行时环境、云平台及应用服务，使开发人员能够在几秒钟内进行应用程序的部署和扩展，无需担心任何基础架构的问题

## Spring Cloud Cluster

Spring Cloud Cluster将取代Spring Integration。提供在分布式系统中的集群所需要的基础功能支持，如：选举、集群的状态一致性、全局锁、tokens等常见状态模式的抽象和实现。

## Spring Cloud Consul

Consul 是一个支持多数据中心分布式高可用的服务发现和配置共享的服务软件,由 HashiCorp 公司用 Go 语言开发, 基于 Mozilla Public License 2.0 的协议进行开源. Consul 支持健康检查,并允许 HTTP 和 DNS 协议调用 API 存储键值对.

Spring Cloud Consul 封装了Consul操作，consul是一个服务发现与配置工具，与Docker容器可以无缝集成。

## Spring Cloud Security

基于spring security的安全工具包，为你的应用程序添加安全控制。

> 这个小弟很牛鼻专门负责整个帮派的安全问题，设置不同的门派访问特定的资源，不能把秘籍葵花宝典泄漏了。

## Spring Cloud Sleuth

日志收集工具包，封装了Dapper和log-based追踪以及Zipkin和HTrace操作，为SpringCloud应用实现了一种分布式追踪解决方案。

## Spring Cloud Data Flow

Data flow 是一个用于开发和执行大范围数据处理其模式包括ETL，批量运算和持续运算的统一编程模型和托管服务。原生云可编配的服务。

## Spring Cloud Stream

**Spring Cloud Stream是创建消息驱动微服务应用的框架**。Spring Cloud Stream是基于Spring Boot创建，用来建立单独的／工业级spring应用，使用spring integration提供与消息代理之间的连接。数据流操作开发包，**封装了与Redis,Rabbit、Kafka等发送接收消息。**

一个业务会牵扯到多个任务，任务之间是通过事件触发的，这就是Spring Cloud stream要干的事了

## Spring Cloud Task

Spring Cloud Task 主要解决短命微服务的任务管理，任务调度的工作，比如说某些定时任务晚上就跑一次，或者某项数据分析临时就跑几次。

## Spring Cloud Zookeeper

ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。ZooKeeper的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户。

操作Zookeeper的工具包，用于使用zookeeper方式的服务发现和配置管理，抱了Zookeeper的大腿。

## Spring Cloud Connectors

Spring Cloud Connectors 简化了连接到服务的过程和从云平台获取操作的过程，有很强的扩展性，可以利用Spring Cloud Connectors来构建你自己的云平台。

便于云端应用程序在各种PaaS平台连接到后端，如：数据库和消息代理服务。

##  Spring Cloud Starters

Spring Boot式的启动项目，为Spring Cloud提供开箱即用的依赖管理。

## Spring Cloud CLI

基于 Spring Boot CLI，可以让你以命令行方式快速建立云组件。