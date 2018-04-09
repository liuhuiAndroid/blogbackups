目标：撸一个网络请求框架
===================
- ☐ [国内值得关注的API集合](http://www.jianshu.com/p/ecf037476603%20)
- ☐ HTTP协议
我们把Http协议中通信的两方称作Client和Server。Client向Server经过Http协议发送一个Request，Server端收到Request后经过一系列的处理返回Client一个Response。
Http协议是无状态的，我们需要Cookie机制来维护状态，open api常常使用**OAuth2.0**协议。
Request分为Request line、http header和body三个部分
Http协议中的请求方法有GET、POST、PUT ... 
状态码用来告诉HTTP客户端，HTTP服务器是否产生了预期的Response

- ☐ 网络五层结构，每一层协议
OSI七层模型：物理层、数据链路层、网络层、传输层、会话层、表示层、应用层
TCP/IP五层模型的协议：物理层、数据链路层、网络层、传输层、应用层
传输层协议：TCP和UDP
应用层协议：FTP、Telnet、SMTP、HTTP、RIP、NFS、DNS。

- ☐ MAC地址和ip地址的区别
1.IP地址是指Internet协议使用的地址，而MAC地址是Ethernet协议使用的地址。IP地址与MAC地址之间并没有什么必然的联系，MAC地址是Ethernet网卡上带的地址，长度为48位。 
2.IP地址现是32位长，正在扩充到128位。IP地址与MAC地址无关，因为Ethern的用户，仍然可通过Modem连接Internet，取得一个动态的IP地址，这个地址每次可以不一致。IP地址通常工作于广域网，路由器处理的就是IP地址。 MAC地址工作于局域网，局域网之间的互连一般通过现有的公用网或专用线路，需要进行网间协议转换。可以在Ethernet上传送IP信息，此时IP地址只是Ethernet信息包数据域的一部分，Ethernet交换机或处理器看不见IP地址，只是将其作为普通数据处理。

- ☐ Volley源码
> -  [Volley解析合集](http://www.jianshu.com/p/fcd54955f6de)

- ☐ http中缓存机制，Last-Modify的作用等。
> - [彻底弄懂HTTP缓存机制及原理](http://www.cnblogs.com/chenqf/p/6386163.html)

- ☐ OkHttp
> -  [Android OkHttp完全解析 是时候来了解OkHttp了](http://blog.csdn.net/lmj623565791/article/details/47911083)
> - [Android Https相关完全解析 当OkHttp遇到Https](http://blog.csdn.net/lmj623565791/article/details/48129405)
> - [Android 一个改善的okHttp封装库 上面两个过时简单看看](http://blog.csdn.net/lmj623565791/article/details/49734867)
> - [Okhttp-wiki 之 Interceptors 拦截器](http://www.jianshu.com/p/2710ed1e6b48)

- ☐ Retrofit2
> - [Retrofit2 完全解析 探索与okhttp之间的关系](http://blog.csdn.net/lmj623565791/article/details/51304204)
> - [OkHttp, Retrofit, Volley应该选择哪一个？](http://www.jianshu.com/p/77d418e7b5d6)
> - 为什么要用okhttp和retrofit以及源码分析等
> - [Retrofit不进行Json解析，直接返回String](http://www.jianshu.com/p/da561ce76850)
> - [这是一份很详细的 Retrofit 2.0 使用教程](http://www.jianshu.com/p/a3e162261ab6)
> - [手把手带你深入剖析 Retrofit 2.0 源码](http://www.jianshu.com/p/0c055ad46b6c)

- ☐ **自己实现一个网络请求框架和对网络请求框架的封装**
>

- ☐ **实现一个文件下载框架也算是网络的范畴**
>

- ☐ 网络请求缓存处理，okhttp如何处理网络缓存的
> [Android Http缓存数据处理](http://blog.csdn.net/wangkai0681080/article/details/51585097)

- ☐ https相关，如何验证证书的合法性，https中哪里用了对称加密，哪里用了非对称加密，对加密算法（如RSA）等是否有了解

- ☐ 视频加密传输
> [Android 视频文件加密](http://blog.csdn.net/qq_24636637/article/details/50524243)

- ☐ Https请求慢的解决办法，DNS，携带数据，直接访问IP

- ☐ TCP与UDP区别与应用（三次握手和四次挥手）涉及到部分细节（如client如何确定自己发送的消息被server收到） HTTP相关 提到过Websocket 问了WebSocket相关以及与socket的区别

- ☐ 多线程断点续传原理
>[android多线程断点续传原理解析](http://blog.csdn.net/crazy__chen/article/details/41947577)

- ☐ 网络优化
>[Android App优化之网络优化](http://www.jianshu.com/p/d4c2c62ffc35)
>这里拓展到Android App性能优化

- ☐ 网络相关的问题，网络的五层模型，又问了TCP和UDP，还有Android相关的长连接，这里问的比较深。

- ☐ 3次握手和4次挥手的原因，以及为什么需要这样做。
>[TCP为什么需要3次握手与4次挥手](http://blog.csdn.net/xifeijian/article/details/12777187)
>为了防止已失效的连接请求报文段突然又传送到了服务端，因而产生错误
>为了解决“网络中存在延迟的重复分组”的问题

- ☐ http中的同步和异步

- ☐ 刘望舒的一系列网络方面博客
>[Android网络编程（一）HTTP协议原理](http://liuwangshu.cn/application/network/1-http.html)
> [Android网络编程（二）HttpClient与HttpURLConnection](http://liuwangshu.cn/application/network/2-httpclienthttp-urlconnection.html)
 >[Android网络编程（三）Volley用法全解析](http://liuwangshu.cn/application/network/3-volley.html) 
>[Android网络编程（四）从源码解析volley](http://liuwangshu.cn/application/network/4-volley-sourcecode.html) 
>[Android网络编程（五）OkHttp2.x用法全解析](http://liuwangshu.cn/application/network/5-okhttp2x.html)
> [Android网络编程（六）OkHttp3用法全解析](http://liuwangshu.cn/application/network/6-okhttp3.html)
> [Android网络编程（七）源码解析OkHttp前篇[请求网络]](http://liuwangshu.cn/application/network/7-okhttp3-sourcecode.html)
> [Android网络编程（八）源码解析OkHttp后篇[复用连接池]](http://liuwangshu.cn/application/network/8-okhttp3-sourcecode2.html)
> [Android网络编程（九）Retrofit2前篇[基本使用]](http://liuwangshu.cn/application/network/9-retrofit2.html) 
>[Android网络编程（十）Retrofit2后篇[注解]](http://liuwangshu.cn/application/network/10-retrofit2-annotations.html)
> [Android网络编程（十一）源码解析Retrofit](http://liuwangshu.cn/application/network/11-retrofit2-sourcecode.html)
