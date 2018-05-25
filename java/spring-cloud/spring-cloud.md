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

### 构建服务注册中心
1. http://start.spring.io/ 使用 Gradle 或 Maven 建立 Spring Boot 2.0 版本工程，并勾选
    建议将 Artifact 设定为 eureka-server。 之后将下载的项目导入 IDEA 或 Eclipse 等 IDE 中。
2. build.gradle 文件的内容如下：
~~~groovy
buildscript {
ext {
    springBootVersion = '2.0.0.RELEASE'
}
repositories {
mavenCentral()
}
dependencies {
    classpath("org.springframework.boot:spring-boot-gradleplugin:${springBootVersion}")
    }
}
apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'
group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8
repositories {
mavenCentral()
    maven { url "https://repo.spring.io/milestone" }
}
ext {
    springCloudVersion = 'Finchley.M8'
}
dependencies {
    compile('org.springframework.cloud:spring-cloud-starter-netflix-eureka-server')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
dependencyManagement {
imports {
    mavenBom "org.springframework.cloud:spring-clouddependencies:${springCloudVersion}"
    }
}
~~~
3. 在 EurekaServerApplication 上加上~@EnableEurekaServer~
4. 在 application.properties 中加入如下内容：

~~~properties
server.port=10000
eureka.instance.hostname=localhost
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
eureka.client.service-url.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/
~~~

5. 启动代码并访问 http://localhost:10000/

### 注册客户端到 eureka 服务器

1. 根据上面服务器的方式再次建立项目，但是需要在 Spring Boot 中多选择 Web 模块。 建立项目名称叫做 hello-service
2. 在在 EurekaClientApplication 上加上@EnableEurekaClient
3. 建立 RESTful 风格的控制器
~~~java
package com.example.eurekaclient1.controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
@RestController
public class HomeController {
        @RequestMapping
        public String home() {
        return "Hello, World";
        }
}
~~~
4. 在 application.properties 中加入如下内容：
~~~properties
spring.application.name=hello-service
eureka.client.service-url.defaultZone=http://localhost:10000/eureka
~~~
5. 启动该程序，在浏览器访问 http://localhost:8080/
6. 我们在该客户端输出中看到类似内容：
DiscoveryClient_HELLO-SERVICE/Peters-MacBook.local:hello-service: registering service...
DiscoveryClient_HELLO-SERVICE/Peters-MacBook.local:hello-service - registration status:
204
7. 在服务器的另一端看到类似输出：
Registered instance HELLO-SERVICE/Peters-MacBook.local:hello-service with status UP
(replication=false)
8. 访问 http://localhost:10000 


### 服务发现与消费

1. 建立 Spring Boot 项目命名为 ribbon-consumer，加入如下依赖：
~~~groovy
compile('org.springframework.boot:spring-boot-starter-web')
compile('org.springframework.cloud:spring-cloud-starter-netflix-eureka-server')
compile('org.springframework.cloud:spring-cloud-starter-netflix-ribbon')
~~~

2. 启动 2 个服务注册中心和 2 个 hello-service 服务并运行在不同端口

3. 在 RibbonConsumerApplication 加入如下内容：
~~~java
package com.example.ribbonconsumer;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;
@SpringBootApplication
@EnableDiscoveryClient
public class RibbonConsumerApplication {
@Bean
@LoadBalanced
RestTemplate restTemplate() {
    return new RestTemplate();
}
public static void main(String[] args) {
    SpringApplication.run(RibbonConsumerApplication.class, args);
}
}
~~~
4. 创建 ConsumerController
~~~java
package com.example.ribbonconsumer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;
@RestController
public class ConsumerController {
@Autowired
private RestTemplate restTemplate;
@RequestMapping("/ribbon-consumer")
        public String helloConsumer() {
            return restTemplate.getForEntity("http://HELLO-SERVICE/", String.class).getBody();
        }
}
~~~
注意 http://HELLO-SERVICE/

5. 修改 application.properties
~~~properties
spring.application.name=ribbon-consumer
server.port=10003
eureka.client.service-url.defaultZone=http://localhost:10000/eureka/
~~~
6. 通过 localhost:10003/ribbon-consumer 发起请求，得到 hello，world

## Spring Cloud Ribbon

### 什么是 Spring Cloud Ribbon

Spring Cloud Ribbon 是一个基于 HTTP 和 TCP 的客户端负载均衡工具，它基于 Netflix Ribbon 实现。通过 Spring Cloud 的封
装，可以让我们轻松地将面向服务的 REST 模版请求自动转换成客户端负责均衡的服务调用。

- 客户端负载均衡
客户端负载均衡和服务器端负载均衡最大的不同点在于服务清单所存储的位置。在客户端负载均衡中，所有客户端节点都护着自己要
访问的服务端清单，而这种服务端的清单来自于服务注册中心（比如 Eureka 客户端）。同服务端负载均更的架构类似，在客户端负
载均衡中也需要心跳去维护服务器端清单的健康性，只是这个步骤需要配合服务注册中心来完成。

在 Spring Cloud 实现的服务治理框架中，默认会创建针对各个服务治理框架的 Ribbon 自动化整合配置。
通过 Spring Cloud Ribbon 的封装，我们在微服务架构中使用客户端负载均衡调用非常简单：
1. 服务提供者只需要启动多个服务实例并注册到一个注册中心或是多个相关联的服务注册中心
2. 服务消费者直接通过调用@LoadBalanced 注解修饰过的 RestTemplate 来实现面向服务的接口调用
~~~java
@SpringBootApplication
@EnableDiscoveryClient
public class RibbonConsumerApplication {
@Bean
@LoadBalanced
RestTemplate restTemplate() {
return new RestTemplate();
}
public static void main(String[] args) {
SpringApplication.run(RibbonConsumerApplication.class, args);
}
}
~~~

~~~java
package com.example.ribbonconsumer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;
@RestController
public class ConsumerController {
@Autowired
private RestTemplate restTemplate;
@RequestMapping("/ribbon-consumer")
public String helloConsumer() {
return restTemplate
.getForEntity("http://HELLO-SERVICE/", String.class)
.getBody();
}
}
~~~
3. 在 application.properties 中加入
~~~properties
spring.application.name=ribbon-consumer
server.port=1000
eureka.client.service-url.defaultZone=http://peer1:10000/eureka/
eureka.server.enable-self-preservation=false
~~~
4. 启动 ribbon-consumer，观察它的控制台信息。 Robbin 输出了当前客户端维护的 HELLO-SERVICE 的服务列表情况。其
中包含了各个实例的位置，Ribbon 就是按照此信息进行轮询访问，以实现基于客户端的负载均衡。另外还输出了一些非常
有用的信息，如对各个实例的请求总数量、第一次连接信息，上一次连接信息，总的请求失败数量等。

5. 再次尝试发送几次请求，并观察 HELLO-SERVICE 控制台，可以看到两个控制台交替打印日志（日志中包含了我们代码中的
输出），由此可以判断当前 ribbon-consumer 对 HELLO-SERVICE 的调用是负载均衡的。
### RestTemplate 详解
- GET 请求
在 RestTemplate 中，对 GET 请求可以通过如下两种方式进行调用实现：
1. getForEntity 方法。该方法的返回值时 ResponseEntity，该对象时 Spring 对 HTTP 请求响应的封装，其中主要存储了
HTTP 的几个重要元素，比如请求状态码的枚举对象 HttpStatus，在它的父类 HttpEntity 中还存储着 HTTP 请求的头信息
对象 HttpHeaders 以及范型类型的请求体对象，比如：
~~~java
RestTemplate restTemplate = new RestTemplate();
ResponseEntity<String> responseEntity = restTemplate.getForEntity(http://USERSERVICE/user?name={1}, String.class, "tom");
String body = responseEntity.getBody();
~~~
或者
~~~java
ResponseEntity<User> responseEntity = restTemplate.getForEntity(http://USERSERVICE/user?name={1}, User.class, "tom");
String body = responseEntity.getBody();
~~~
2. getForObject 方法。该方法可以理解为 getFotEntity 的进一步封装，它通过 HttpMessageConverterExtractor 对 HTTP
的请求响应体 body 内容进行对象转换，实现请求直接返回包装好的对象内容，比如：
~~~java
RestTemplate restTemplate = new RestTemplate();
String result = restTemplate.getForObject(uri, String.class);
~~~
或者
~~~java
User result = restTemplate.getForObject(uri, User.class);
~~~
当不需要关注请求响应除了 body 外的其它内容时，该函数非常好用。