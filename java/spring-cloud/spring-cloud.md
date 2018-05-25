## spring-cloud 简介
Spring Cloud 是一个基于 Spring Boot 实现的微服务架构开发工具。它为微服务架构中涉及的配置管理、服务治理、断路器、智能
路由、微代理、控制总线、全局锁、决策竞选、分布式会话和集群状态管理等操作提供了一种简单的开发方式。

Spring Cloud 包含了多个子项目：

- Spring Cloud Config
-  Spring Cloud Netflix
- Spring Cloud Bus
- Spring Cloud Cluster
- Spring Cloud Cloudfoundry
- Spring Cloud Consul
- Spring Cloud Stream
- Spring Cloud AWS
- Spring Cloud Security
- Spring Cloud Sleuth
- Spring Cloud ZooKeeper
- Spring Cloud Starters
- Spring Cloud CLI: 用于在 Groovy 中快速创建 Spring Cloud 应用的 Spring Boot CLI 插件

## Spring Cloud Eureka

Spring Cloud Eureka 是 Spring Cloud Netflix 微服务套件中的一部分，它基于 Netflix Eureka 做了二次封装，主要负责完成微服
务架构中的服务治理功能。

服务治理是微服务架构中最为核心和基础的模块，它主要用来实现各个微服务实例的自动化注册与发现

### Eureka 服务端
也称为服务注册中心。支持高可用配置。如果 Eureka 以集群模式部署，当集群中有分片出现故障时，那么 Eureka 就转入自我保护
模式。
### Eureka 客户端
主要处理服务的注册与发现。客户端服务通过注解和参数配置的方式，嵌入在客户端应用程序的代码中，在应用程序运行时，
Eureka 客户端向注册中心注册自身提供的服务并周期性的发送心跳来更新它的服务租约。同时它也能从服务端查询当前注册的服务
信息并把它们缓存到本地并周期性的刷新服务状态。


