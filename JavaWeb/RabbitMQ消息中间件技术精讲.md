## 第1章 课程介绍
#### 1-3 业界主流消息中间件介绍
ActiveMq 性能一般
Kafka 高性能，对消息的重复、丢失、错误没有严格要求，适合日志收集
RocketMQ 高性能，高可靠。商业版收费 
RabbitMQ 高可用

## 第2章 低门槛，入门RabbitMQ核心概念
#### 2-2 哪些互联网大厂在使用RabbitMQ,为什么？
- 滴滴、美团、头条、去哪儿
- 开源、性能优秀、稳定性保障
- 提供可靠性消息投递模式（confirm）、返回模式（return）
- 与SpringAMQP完美的整合、API丰富
- 集群模式丰富，表达式配置，HA模式，镜像队列模型
- 保证数据不丢失的前提做到高可靠性、可用性

#### 2-3 RabbitMQ高性能的原因
- Erlang语言最初在于交换机领域的架构模式，这样使得RabbitMQ在Broker之间进行数据交互的性能是非常优秀的
- Erlang的优点：**Erlang有着和原生Socket一样的延迟**

#### 2-4 AMQP高级消息队列协议与模型
- AMQP全称：Advanced Message Queuing Protocol
- AMQP翻译：高级消息队列协议
- AMQP定义：是具有现代特征的二进制协议。是一个提供统一消息服务的应用层标准高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。
- AMQP协议模型
![image.png](https://upload-images.jianshu.io/upload_images/1956963-2002ca730ac5d9de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2-5 AMQP核心概念讲解
- Server：又称Broker，接收客户端的连接，实现AMQP实体服务
- Connection：连接，应用程序与Broker的网络连接
- Channel：网络信道，几乎所有的操作都在Channel中进行，Channel是进行消息读写的通道。客户端可建立多个Channel，每个Channel代表一个会话任务。
- Message：消息，服务器与应用程序之间传送的数据，由Properties和Body组成。Properties可以对消息进行修饰，比如消息的优先级、延迟等高级特性；Body则就是消息体内容。
- Virtual host：虚拟主机，用于进行逻辑隔离，最上层的消息路由。一个Virtual host里面可以有若干个Exchange和Queue，同一个Virtual Host里面不能有相同名称的Exchange和Queue
- Exchange：交换机，接收消息，根据路由键转发消息到绑定的队列
- Binding：Exchange和Queue之间的虚拟连接，binding中可以包含routing key
- Routing key：一个路由规则，虚拟机可用它来确定如何路由一个特定消息
- Queue：也称为Message Queue，消息队列，保存消息并将它们转发给消费者

#### 2-6 RabbitMQ整体架构与消息流转
- 整体架构
![image.png](https://upload-images.jianshu.io/upload_images/1956963-efe21c0c81ffe9ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 消息流转图
![image.png](https://upload-images.jianshu.io/upload_images/1956963-dfb107abf76ed562.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2-7 RabbitMQ环境安装
- 官网地址：http://www.rabbitmq.com
- 提前准备：安装Linux必要依赖包
vim /etc/hostname
- 下载RabbitMQ必须安装包
可以下载rpm包，一键安装
RabbitMQ 和 Erlang版本需要匹配
rpm -ivh erlang.rpm
rpm -ivh socat.rpm
rpm -ivh rabbitmq-server-3.6.5-1.noarch.rpm
- 配置文件修改
- 服务的启动：rabbitmq-server start &  //后台启动
- 服务的停止：rabbitmqctl stop_app
- 管理插件：rabbitmq-plugins enable rabbitmq_management
- 访问地址：http://192.168.10.127:15672/
```
ps -ef | grep rabbit
lsof -i:5672 # 看到rabbitmq就证明启动OK
rabbitmq-plugins list # 默认安装插件
```
```
准备：
yum install 
build-essential openssl openssl-devel unixODBC unixODBC-devel 
make gcc gcc-c++ kernel-devel m4 ncurses-devel tk tc xz

下载：
wget www.rabbitmq.com/releases/erlang/erlang-18.3-1.el7.centos.x86_64.rpm
wget http://repo.iotti.biz/CentOS/7/x86_64/socat-1.7.3.2-5.el7.lux.x86_64.rpm
wget www.rabbitmq.com/releases/rabbitmq-server/v3.6.5/rabbitmq-server-3.6.5-1.noarch.rpm

配置文件：
vim /usr/lib/rabbitmq/lib/rabbitmq_server-3.6.5/ebin/rabbit.app
比如修改密码、配置等等，例如：loopback_users 中的 <<"guest">>,只保留guest
服务启动和停止：
启动 rabbitmq-server start &
停止 rabbitmqctl stop_app

管理插件：rabbitmq-plugins enable rabbitmq_management
访问地址：http://192.168.11.76:15672/
```

#### 2-9 命令行与管理台结合讲解
rabbitmqctl 命令最丰富
简单操作
- rabbitmqctl stop_app：关闭应用
- rabbitmqctl start_app：启动应用
- rabbitmqctl status：节点状态
- rabbitmqctl add_user username password：添加用户
- rabbitmqctl list_users：列出所有用户
- rabbitmqctl delete_user username：删除用户
- rabbitmqctl clear_permissions -p vhostpath username：清除用户权限
- rabbitmqctl list_user_permissions username：列出用户权限
- rabbitmqctl change_password username newpassword：修改密码
- rabbitmqctl set_permissions -p vhostpath username "." "." "." 设置用户权限
- rabbitmqctl add_vhost vhostpath：创建虚拟主机
- rabbitmqctl list_vhosts：列出所有虚拟主机
- rabbitmqctl list_permissions -p vhostpath：列出虚拟主机上所有权限
- rabbitmqctl delete_vhost vhostpath：删除虚拟主机
- rabbitmqctl list_queues：查看所有队列信息
- rabbitmqctl -p vhostpath purge_queue blue：清除队列里的消息
高级操作
- rabbitmqctl reset：移除所有数据，要在rabbitmqctl stop_app之后使用
- rabbitmqctl join_cluster <clusternode> [--ram]：组成集群命令
- rabbitmqctl cluster_status：查看集群状态
- rabbitmqctl change_cluster_node_type disc|ram：修改集群节点的存储形式
- rabbitmqctl forget_cluster_node [--offline]：忘记节点（摘除节点）
- rabbitmqctl rename_cluster_node oldnode1 newnode1：修改节点名称

```
lsof -i:5672 # 查看rabbitmq是否启动成功
rabbitmqctl list_queues # 查看所有队列信息
rabbitmqctl list_vhosts # 列出所有虚拟主机
rabbitmqctl status # 节点状态
```

#### 2-10 生产者消费者模型构建
- ConnectionFactory：获取连接工厂
- Connection：一个连接
- Channel：数据通信信道，可发送和接收消息
- Queue：具体的消息存储队列
- Producer & Consumer 生产者和消费者
```
rabbitmq-api项目
quickstart包下的Consumer和Producer
```
```
durable：持久化
exclusive：独有的
autoAck：自动应答
```

#### 2-12 交换机详解
- Exchange：接收消息，并根据路由键转发消息所绑定的队列

**交换机属性**
- Name：交换机名称
- Type：交换机类型direct、topic、fanout、headers
- Durability：是否需要持久化，true为持久化 
- Auto Delete：当最后一个绑定到Exchange上的队列删除后，自动删除该Exchange
- Internal：当前Exchange是否用于RabbitMQ内部使用，默认为false
- Arguments：扩展参数，用于扩展AMQP协议自定制化使用

**Direct Exchange**
- 所有发送到Direct Exchange的消息被转发到RouteKey中指定的Queue

注意：Direct模式可以使用RabbitMQ自带的Exchange：default Exchange，所以不需要将Exchange进行任何绑定操作，消息传递时，RouteKey必须完全匹配才会被队列接收，否则该消息会被抛弃。

**Topic Exchange**
- 所有发送到Topic Exchange的消息被转发到所有关心RouteKey中指定Topic的Queue上
- Exchange将RouteKey和某个Topic进行模糊匹配，此时队列需要绑定一个Topic

注意：可以使用通配符进行模糊匹配
符号“#”匹配一个或多个词
符号“*”匹配不多不少一个词

**Fanout Exchange**
- 不处理路由键，只需要简单的将队列绑定到交换机上
- 发送到交换机的消息都会被转发到与该交换机绑定的所有队列上
- Fanout交换机转发消息是最快的

#### 2-15 绑定、队列、消息、虚拟主机详解
**Binding 绑定**
- Exchange和Exchange、Queue之间的连接关系
- Binding中可以包含RoutingKey或者参数

**Queue 消息队列**
- 消息队列，实际存储消息数据
- Durability：是否持久化，Durable：是，Transient：否
- Auto delete：如选yes，代表当最后一个监听被移除之后，该Queue会自动被删除

**Message 消息**
- 服务器和应用程序之间传送的数据
- 本质上就是一段数据，由Properties和Payload（Body）组成
- 常用属性：delivery mode、headers（自定义属性）
- 其他属性：content_type、content_encoding、priority、correlation_id、reply_to、expiration、message_id、timestamp、type、user_id、app_id、cluster_id

```
api.message包
```

**Virtual host 虚拟主机**
- 虚拟地址，用于进行逻辑隔离，最上层的消息路由
- 一个Virtual Host里面可以用若干个Exchange和Queue
- 同一个Virtual Host里面不能有相同名称的Exchange和Queue

## 第3章 渐进式，深入RabbitMQ高级特性
#### 3-2 消息如何保障 100% 的投递成功方案
**什么是生产端的可靠性投递？**
- 保障消息的成功发出
- 保障MQ节点的成功接收
- 发送端收到MQ节点确认应答
- 完善的消息进行补偿机制

**BAT/TMD互联网大厂的解决方案：**
- 消息落库，对消息状态进行打标
![image.png](https://upload-images.jianshu.io/upload_images/1956963-885dbea1d069c342.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
保障MQ我们思考如果第一种可靠性投递，在高并发的场景下是否合适？
- 消息的延迟投递，做二次确认，回调检查
![image.png](https://upload-images.jianshu.io/upload_images/1956963-67bed1987fe7d1ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 3-4 幂等性概念及业界主流解决方案
**幂等性是什么？**

**消费端 幂等性保障**
在海量订单产生的业务高峰期，如何避免消息的重复消费问题？
- 消费端实现幂等性，就意味着，我们的消息永远不会消费多次，即使我们收到了多条一样的消息

**业界主流的幂等性操作：**
- 唯一ID+指纹码机制，利用数据库主键去重
![image.png](https://upload-images.jianshu.io/upload_images/1956963-d9a6e17346c6a669.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 利用Redis的原子性去实现
![image.png](https://upload-images.jianshu.io/upload_images/1956963-bba4d208a7a857f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 3-5 Confirm确认消息详解
![image.png](https://upload-images.jianshu.io/upload_images/1956963-86957d5623a10d55.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-5cc5227181f483d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-77f4f62c0436bdbf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
api.confirm包
```

#### 3-6 Return返回消息详解
![image.png](https://upload-images.jianshu.io/upload_images/1956963-c771aa4a056a4a6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-e086c6e61c6023ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
api.return包
```

#### 3-7 自定义消费者使用
![image.png](https://upload-images.jianshu.io/upload_images/1956963-603f6256dd7d4c26.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
api.consumer包
```

#### 3-8 消费端的限流策略
![image.png](https://upload-images.jianshu.io/upload_images/1956963-41e3be0ec0a1b0bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-60f0324ea31cb11c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-f25b31d3760bb8a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-5afbb84475080dc8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
api.limit包
注意：autoAck 设置为 false
```

#### 3-10 消费端ACK与重回队列机制
![image.png](https://upload-images.jianshu.io/upload_images/1956963-fae6689e3a0e74b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-af9c34b819d1d19e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
api.ack包
注意：手工签收需要设置autoAck = false
```

#### 3-11 TTL消息详解
![image.png](https://upload-images.jianshu.io/upload_images/1956963-b914c994951b546b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
通过管控台操作
```

#### 3-12 死信队列详解
![image.png](https://upload-images.jianshu.io/upload_images/1956963-82c52da42ad68d71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-bd272a435f635732.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-8a57c7972023f9d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-450528dd86437c66.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-ee6cfd8f372eb8bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
api.dlx包
```

## 第4章 手把手，整合RabbitMQ&Spring家族
#### 4-2 SpringAMQP用户管理组件-RabbitAdmin应用
![image.png](https://upload-images.jianshu.io/upload_images/1956963-8675f8f285a4bba7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-ded0187b4ebcdcf6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-cec0bb1590e4e932.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
rabbitmq-spring
pom.xml
RabbitMQConfig
ApplicationTests
```

#### 4-4 SpringAMQP用户管理组件-RabbitAdmin源码分析
RabbitAdmin源码分析

#### 4-5 SpringAMQP-RabbitMQ声明式配置使用
![image.png](https://upload-images.jianshu.io/upload_images/1956963-aeb9a78c9e7f6145.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-ae1d2fbc48b0103d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
RabbitMQConfig
```

#### 4-6 SpringAMQP消息模板组件-RabbitTemplate实战
![image.png](https://upload-images.jianshu.io/upload_images/1956963-1344ba93d6f5e601.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
RabbitMQConfig
ApplicationTests
```

#### 4-7 SpringAMQP消息容器-SimpleMessageListenerContainer详解
![image.png](https://upload-images.jianshu.io/upload_images/1956963-6171b07d4ca67d26.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-12f6b241cc945d03.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-769b2b0a4d2f90e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-70bb04e466c8977e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
RabbitMQConfig
```

#### 4-8 SpringAMQP消息适配器-MessageListenerAdapter使用
![image.png](https://upload-images.jianshu.io/upload_images/1956963-683721df1b1734a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-c696e3b8008fd72b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
RabbitMQConfig
MessageDelegate
```

#### 4-10 SpringAMQP消息转换器-MessageConverter讲解
![image.png](https://upload-images.jianshu.io/upload_images/1956963-d95b3aa0c533f265.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-09c620380a26b999.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-d4c0008b9c8fa5f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
RabbitMQConfig
```

#### 4-12 RabbitMQ与SpringBoot2.0整合实战
![image.png](https://upload-images.jianshu.io/upload_images/1956963-ac69f9e38a1218ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-f0f23835302f2f35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-06bcd87d5bf7336c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-2e8329e343ab339a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-b0f5e7c12caf9bd9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-54d6c035e2497b88.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
springboot-consumer
springboot-producer
```

#### 4-16 RabbitMQ与Spring Cloud Stream整合实战
- Spring Cloud Stream整体架构核心概念图
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-66bf90f0bd1ea1d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- Barista接口：Barista接口是定义来作为后面类的参数，这一接口定义通道类型和通道名称，通道名称是作为配置用，通道类型则决定了app会使用这一通道进行发送消息还是从中接收消息
- @Output：输出注解，用于定义发送消息的接口
- @Input：输入注解，用于定义消息的消费者接口
- @StreamListener：用于定义监听方法的注解
- 使用Spring Cloud Stream非常简单，只需要使用好这3个注解即可，在实现高性能消息的生产和消费的场景非常合适，但是使用Spring Cloud Stream框架有一个非常大的问题就是不能实现可靠性投递，也就是没法保证消息的100%可靠性，会存在少量消息丢失的问题
- 这个原因是因为Spring Cloud Stream框架为了和Kafka兼顾，所以在实际工作中使用它的目的就是针对高性能的消息通信的！这点就是在当前版本Spring Cloud Stream的定位
```
springcloudstream-producer
springcloudstream-consumer
```

## 第5章 高可靠，构建RabbitMQ集群架构
#### 5-2 RabbitMQ集群架构模式-主备模式（Warren）
![image.png](https://upload-images.jianshu.io/upload_images/1956963-e71e9442c9eed6e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-c0aa99409ea5572d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-c2748a0f345a3fe0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 5-3 RabbitMQ集群架构模式-远程模式（Shovel）
![image.png](https://upload-images.jianshu.io/upload_images/1956963-0fb6d23d2597c029.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-2df589f2094b9ece.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-bde3d0cdce66d382.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-1744a2cd410ab025.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-b3842e0bed0f160e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-be45b189a1ee6b6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 5-4 RabbitMQ集群架构模式-镜像模式（Mirror） 重要！！！
![image.png](https://upload-images.jianshu.io/upload_images/1956963-510ba3626b472499.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-be39bad66c4fbfec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-322eed9fbdcbd091.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 5-5 RabbitMQ集群架构模式-多活模式（Federation）
![image.png](https://upload-images.jianshu.io/upload_images/1956963-36bd686f4225853a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-ba394302dd54600b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-00f5fa92f7580676.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-2dada29c8104a8a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 5-6 RabbitMQ集群镜像队列构建实现可靠性存储
RabbitMQ消息服务用户手册.docx

#### 5-7 RabbitMQ集群整合负载均衡基础组件HaProxy
![image.png](https://upload-images.jianshu.io/upload_images/1956963-951db3f6aba9754c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-33693d54367570a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-8e18306c28b1106f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
RabbitMQ消息服务用户手册.docx

#### 5-8 RabbitMQ集群整合高可用组件KeepAlived
![image.png](https://upload-images.jianshu.io/upload_images/1956963-26fa0141c220f7d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-475ae4125bcfda90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-f2998f320465baaa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-d41c1fcc6ce16263.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
RabbitMQ消息服务用户手册.docx
安装keepalived，修改配置

#### 5-10 RabbitMQ集群配置文件详解
![image.png](https://upload-images.jianshu.io/upload_images/1956963-baa127337818b4c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-e0ecb1c3a2e625ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 5-11 RabbitMQ集群恢复与故障转移的5种解决方案
![image.png](https://upload-images.jianshu.io/upload_images/1956963-f03363ff0dd0a0d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-bd6a6695ee780e6c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-5ba269cb677c26e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![ ](https://upload-images.jianshu.io/upload_images/1956963-598eb1c9ca7219c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-9e217d775fb365de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-0f6926270eadb20e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 5-12 RabbitMQ集群延迟队列插件应用
![image.png](https://upload-images.jianshu.io/upload_images/1956963-56686c187d616091.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-092865a295b9f369.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
RabbitMQ消息服务用户手册.docx 制定扩展

## 第6章 追前沿，领略SET化架构衍化与设计
#### 6-2 BAT、TMD大厂单元化架构设计衍变之路分享
![image.png](https://upload-images.jianshu.io/upload_images/1956963-3a621c80ae270e2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-a7a531c60a4cc8b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-32fd1a1dd9599308.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-e108de478a433fdc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-25aef14383b8a83d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-3c099b0492206570.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-23e27624db70b862.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-27e903b3ba68532c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-69e3deb435cee177.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-bd8ea7558a41bac9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 6-3 SET化架构设计策略(异地多活架构)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-f182de85281a1201.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-6cb303d6edcc0fa1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-fad8e929c43b082c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-c100a79f6c9edbf1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-2310fb71d971d647.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-14921c0f10955b4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-613f85be7405ed46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-8609607b417c9ed9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-55bf7d3bd3619492.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-8994e439527054f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-e83f87c3e8cc8141.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 6-4 SET化架构设计原则
![image.png](https://upload-images.jianshu.io/upload_images/1956963-a52d743fec5b940e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-0f287fc84809884e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-15efe70ad1f10149.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 6-5 SET化消息中间件架构实现
![image.png](https://upload-images.jianshu.io/upload_images/1956963-d1e22cbd3879cdab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-244f24efb4d7e4ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-b019a6438c61d0cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
rabbitmq-plugins enable rabbitmq_federation
rabbitmq-plugins enable rabbitmq_federation_management
```
![image.png](https://upload-images.jianshu.io/upload_images/1956963-7465789378213ef9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-bc74cfb9ee424e05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-1f2ce0115f739144.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 第7章 学大厂，拓展基础组件封装思路
#### 7-2 一线大厂的MQ组件实现思路和架构设计思路
![image.png](https://upload-images.jianshu.io/upload_images/1956963-c03c954d029decd4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-c3e01da45d161419.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-1fb253d19c8da51d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-9efab1ed31770031.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 7-3 基础MQ消息组件设计思路-1（迅速，确认，批量，延迟）
![image.png](https://upload-images.jianshu.io/upload_images/1956963-d7e5d964beb5407b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![ ](https://upload-images.jianshu.io/upload_images/1956963-80348f958df3bbd4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-8be9197826702982.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-969e86d830923a6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-43ef6dc6e263f36e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-c983eac12e1b08f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 7-4 基础MQ消息组件设计思路-2（顺序）
![image.png](https://upload-images.jianshu.io/upload_images/1956963-33ec582a05b8af9f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-a82f2289dcf88233.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-4e30c6534b42b31e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 7-5 基础MQ消息组件设计思路-3(事务)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-1a133791575c17a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-b5b9d8db1eb9e263.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-1cea612eebd84071.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 7-6 消息幂等性保障-消息路由规则架构设计思路
![image.png](https://upload-images.jianshu.io/upload_images/1956963-dda67d201d99b8b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![ ](https://upload-images.jianshu.io/upload_images/1956963-b2359ca29895f376.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## [附录：代码](https://github.com/liuhuiAndroid/rabbitmq-study)
