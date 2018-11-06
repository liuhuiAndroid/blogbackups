### 哪些开源框架使用了Netty
Dubbo、RocketMQ、Spark、Elasticsearch、Cassandra、Flink、Netty-SocketIO、Spring5、Play、Grpc

### Netty是什么？
- 异步事件驱动框架，用于快速开发高性能服务端和客户端
- 封装了JDK底层BIO和NIO模型，提供高度可用的API
- 自带编解码器解决拆包粘包问题，用户只用关心业务逻辑
- 精心设计的reactor线程模型支持高并发海量连接
- 自带各种协议栈让你处理任何一种通用协议都几户不用亲自动手

### Netty基本组件
NioEventLoop ——> Thread
Channel ——> Socket
ByteBuf ——> IO Bytes
Pipeline ——> Logic Chain
ChannelHandler——> Logic

# 三 Netty服务端启动过程
### Netty服务端启动过程
- 创建服务端Channel
![image.png](https://upload-images.jianshu.io/upload_images/1956963-74601991c9f3d9c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/1956963-088462efef189f2d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 初始化服务端Channel
![image.png](https://upload-images.jianshu.io/upload_images/1956963-9b17fa2fdc646a22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/1956963-a71e823cfec2027a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 注册selector
![image.png](https://upload-images.jianshu.io/upload_images/1956963-b68a0eda2b46edd2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 端口绑定
![image.png](https://upload-images.jianshu.io/upload_images/1956963-316166967eb8774c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/1956963-9f6fe716f5cd7ba3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 四
