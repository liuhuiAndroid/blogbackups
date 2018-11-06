## 第1章 Spring Cloud介绍
#### 1-1 SpringCloud导学
Spring Cloud利用Spring Boot简化构建分布式应用。
- Spring Cloud Eureka
Eureka Server、Eureka Client、Eureka 高可用、服务发现机制
- Spring Cloud Config
Config Server、Config Client、Spring Cloud Bus（结合RabbitMQ）、自动刷新
- Spring Cloud Ribbon
RestTemplate、Feign、Ribbon（分析源码，了解底层）
- Spring Cloud Zuul
动态路由、校验
- Spring Cloud Hystrix
Hystrix Dashboar、熔断机制

![image.png](https://upload-images.jianshu.io/upload_images/1956963-005d0661736effc1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 第2章 微服务介绍
#### 2-1 微服务和其他常见架构
- 什么是微服务？
> In short, the microservice architectural style is an approach to developing a single application as **a suite of small services**, each **running in its own process** and communicating with lightweight mechanisms, often an HTTP resource API. These services are **built around business capabilities** and **independently deployable** by fully automated deployment machinery. There is **a bare minimum of centralized management of these services**, which may be written in different programming languages and use different data storage technologies.

**微服务是一种架构风格**
一系列微小的服务共同组成
运行在自己的进程里
每个服务为独立的业务开发
独立部署
分布式管理

![image.png](https://upload-images.jianshu.io/upload_images/1956963-64510f2df4fa4386.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 单体架构的优点
容易测试、容易部署
- 单体架构的缺点
开发效率低、代码维护难、部署不灵活、稳定性不高、扩展性不够
- 分布式定义
旨在支持应用程序和服务的开发，可以利用物理架构由**多个自治的处理元素**，**不共享主内存**，但**通过**网络发送**消息**合作。

#### 2-2 从一个极简的微服务架构开始
![image.png](https://upload-images.jianshu.io/upload_images/1956963-dca53ebdfd3fa75e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 微服务架构的基础框架/组件
服务注册发现
服务网关
后端通用服务
前端服务 对后端通用服务聚合和裁剪后暴露给外部
- Spring Cloud是什么?
Spring Cloud是一个开发工具集，含了多个子项目
-- 利用Spring Boot的开发便利
-- 主要是基于Netflix开源组件的进一步封装
Spring Cloud **简化了分布式开发**
**掌握如何使用，更要理解分布式、架构的特点**

## 第3章 服务注册与发现
#### 3-1 Spring Cloud Eureka
- 基于Netflix Eureka做了二次封装
- 两个组件组成：
Eureka Server 注册中心 提供服务的注册与发现
Eureka Client 服务注册

#### 3-2 Eureka Server
```
IDEA旗舰版	
IDEA 2018 LICENSE SERVER
http://idea.congm.in
```
```
New Project选择Spring Initializr
com.lh eureka
Dependencies选择Cloud Discovery选择Eureka Server 
配置Spring Boot和Spring Cloud版本
启动项目访问 http://localhost:8080
配置自己注册到自己
eureka ： mvn clean package打包
java -jar eureka.jar
或者后台执行 nohup java -jar target/eureka.jar > /dev/null 2>&1 &
ps -ef | grep eureka
停止：kill -9 4681
```
代码和配置 
```
@SpringBootApplication
//触发Spring Boot的自动配置机制
@EnableEurekaServer
public class EurekaApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaApplication.class, args);
	}

}
```
application.yml
```
eureka:
  client:
    service-url:
      # 指定eureka服务器的地址
      defaultZone: http://localhost:8761/eureka/
    # 表示是否自己注册到Eureka server 默认为true
    register-with-eureka: false
  server:
    # 注册中心关闭自我保护机制，修改检查失效服务的时间，测试的时候关闭
    enable-self-preservation: false
spring:
  application:
    # 指定进行服务注册时该服务的名称
    name: eureka
server:
  # eureka服务器端口号
  port: 8761
```

#### 3-3 Eureka Client的使用
```
New Project选择Spring Initializr
com.lh client
Dependencies选择Cloud Discovery选择Eureka Discovery
配置Spring Boot和Spring Cloud版本，启动
```
代码和配置
```
@SpringBootApplication
//触发Spring Boot的自动配置机制
@EnableDiscoveryClient
public class ClientApplication {

	public static void main(String[] args) {
		SpringApplication.run(ClientApplication.class, args);
	}

}
```
application.yml
```
eureka:
  client:
    service-url:
      # 指定eureka服务器的地址
      defaultZone: http://localhost:8761/eureka/ # ,http://localhost:8762/eureka/,http://localhost:8763/eureka/
#  设置主机名
#  instance:
#    hostname: clientName
spring:
  application:
    # 指定进行服务注册时该服务的名称
    name: client
```

#### 3-4 Eureka的高可用
- @EnableEurekaServer @EnableDiscoveryClient
- 心跳检测、健康检查、负载均衡等功能
- Eureka的高可用，生产上建议至少两台以上
- **分布式系统中，服务注册中心是最重要的基础部分**


```
两个Eureka 互相注册
启动配置：VM options:  -Dserver.port=8761
启动配置：VM options:  -Dserver.port=8762
修改defaultZone 1注册到2上，2注册到1上

client只注册到8761但是会自动注册到8762上
client最好配置两个service地址实现高可用

三个Eureka 需要两两注册，Client配置三个地址
```

#### 3-5 Eureka总结
- @EnableEurekaServer、@EnableEurekaClient
- 心跳检测、健康检查、负载均衡等功能
- Eureka的高可用，生产上建议至少两台以上
- **分布式系统中，服务注册中心是最重要的基础部分**

#### 3-6 分布式下服务注册的地位和原理
- 分布式系统中为什么需要服务发现？
- 服务发现的两种方式
客户端发现：Eureka
服务端发现：Nginx、Zookeeper、k8s，需要代理
- 微服务的特点：异构
不同语言
不同类型的数据库
- Spring Cloud的服务调用方式
REST or RPC
Node.js的eureka-js-client

**理解分布式、架构的特点、原理更重要！！！**

## 第4章 服务拆分
#### 4-1 微服务拆分的起点
- 如何拆分？
**先明白起点和终点**
起点：既有架构的形态
终点：好的架构不是设计出来的，而是进化而来的，一直在演进ing
**需要考虑的因素与坚持的原则**
- 适合上微服务么？
业务形态不适合的：
系统中包含很多强事务场景的
业务相对稳定，迭代周期长
访问压力不大，可用性要求不高
......

#### 4-2 康威定律和微服务
- 微服务架构的理论基础：康威定律
>Organizations which design systems are constrained to produce designs which are copies of the communication structures of these organizations. 

任何组织在设计一套系统时，所交付的设计方案在结构上都与该组织的沟通结构保持一致。
简单来时就是：沟通的问题会影响系统的设计。
- 微服务和团队结构

#### 4-3 点餐业务服务拆分分析
- 服务拆分
业务量小： PC端和手机端
业务量大：订单、商品、支付
- 服务拆分的方法论
扩展立方模型:X轴 水平复制（负载均衡运行多个副本）、Z轴 数据分区（每个服务器负责一个数据子集）、Y轴功能解耦（不同职责分成不同服务）
- 如何拆功能
**单一职责、松耦合、高内聚**
关注点分离： 按职责 按通用性 按粒度级别
- 服务和数据的关系
先考虑业务功能，再考虑数据
无状态服务

#### 4-4 商品服务API和SQL介绍
- 商品列表接口 GET /product/list
- 类目表product_category、商品表product_info
- SpringCloud_Sell.sql 见Github


#### 4-5 商品服务编码
```
New Project选择Spring Initializr
com.lh product
Dependencies选择Cloud Discovery选择Eureka Discovery
配置Spring Boot和Spring Cloud版本
application.yml 注册到eureka
spring-data-jpa、mysql、lombok依赖引入和配置，实体类，dao，go to test 测试
ProductService 查询所有在架商品列表，go to test 测试
新建ResultVO、ProductVO对象
ProductController
google浏览器安装JsonView插件，火狐浏览器自带。
代码比较简单就不贴了，具体见github
```
#### 4-8 订单服务API和SQL介绍
- 创建订单接口 POST /order/create
- 订单order_master、订单商品order_detail

#### 4-9 订单服务编码
```
New Project选择Spring Initializr
com.lh order
Dependencies选择Cloud Discovery选择Eureka Discovery
配置Spring Boot和Spring Cloud版本
application.yml 注册到eureka
spring-data-jpa 、mysql、lombok依赖引入和配置，实体类，dao
新建OrderDTO数据传输对象
OrderService 订单入库
新建OrderForm对象
OrderController 参数校验、创建订单，postman测试
代码比较简单就不贴了，具体见github
```
#### 4-12 再看“拆数据”
- 每个微服务都有单独的数据存储
- 依据服务特点选择不同结构的数据库类型
- **难点在确定边界**
- 针对边界设计API
- 依据边界权衡数据冗余

## 第5章 应用通信
#### 5-1 HTTP vs RPC
- 应用间通信：HTTP VS RPC
Spring Cloud
Dubbo
- Spring Cloud 中服务间两种restful调用方式
RestTemplate
feign

#### 5-2 RestTemplate的三种使用方式
-  RestTemplate
传统情况下在Java代码里访问RESTful服务，一般使用Apache的HttpClient。不过此种方法使用起来太过繁琐。Spring提供了一种简单便捷的模板类来进行操作，这就是RestTemplate。
```
RestTemplate的三种使用方式的代码见order项目中的ClientController、RestTemplateConfig
```

#### 5-3 客户端负载均衡器：Ribbon
RestTemplate、Feign 和 Zuul 都使用到了Ribbon的注解：@LoadBalanced，添加此注解后，Ribbon会通过LoadBalancerClient自动基于某种规则连接目标服务，从而实现负载均衡
- Ribbon核心
服务发现、服务选择规则、服务监听
- Ribbon主要组件
ServerList、IRule、ServerListFilter

#### 5-4 追踪源码自定义负载均衡策略
Ctrl+Alt点击方法可以直接跳转到实现类
Show Diagram可以查看类结构（Intellij 社区版不支持Diagrams）
断点查看**获取服务列表**的地方BaseLoadBalancer.getAllServers()
断点查看**负载均衡策略** 轮询的方式 DynamicServerListLoadBalanced构造函数，DynamicServerListLoadBalanced是启动日志上找到的
```
日志
2018-10-31 18:04:53.133  INFO 8624 --- [nio-9088-exec-2] c.n.l.DynamicServerListLoadBalancer      : DynamicServerListLoadBalancer for client PRODUCT initialized
```
可以使用配置修改负载均衡策略为随机规则，测试
```
PRODUCT: # 应用名称
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

#### 5-5 Feign的使用
- 订单服务调用商品服务，使用Feign来进行应用间通信
```
添加依赖和注解，定义好调用哪些接口 

<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>

@EnableFeignClients

// client包ProductClient，感觉类似retrofit
@FeignClient(name = "product")
public interface ProductClient {

    @GetMapping("/msg")
    String productMsg();

}

// 直接使用
@Autowired
private ProductClient productClient;
```
```
启动报错，我发现把mvn repository中的open feign全删了就好了，不知道什么原因。
也看了别人的一些回答：https://blog.csdn.net/jiadajing267/article/details/81040808
```
- Feign的定义
声明式REST客户端（伪RPC）
采用了基于接口的注解
Feign内部使用了Ribbon作为负载均衡

#### 5-6 获取商品列表(Feign)
```
商品服务提供 /product/listForOrder 接口给订单服务提供获取商品列表的方法
订单服务通过feign调用该接口
测试订单服务 /getProductList 接口
代码见github，很简单
```

#### 5-7 扣库存(Feign)
```
商品服务提供 /product/decreaseStock 接口给订单服务提供扣库存的方法
订单服务通过feign调用该接口
测试订单服务 /decreaseStock 接口
代码见github，很简单
```

#### 5-8 整合接口打通下单流程(Feign)
```
OrderServiceImpl完成下单业务，测试
业务代码不需要太多关心
```

#### 5-9 项目改造成多模块
```
product项目不能把自己的表映射的实体类暴露出去
部分对象重复定义了
订单服务定义了商品服务接口，不合理，需要写到商品

多模块
product项目拆成product-server(所有的业务逻辑)、product-client(对外暴露的接口)和product-common(公用的对象)
商品服务和订单服务都需要改成多模块

@EnableFeignClients(basePackages = "com.lh.product.client")

mvn -Dmaven.test.skip=true -U clean install # 安装product项目
order-server需要引入product-client项目
```

#### 5-11 RabbitMQ的安装
```
docker run -d --hostname my-rabbit -p 5672:5672 -p 15672:15672 rabbitmq:3.7.3-management
docker ps 
访问: localhost:15672
```

#### 5-12 微服务，Docker和DevOps
- 微服务和容器：天生一对
从系统环境开始，自底至上打包应用
轻量级，对资源的有效隔离和管理
可复用，版本化

#### Tip
```
SecretCRT连接MySQL乱码需要设置MySQL连接编码： --default-character-set=utf8
```

