## 第6章 统一配置中心
#### 6-1 统一配置中心概述
- 为什么需要统一配置中心？
不方便维护
配置内容安全与权限
更新配置项目需重启
![image.png](https://upload-images.jianshu.io/upload_images/1956963-e9911943acb8937a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 6-2 Config Server
```
New Project选择Spring Initializr
com.lh config
Dependencies选择Cloud Discovery选择Eureka Discovery，还需要选择Cloud Config下Config Server
配置Spring Boot和Spring Cloud版本
application.yml 注册到eureka
@EnableConfigServer
github上新建spring-cloud-study-config-repo存放配置文件order.yml
配置文件配置git仓库地址
访问：localhost:8080/order-a.yml 可以查看配置

配置格式：/{name}-{profiles}.yml   /{label}/{name}-{profiles}.yml
name：服务名
profiles：环境
label：分支（branch）
测试order-dev.yml和order-release.yml
还可以测试分支，比较简单，生产上一般只是用master分支
项目启动日志可以查看到本地git存放位置，也可以在配置文件中设置
```

#### 6-3 Config Client
```
order项目引入spring-cloud-config-client依赖
添加配置bootstrap.yml
EnvController检验是否拿到配置

统一配置中心的高可用很简单，直接启动多个
order和product项目会随机访问config服务获取配置
注意：eureka配置需要放在bootstrap.yml中

通用配置放在order.yml，不同的配置写在order-dev.yml或者order-rel.yml
到目前为止没有完成我们的目标：配置动态刷新
```

#### 6-4 Spring Cloud Bus自动更新配置理论
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-bfb6aae6cdbf4ad1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
git上的配置变更，远端git通过Webhooks功能访问/bus/refresh接口，把配置更新信息发送到RabbitMQ上

#### 6-5 Spring Cloud Bus实操
```
修改版本为BUILD-SNAPSHOT版本
引入spring-cloud-starter-bus-amqp依赖
启动config项目，rabbitmq management可以看到多了一个springCloudBus.anonymous队列

同样order项目修改版本、引入依赖，feign升级为openfeign，启动项目
rabbitmq management可以看到又多了一个队列

暴露bus-refresh接口，修改config配置：management.endpoints.web.exposure.include: *，重启项目
github上修改配置，post访问config接口：http://localhost:8080/actuator/bus-refresh
curl -v -X POST "http://localhost:8080/actuator/bus-refresh"
rabbitmq management可以看到队列中发送过一条消息

order日志可以看到配置刷新了，但是接口访问没刷新
使用配置的地方新增注解：@RefreshScope

修改github配置，调用接口，访问order env接口可以发现配置修改成功
实际使用：建议使用GirlConfig、GirlController这种方式
配置WebHooks，实现自动调用接口，需要使用内网穿透natapp.cn
```

#### 6-6 集成WebHooks实现动态更新
```
github Webhooks
config提供了一个用于Webhooks的路由:http://xxx/monitor
测试有点小bug，不耽误进度，先遗留吧
```

## 第7章 消息和异步
#### 7-1 异步和消息
**异步：**客户端请求不会阻塞进程，服务端的响应可以是非即时的
**异步的常见形态：**通知、请求/异步响应、消息
MQ应用场景：异步处理、流量削峰、日志处理和应用解耦

#### 7-2 RabbitMQ的基本使用
```
修改版本M3 M2,引入spring-boot-starter-amqp依赖，github上修改配置
编写MQReceiver接收MQ消息，在MqSenderTest发送消息，测试
自动创建队列
自动创建队列，Exchange和Queue绑定
MQReceiver设置两个Consume分别接收不同Exchange发来的消息
```

#### 7-4 Spring Cloud Stream的使用
```
引入spring-cloud-starter-stream-rabbit依赖,配置可以直接使用rabbitmq的
StreamClient、StreamReceiver、SendMessageController

消息分组
现在启动两个实例，都会收到消息，现在想只让一个实例接收消息
配置spring.cloud.stream.bindings.myMessage.group=order分组

测试传递对象OrderDTO

希望在管理台上能够查看消息内容，而不是看到base64编码后的内容
配置spring.cloud.stream.bindings.myMessage.content-type=application/json

消费者希望对消息有个回应，可以使用@SendTo注解，需要消费者消费
```

#### 7-6 商品和订单服务中使用MQ
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-da8c26143ef886a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
商品服务有库存变化都发送一条消息到消息队列，订单服务监听库存变化，记录到redis
```
product接入到配置中心，添加依赖，修改bootstrap.yml
product添加amqp依赖，配置mq
ProductServiceImpl 扣库存发送mq消息，
postman调用扣库存接口，数据库确认数据，rabbitmq manager 查看mq message
扣库存接口：http://localhost:8088/product/decreaseStock，数据：[{"productId": "157875196366160022","productQuantity": "1"}]

order服务接收消息，ProductInfoReceiver，postman调用扣库存接口，测试

docker run -d -p 6379:6379 redis:4.0.8
docker ps 
可以通过Redis Desktop Manager连接到redis
order服务添加spring-boot-starter-data-redis依赖
bootstrap.yml编写redis配置，然后放到github上
order服务把数据存储到redis中，测试

注意：需要在完成整个购物车扣库存逻辑之后，再发消息，整理代码
```

#### 7-9 异步扣库存分析
- 原始流程
1. 查询商品信息（调用商品服务）
2. 计算总价（生成订单详情）
3. 商品服务扣库存（调用商品服务）
4. 订单入库（生成订单）
- 秒杀场景
1. 库存在Redis中保存
2. 收到请求Redis判断是否库存充足，减掉Redis中库存
3. 订单服务创建订单写入数据库，并发送消息
4. 订单入库异常需要手动回滚Redis
- 需要考虑到的事情
1. 可靠的消息投递
2. 用户体验的变化

## 八、服务网关
#### 8-1 服务网关和Zuul
- 为什么需要网关服务？
统一入口
- 服务网关的要素
稳定性，高可用
性能、并发性
安全性
扩展性
- 常用的网关方案
Nginx + Lua
Kong 商业软件，付费
Tyk 开源轻量级网关，Go语言开发
Spring Cloud Zuul
- Zuul的特点
路由 + 过滤器 = Zuul
核心是一系列的过滤器
- Zuul的四种过滤器API
前置（Pre）、路由（Route）、后置（Post）、错误（Error）

#### 8-2 Zuul：路由转发，排除和自定义
```
New Project选择Spring Initializr
com.lh api-gateway
Dependencies选择Cloud Discovery选择Eureka Discovery，Cloud Config下Config Client，还需要选择Cloud Routing下Zuul
配置Spring Boot M3和Spring Cloud M2版本
bootstrap.yml，@EnableZuulProxy，启动

希望通过网关访问到product服务的product/list接口
localhost:9000/product/product/list # 第一个product是指product服务

如果想自定义路由：
需要在api-gateway服务使用zuul配置：
zuul.routes.myProduct.path=/myProduct/**
zuul.routes.myProduct.serviceId=product
简洁写法：
zuul.routes.product=/myProduct/**

localhost:9000/application/routes 可以查看所有路由
需要增加配置：management.security.enabled=false

禁止某个路由：
zuul:
  ignored-patterns:
    - /**/product/listForOrder
```

#### 8-3 Zuul：Cookie和动态路由
配置：zuul.routes.myProduct.sensitiveHeaders= # 敏感头过滤设置为空，否则无法获取cookie
动态路由只需要把配置放到github上，然后编写ZuulConfig，完成配置的动态注入
```
@Component
public class ZuulConfig {

    @ConfigurationProperties("zuul")
    @RefreshScope
    public ZuulProperties zuulProperties() {
        return new ZuulProperties();
    }
    
}
```

#### 8-4 Zuul：路由和高可用小结
前置（Pre）：限流、鉴权
后置（Post）：统计、日志
- Zuul的高可用
多个Zuul节点注册到Eureka Server
- Nginx 和 Zuul 混搭

## 九、Zuul综合使用
#### 9-1 Zuul：Pre和Post过滤器
```
实现访问任意接口都需要带token参数
TokenFilter 
AddResponseHeaderFilter
```

#### 9-2 Zuul：限流
时机：请求被转发之前调用
令牌桶限流方案
![image.png](https://upload-images.jianshu.io/upload_images/1956963-c269ff89b3521b4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
RateLimitFilter 限流拦截器
```

#### 9-3 Zuul鉴权和添加用户服务
/order/create 只能买家访问
/order/finish 只能卖家访问
/product/list 都可访问
```
AuthFilter 权限拦截，区分买家和卖家
见API.md 买家登录和卖家登录

New Project选择Spring Initializr
com.lh user
Dependencies选择Cloud Discovery选择Eureka Discovery，Cloud Config下Config Client，还需要选择MySQL、JPA、Redis
配置Spring Boot M3和Spring Cloud M2版本
bootstrap.yml、github配置user.yml，启动项目
user 需要改成多模块，对外提供接口
Entity定义，repository和service编写
```

#### 9-4 模拟买家卖家登录功能实现
```
LoginController 登录功能编写
```

#### 9-6 完结订单接口开发
/order/finish 接口开发，测试

#### 9-7 完成权限校验
- 在前置过滤器中实现相关逻辑
```
AuthFilter 编写，只能买家访问或者只能卖家访问
注意api-gateway 敏感头设置，排除所有的敏感头
zuul.sensitiveHeaders: # 全部服务忽略敏感头

对买家和卖家分别建Filter，拆分AuthFilter
```

#### 9-9 Zuul：跨域
- 在被调用的类或方法上增加@CrossOrigin注解
- 在Zuul里增加CorsFilter过滤器
```
CorsConfig
```
![image.png](https://upload-images.jianshu.io/upload_images/1956963-fed34e17e959db59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 十、服务容错
#### 10-1 服务容错和Hystrix
- 雪崩效应
- 防雪崩利器——Spring Cloud Hystrix
- 基于Netflix对应的Hystrix
服务降级、依赖隔离、服务熔断、监控等功能
- 服务降级
优先核心服务，非核心服务不可用或弱可用
通过HystrixCommand注解指定
fallbackMethod（回退函数）中具体实现降级逻辑

#### 10-2 触发降级
```
order HystrixController 
引入依赖spring-cloud-starter-hystrix
启动类加注解@EnableCircuitBreaker
注：@SpringCloudApplication注解包含三个注解
@HystrixCommand
```
![image.png](https://upload-images.jianshu.io/upload_images/1956963-0cf38e1f20981954.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 10-3 超时设置
```
@DefaultProperties
product 接口休眠2秒，order接口也会触发降级
需要配置超时时间
```
![image.png](https://upload-images.jianshu.io/upload_images/1956963-f3589b379e57bb1c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-219e89a0beb8e91e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 10-4 探讨断路器模式
- **依赖隔离**
线程池隔离
Hystrix自动实现了依赖隔离
- 熔断
![image.png](https://upload-images.jianshu.io/upload_images/1956963-39b79a0167f5932c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Circuit Breaker：断路器

#### 10-5 使用配置项
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-3527d495e1ca487a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
超时和熔断配置都可以写到bootstrap.yml，然后放入统一配置中心
然后还需要配合注解@HystrixCommand
使用commandKey 可以设置特殊值
```

#### 10-6 feign-hystrix的使用
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-7f5efffa0594c07c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
配置feign.hystrix.enabled=true
@FeignClient中有一个fallback可以配置服务降级方法，断点调试
order 服务还需要配置扫描到product 服务中的Feign接口，否则无法识别到ProductClientFallback

#### 10-7 hystrix-dashboard
```
可视化组件，order服务添加依赖spring-cloud-starter-hystrix-dashboard
order服务启动类增加注解@EnableHystrixDashboard
访问localhost:8081/hystrix，填入单个Hystrix App：http://localhost:8081/hystrix.stream   1000ms  Order
注意：order配置：management.context-path=/  否则hystrix.stream有前缀application
可以看到可视化页面
postman 有功能可以在一定时间发送很多请求，保存新建一个collection，run，配置发送100次请求，观察指标
```

## 十一、服务跟踪
#### 11-1 服务追踪（上）
- 链路监控
Spring Cloud Sleuth
Zipkin
```
order项目引入spring-cloud-starter-sleuth依赖
下单接口被调用，sleuth会打印日志
日志级别配置：
logging.level.org.springframework.cloud.netflix.feign=debug
product服务也加上sleuth，调整日志级别

使用zipkin可视化页面查看
docker run -d 9411:9411 openzipkin/zipkin
访问localhost:9411
数据如何与页面整合呢？
需要添加依赖spring-cloud-sleuth-zipkin
spring-cloud-starter-zipkin依赖包含了上面这两个，可以注释掉上面两个依赖
bootstrap.yml配置zipkin：spring.zipkin.base-url=http://localhost:9411
spring.sleuth.sampler.percentage=1 # 百分比，抽样观察，测试时设置为1
调用接口，localhost:9411，可以发现链路追踪有数据，查看接口耗时
```

#### 11-2 服务追踪（下）
- 分布式追踪系统
核心步骤：数据采集、数据存储、查询展示
OpenTracing 规范
- Annotation 事件类型
cs：客户端发起请求的时间
cr：客户端收到处理完请求的时间
ss：服务端处理完逻辑的时间
sr：服务端收到调用端请求的时间
客户端调用时间=cr-cs
服务端处理时间=sr-ss
- ZipKin 关键概念
traceId
spanId
parentId

## 十二、容器部署
#### 12-1 运行第一个docker容器
```
eureka 服务 Dockerfile 编写
可以在网易云镜像仓库找java镜像，官方镜像比较慢
编写好Dockerfile，构建镜像
mvn clean package -Dmaven.test.skip=true # 构建应用
docker build -t seckill44/eureka .  # 构建镜像
docker run -p 8761:8761 -d seckill44/eureka # 完成
可以上传镜像到镜像仓库中
```

#### 12-2 rancher安装
[Rancher 中文站点](https://www.cnrancher.com/)
Rancher是开源的企业级容器管理平台，更方便管理Docker
```
Download Rancher：
sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher
Rancher 2.0 暂时不使用
下载Rancher太慢可以使用aliyun镜像加速器，重启docker：systemctl restart docker
访问192.168.30.77:8080 —— Rancher 页面
首先添加主机Custom，
rancher-server 和 rancher-agent 通常不放在同一台服务器上
rancher-agent 安装docker并启动，安装Rancher Agent
基础架构，主机页面可以查看到Agent
Rancher商店可以启动mysql很方便
```

#### 12-3 部署eureka和config
```
config DockerFile拷贝 eureka DockerFile 修改端口
mvn clean package -Dmaven.test.skip=true # 构建应用
docker build -t seckill44/config .  # 构建镜像
推送config镜像到网易云镜像中心：docker tag 镜像id hub.c.163.com/seckill44/config
docker push hub.c.163.com/seckill44/config
以上操作可以写成脚本 build.sh
bash build.sh
打开Rancher页面，用户，添加应用SpringCloudProject
添加服务，名称eureka，镜像地址填写，添加端口映射8761-8761，创建
添加服务，名称config，注意配置修改eureka地址为http://eureka:8761/eureka，镜像地址填写，添加端口映射8080-8080，创建，查看启动日志，eureka查看是否注册上来
```

#### 12-4 构建eureka高可用服务
```
application-eureka1.yml  server.port=8761 注册到8762上
application-eureka2.yml  server.port=8762 注册到8761上
application.yml : spring.profiles.active=eureka1
mvn clean package -Dmaven.test.skip=true # 构建应用
java -jar -Dspring.profiles.active=eureka1 target/*.jar
java -jar -Dspring.profiles.active=eureka2 target/*.jar
访问localhost:8761  localhost:8762

修改配置localhost 改为eureka
bash build.sh，构建镜像
删除eureka，添加服务eureka1，填写镜像和端口8761，配置环境变量spring.profiles.active=eureka1
添加服务eureka2，填写镜像和端口8762，配置环境变量spring.profiles.active=eureka2
config 需要修改注册地址到eureka1:8761和eureka2:8762，重新构建发布启动，可以升级，升级完成
```

#### 12-5 构建product服务
```
product DockerFile拷贝 修改端口，修改eureka端口
运行 bash build.sh # 构建应用、构建镜像、推送镜像
镜像权限设置为公开
打开Rancher页面，
添加服务，名称product，镜像地址填写，端口映射不需要，创建
------------------
查看启动日志，发现不能从配置中心获取配置，日志访问地址有问题
config注册到eureka的地址很乱，之前我也就发现了
修改config配置：eureka.instance.prefer-ip-address=true，重新bash build.sh 构建，升级config
刷新eureka可以发现鼠标放在config容器左下角显示ip+端口，可以访问
product 也加上eureka.instance.prefer-ip-address=true，重新构建升级，成功启动
升级，修改端口，测试，访问接口
注意数据库，redis地址都需要是外部地址
```

#### 12-6 构建order服务
```
order 配置也加上eureka.instance.prefer-ip-address=true，修改zipkin地址
order DockerFile拷贝 修改端口，修改eureka端口
运行 bash build.sh # 构建应用、构建镜像、推送镜像
镜像权限设置为公开
打开Rancher页面，
添加服务，名称order，镜像地址填写，端口映射不需要，创建
测试下单接口
```

#### 12-7 构建api-gateway
```
api-gateway 配置也加上eureka.instance.prefer-ip-address=true
api-gateway DockerFile拷贝 修改端口，修改eureka端口
运行 bash build.sh # 构建应用、构建镜像、推送镜像
镜像权限设置为公开
打开Rancher页面，
添加服务，名称order，镜像地址填写，端口映射不需要，创建
添加服务负载均衡，名称lb-api-gateway，端口80，选择api-gateway 8080，创建
lb-api-gateway服务 访问192.168.30.154/product/product/list
负载均衡器80端口会转发到api-gateway容器8080端口，再转发到product服务
好处：api-gateway 容易扩容，实现了gateway的高可用
```

## 十三、版本升级
#### 13-1 升级介绍&eureka
- Spring Cloud 正式版
GA：general availability 面向大众的可用版本
M：milestone 里程碑版本
SNAPSHOT：快照，代码可变

- 升级eureka
手记：[SpringCloud升级到Finchley.RELEASE](https://www.imooc.com/article/41648)
```
Spring Boot版本选择：2.0.2.RELEASE
Spring Cloud版本选择：Finchley.RELEASE
仓库地址可以删除
阿里云镜像日志修改可以让下载速度变快
```

#### 13-2 升级config&product&api-gateway
- 升级config
```
Spring Boot版本选择：2.0.2.RELEASE
Spring Cloud版本选择：Finchley.RELEASE
删除仓库地址
启动检查有没有注册到eureka，访问配置测试
```
- 升级product
```
Spring Boot版本选择：2.0.2.RELEASE
Spring Cloud版本选择：Finchley.RELEASE
删除仓库地址
启动失败
需要添加spring-boot-starter-web依赖，正常启动，由于多了webflux可供选择
访问接口测试：localhost:8005/product/list
maven clean install -Dmaven.test.skip=true -U，feign报错
feign 已经变成了 openfeign，修改依赖
maven clean install -Dmaven.test.skip=true -U，成功
```
- 升级api-gateway
```
Spring Boot版本选择：2.0.2.RELEASE
Spring Cloud版本选择：Finchley.RELEASE
删除仓库地址
启动成功，通过api-gateway访问product服务成功
注意：zuul需要设置超时时间
查看官方文档Zuul Timeouts 配置：
ribbon.ReadTimeout=5000
ribbon.SocketTimeout=5000
```

#### 13-4 升级order
- 升级config
```
Spring Boot版本选择：2.0.2.RELEASE
Spring Cloud版本选择：Finchley.RELEASE
feign 依赖修改成 openfeign
hystrix 依赖修改成 netflix-hystrix
添加spring-boot-starter-web
删除仓库地址
启动项目
报错StreamClient INPUT和OUTPUT不能定义成一样 修改
启动项目，测试下单接口，失败
通过feign调用接口需要处理超时：
查看官方文档Feign Timeouts 配置：
feign:
  client:
    config:
      default:
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: basic
启动项目，测试下单接口，成功
配置日志级别包路径修改
测试HystrixController，服务降级没问题
sleuth+zipkin 链路监控测试，先启动访问zipkin，测试下单接口，查看日志
spring.sleuth.sampler.probability=1  参数名修改了
zipkin 查看数据为空，需要配置spring.zipkin.sender.type=web，项目中有rabbitmq需要此配置，默认就是http
zipkin 查看数据已经有了
```

#### 13-5 升级配置自动刷新
- 配置的动态刷新
```
order 项目添加spring-cloud-starter-bus-amqp依赖
localhost:15672 查看队列已经存在
配置好内网穿透和Webhooks
Test Push Events 可以查看到日志，说明Webhooks可以请求到
从mq图上可知，消息传递到mq没问题，order服务接收到消息有问题
增加日志配置：logging.level.org.springframework.cloud.bus=dehug
可以查看到order服务收到了消息
debug模式修改DefaultBusPathMatcher match中path对象的值，可以刷新配置
git clone spring-cloud-bus 查看源码
git reset --hard 2.0.0.RELEASE这个节点
idea引入spring-cloud-bus项目
BusEnvironmentPostProcessor#getDefaultServiceId
修改上面这个方法，可以刷新配置
编写GirlConfig和GirlController测试
```
配置自动刷新功能在生产上暂时不使用，等官方出新版本
