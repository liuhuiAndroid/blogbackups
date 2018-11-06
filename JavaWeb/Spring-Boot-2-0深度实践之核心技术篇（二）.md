## 第6章 Web MVC REST 应用
#### 6-1 Web MVC REST应用和REST介绍
- REST 简介
REST = RESTful = Representational State Transfer，is one way of providing interoperability between computer systems on the Internet.
- Web MVC REST 支持
- REST 内容协商
- CORS
- 架构约束
统一接口、C/S架构、无状态、可缓存、分层系统、按需代码
- 三个基本方面
资源识别、资源操作、自描述消息

#### 6-2 Web MVC REST 支持
**注解驱动**
- 定义：@Controller、组合注解@RestController
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-06d3151f2f45f41e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 映射：@RequestMapping、别名@*Mapping
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-09d087fb13a4a723.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 请求：@RequestParam、@RequestHeader、@CookieValue
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-acbb586a511a0b84.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 响应：@ResponseBody、ResponseEntity
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-58e34a72e8ac3152.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 拦截：@RestControllerAdvice
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-ff78e6ebdd283f0f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 跨域：@CrossOrigin
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-6156df8fdff787b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 6-3 REST 内容协商
**核心组件**
- 生产媒体类型：@RequestMapping#produces
- HTTP消息转换器：HttpMessageConverter
- REST配置器：WebMvcConfigurer
- 处理方法参数解析器：HandlerMethodArgumentResolver
- 处理方法返回值解析器：HandlerMethodReturnValueHandler
- 内容协商管理器：ContentNegotiationManager
- 媒体类型：MediaType
- 消费媒体类型：@RequestMapping#consumes
- 生产媒体类型：@RequestMapping#produces
- HTTP消息转换器：HttpMessageConverter
- REST配置器：WebMvcConfigurer

![图片.png](https://upload-images.jianshu.io/upload_images/1956963-40fe2aeda4a46097.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 6-4 Web MVC REST 处理流程
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-d48a77ec26d48efd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 6-5 Web MVC REST 处理流程源码分析
```
@SpringBootApplication(scanBasePackages = "com.lh.diveinspringboot")
public class SpringBootRestBootstrap {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootRestBootstrap.class, args);
    }
    
}
```
```

```
```
@RestController
public class HelloWorldRestController {

    @GetMapping(value = "/hello-world")
    public String helloWorld(@RequestParam(required = false) String message) {
        return "Hello,World! : " + message;
    }

    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }

}
```
结合处理流程图分析源码

#### 6-6 Web MVC REST 内容协商处理流程
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-5175fc6254cc7fb2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 6-7 Web MVC REST 内容协商处理流程源码分析
```
@RestController
public class UserRestController {

    @PostMapping(value = "/echo/user")
    public User user(@RequestBody User user) {
        return user;
    }

}
```

#### 6-8 理解媒体类型
**理解请求的媒体类型**
经过 ContentNegotiationManager 的 ContentNegotiationStrategy 解析请求中的媒体类型，比如： Accept 请求头
- 如果成功解析，返回合法 MediaType 列表
- 否则，返回单元素 */* 媒体类型列表 - MediaType.ALL

**理解可生成的媒体类型**
返回 @Controller HandlerMethod @RequestMapping.produces() 属性所指定的 MediaType 列表：
- 如果 @RequestMapping.produces() 存在，返回指定 MediaType 列表
- 否则，返回已注册的 HttpMessageConverter 列表中支持的 MediaType 列表

**理解 @RequestMapping#consumes**
用于 @Controller HandlerMethod 匹配：
- 如果请求头 Content-Type 媒体类型兼容 @RequestMapping.consumes() 属性，执行该 HandlerMethod
- 否则 HandlerMethod 不会被调用

**理解 @RequestMapping#produces**
用于获取可生成的 MediaType 列表
- 如果该列表与请求的媒体类型兼容，执行第一个兼容 HttpMessageConverter 的实现，默认
@RequestMapping#produces 内容到响应头 Content-Type
- 否则，抛出 HttpMediaTypeNotAcceptableException , HTTP Status Code : 415

#### 6-9 理解媒体类型源码分析
```
@RestController
public class UserRestController {

    @PostMapping(value = "/echo/user",
            produces = "application/json;charset=UTF-8")
    public User user(@RequestBody User user) {
        return user;
    }

}
```
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-e9e510ce406e5e89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
@RestController
public class UserRestController {

    @PostMapping(value = "/echo/user",
            produces = "application/json;charset=GBK",
            consumes = "application/json;charset=UTF-8")
    public User user(@RequestBody User user) {
        return user;
    }

}
```

#### 6-10 扩展 REST 内容协商-反序列化部分
###### 自定义 HttpMessageConverter
**需求：**
实现 Content-Type 为 text/properties 媒体类型的 HttpMessageConverter

**实现步骤：**
- 实现 HttpMessageConverter - PropertiesHttpMessageConverter
- 配置 PropertiesHttpMessageConverter 到 WebMvcConfigurer#extendMessageConverters

**代码：**
```
public class PropertiesHttpMessageConverter extends AbstractGenericHttpMessageConverter<Properties> {

    public PropertiesHttpMessageConverter() {
        // 设置支持的 MediaType
        super(new MediaType("text", "properties"));
    }

    @Override
    protected void writeInternal(Properties properties, Type type, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {
        // Properties -> String
        // OutputStream -> Writer
        HttpHeaders httpHeaders = outputMessage.getHeaders();
        MediaType mediaType = httpHeaders.getContentType();
        // 获取字符编码
        Charset charset = mediaType.getCharset();
        // 当 charset 不存在时，使用 UTF-8
        charset = charset == null ? Charset.forName("UTF-8") : charset;
        // 字节输出流
        OutputStream outputStream = outputMessage.getBody();
        // 字符输出流
        Writer writer = new OutputStreamWriter(outputStream, charset);
        // Properties 写入到字符输出流
        properties.store(writer,"From PropertiesHttpMessageConverter");
    }

    @Override
    protected Properties readInternal(Class<? extends Properties> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {

        // 字符流 -> 字符编码
        // 从 请求头 Content-Type 解析编码
        HttpHeaders httpHeaders = inputMessage.getHeaders();
        MediaType mediaType = httpHeaders.getContentType();
        // 获取字符编码
        Charset charset = mediaType.getCharset();
        // 当 charset 不存在时，使用 UTF-8
        charset = charset == null ? Charset.forName("UTF-8") : charset;

        // 字节流
        InputStream inputStream = inputMessage.getBody();
        InputStreamReader reader = new InputStreamReader(inputStream, charset);
        Properties properties = new Properties();
        // 加载字符流成为 Properties 对象
        properties.load(reader);
        return properties;
    }

    @Override
    public Properties read(Type type, Class<?> contextClass, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
        return readInternal(null, inputMessage);
    }
}
```
RestWebMvcConfigurer#extendMessageConverters
```
@Configuration
public class RestWebMvcConfigurer implements WebMvcConfigurer {

    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        converters.set(0, new PropertiesHttpMessageConverter()); // 添加到集合首位
    }
    
}
```
```
@Controller
public class PropertiesRestController {

    @PostMapping(
            value = "/add/props",
            consumes = "text/properties;charset=UTF-8"
    )
    public Properties addProperties(Properties properties) {
        return properties;
    }
    
}
```

#### 6-11 扩展 REST 内容协商-序列化部分
代码：
PropertiesHttpMessageConverter#writeInternal
```
public class PropertiesHttpMessageConverter extends AbstractGenericHttpMessageConverter<Properties> {

    @Override
    protected void writeInternal(Properties properties, Type type, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {
        // Properties -> String
        // OutputStream -> Writer
        HttpHeaders httpHeaders = outputMessage.getHeaders();
        MediaType mediaType = httpHeaders.getContentType();
        // 获取字符编码
        Charset charset = mediaType.getCharset();
        // 当 charset 不存在时，使用 UTF-8
        charset = charset == null ? Charset.forName("UTF-8") : charset;
        // 字节输出流
        OutputStream outputStream = outputMessage.getBody();
        // 字符输出流
        Writer writer = new OutputStreamWriter(outputStream, charset);
        // Properties 写入到字符输出流
        properties.store(writer,"From PropertiesHttpMessageConverter");
    }

}
```

#### 6-12 自定义 Resolver 实现
###### 自定义 HandlerMethodArgumentResolver
**需求：**
- 不依赖 @RequestBody ， 实现 Properties 格式请求内容，解析为 Properties 对象的方法参数
- 复用 PropertiesHttpMessageConverter

**实现步骤：**
- 实现 HandlerMethodArgumentResolver - PropertiesHandlerMethodArgumentResolver
- RequestMappingHandlerAdapter#setArgumentResolvers

**代码：**
PropertiesHandlerMethodArgumentResolver
RestWebMvcConfigurer#addArgumentResolvers

#### 6-13 自定义 Handler 实现
###### 自定义 HandlerMethodReturnValueHandler
**需求：**
- 不依赖 @ResponseBody ，实现 Properties 类型方法返回值，转化为 Properties 格式内容响应内容
- 复用 PropertiesHttpMessageConverter

**实现步骤：**
- 实现 HandlerMethodReturnValueHandler - PropertiesHandlerMethodReturnValueHandler
- RequestMappingHandlerAdapter#setReturnValueHandlers

**代码：**
@Controller
PropertiesHandlerMethodReturnValueHandler

#### 6-14 REST 内容协商CORS
**Cross-Origin Resource Sharing（CORS）**
- 注解驱动：@CrossOrigin
- 代码驱动：WebMvcConfigurer#addCorsMappings
- Filter组件：CorsFilter

代码：
```hosts
127.0.0.1 api.rest.org
```
```
@Controller
public class HelloWorldController {

    @RequestMapping("")
    public String index() {
        return "index";
    }

}
```
```
@RestController
public class HelloWorldRestController {

    @CrossOrigin("*")
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }

}
```
```
@Configuration
public class RestWebMvcConfigurer implements WebMvcConfigurer {

    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**").allowedOrigins("*");
    }

}
```

## 第7章 渐行渐远的 Servlet
#### 7-1 渐行渐远的Servlet
- Servlet 简介
- Spring Servlet Web
- Spring Boot Servlet Web

#### 7-2 Servlet 核心 API
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-4fdc85e57daa56a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 7-3 Servlet 版本
- 基本概念
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-d82f734fa336a420.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 7-4 Servlet 注册
- 传统web.xml注册方式
- 注解注册方式
- 编码注册方式

#### 7-5 理解 Servlet 组件生命周期
**理解 Servlet 生命周期**
- 初始化：Servlet#init(ServletConfig)
- 服务：Servlet#service(ServletRequest,ServletResponse)
- 销毁：Servlet#destory()

**理解 Filter 生命周期**
- 初始化：Filter#init(FilterConfig)
- 过滤：Filter#doFilter(ServletRequest,ServletResponse,FilterChain)
- 销毁：Filter#destory()

**理解 ServletContext 生命周期**
- 初始化：ServletContextListener#contextInitialized
- 销毁：ServletContextListener#contextDestoryed()

**DispatcherServlet初始化过程**
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-6ea6b427f456c2c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
结合源码分析

#### 7-6 Servlet 异步支持
- DeferredResult 支持
- Callable 支持
- CompletionStage 支持

代码：
```
@ComponentScan(basePackages = "com.lh.diveinspringboot.controller")
public class DefaultAnnotationConfigDispatcherServletInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    
    @Override
    protected Class<?>[] getRootConfigClasses() { // web.xml
        return new Class[0];
    }

    @Override
    protected Class<?>[] getServletConfigClasses() { // DispatcherServlet
        return new Class[]{
                getClass() // 返回当前类
        };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

}
```
```
@RestController
public class HelloWorldAsyncController {

    @GetMapping("/hello-world")
    public DeferredResult<String> helloWorld() {
        DeferredResult<String> result = new DeferredResult<>();
        result.setResult("Hello,World");
        result.onCompletion(() -> System.out.println("执行结束"));
        return result;
    }
}
```
```
mvn -Dmaven.test.skip -U clean package // 不执行测试用例，也不编译测试用例类
java -jar target\spring-servlet-0.1-SNAPSHOT-war-exec.jar
```

#### 7-7 DeferredResult 增加线程信息
```
@RestController
public class HelloWorldAsyncController {

    @GetMapping("/hello-world")
    public DeferredResult<String> helloWorld() {
        DeferredResult<String> result = new DeferredResult<>();
        result.setResult("Hello,World");
        println("Hello,World");
        result.onCompletion(() -> println("执行结束"));
        return result;
    }

    private static void println(Object object) {
        String threadName = Thread.currentThread().getName();
        System.out.println("HelloWorldAsyncController[" + threadName + "]: " + object);
    }

}
```
查看日志发现全部是在同一个线程中处理。

#### 7-8 DeferredResult 设置 timeout 以及处理回调
```
@RestController
public class HelloWorldAsyncController {

    @GetMapping("/hello-world")
    public DeferredResult<String> helloWorld() {
        DeferredResult<String> result = new DeferredResult<>(50L);
        println("Hello,World");
        result.onCompletion(() -> println("执行结束"));
        result.onTimeout(() -> println("执行超时"));
        return result;
    }

    private static void println(Object object) {
        String threadName = Thread.currentThread().getName();
        System.out.println("HelloWorldAsyncController[" + threadName + "]: " + object);
    }

}
```
查看日志发现回调是在另一个线程中处理。

#### 7-9 DeferredResult 异步执行
```
@RestController
@EnableScheduling
public class HelloWorldAsyncController {

    private final BlockingQueue<DeferredResult<String>> queue = new ArrayBlockingQueue<>(5);

    // 超时随机数
    private final Random random = new Random();

    // 这个注解在容器启动时便会生效,5秒执行一次任务
    @Scheduled(fixedRate = 5000)
    public void process() throws InterruptedException { // 定时操作
        DeferredResult<String> result = null;
        do {
            result = queue.take();
            // 随机超时时间
            long timeout = random.nextInt(100);
            // 模拟等待时间，RPC 或者 DB 查询
            Thread.sleep(timeout);
            // 计算结果
            result.setResult("Hello,World");
            println("执行计算结果，消耗：" + timeout + " ms.");
        } while (result != null);
    }

    @GetMapping("/hello-world")
    public DeferredResult<String> helloWorld() {
        DeferredResult<String> result = new DeferredResult<>(50L);
        // 将指定元素插入此队列中
        queue.offer(result);
        println("Hello,World");
        result.onCompletion(() -> println("执行结束"));
        result.onTimeout(() -> println("执行超时"));
        return result;
    }

    private static void println(Object object) {
        String threadName = Thread.currentThread().getName();
        System.out.println("HelloWorldAsyncController[" + threadName + "]: " + object);
    }

}
```

#### 7-10 Callable 异步执行
```
@RestController
public class HelloWorldAsyncController {

    @GetMapping("/callable-hello-world")
    public Callable<String> callableHelloWorld() {
        final long startTime = System.currentTimeMillis();
        println("Hello,World");
        return () -> {
            long costTime = System.currentTimeMillis() - startTime;
            println("执行计算结果，消耗：" + costTime + " ms.");
            return "Hello,World";
        };
    }

    private static void println(Object object) {
        String threadName = Thread.currentThread().getName();
        System.out.println("HelloWorldAsyncController[" + threadName + "]: " + object);
    }

}
```

#### 7-11 CompletionStage 异步执行
```
@RestController
public class HelloWorldAsyncController {

    @GetMapping("/completion-stage")
    public CompletionStage<String> completionStage(){
        final long startTime = System.currentTimeMillis();
        println("Hello,World");
        return CompletableFuture.supplyAsync(()->{
            long costTime = System.currentTimeMillis() - startTime;
            println("执行计算结果，消耗：" + costTime + " ms.");
            return "Hello,World"; // 异步执行结果
        });
    }

    private static void println(Object object) {
        String threadName = Thread.currentThread().getName();
        System.out.println("HelloWorldAsyncController[" + threadName + "]: " + object);
    }

}
```

#### 7-12 MVC 异步支持原理分析
- HandlerMethodReturnValueHandler
- Servlet 3.0 AsyncContext
- DispatcherServlet 整合

#### 7-13 异步 Servlet 实现
```
@WebServlet(
        asyncSupported = true, // 激活异步特性
        name = "asyncServlet", // Servlet 名字
        urlPatterns = "/async-servlet"
)
public class AsyncServlet extends HttpServlet {

    @Override
    public void service(HttpServletRequest request, HttpServletResponse resp) throws IOException {
        // 判断是否支持异步
        if (request.isAsyncSupported()) {
            // 创建 AsyncContext
            AsyncContext asyncContext = request.startAsync();
            // 设置超时时间
            asyncContext.setTimeout(1000L);
            asyncContext.addListener(new AsyncListener() {
                @Override
                public void onComplete(AsyncEvent event) {
                    println("执行完成");
                }

                @Override
                public void onTimeout(AsyncEvent event) {
                    HttpServletResponse servletResponse = (HttpServletResponse)event.getSuppliedResponse();
                    servletResponse.setStatus(HttpServletResponse.SC_SERVICE_UNAVAILABLE);
                    println("执行超时");
                }

                @Override
                public void onError(AsyncEvent event) {
                    println("执行错误");
                }

                @Override
                public void onStartAsync(AsyncEvent event) {
                    println("开始执行");
                }
            });

            println("Hello,World");
            ServletResponse servletResponse = asyncContext.getResponse();
            // 设置响应媒体类型
            servletResponse.setContentType("text/plain;charset=UTF-8");
            // 获取字符输出流
            PrintWriter writer = servletResponse.getWriter();
            writer.println("Hello,World");
            writer.flush();
        }
    }

    private static void println(Object object) {
        String threadName = Thread.currentThread().getName();
        System.out.println("AsyncServlet[" + threadName + "]: " + object);
    }

}
```
Java Specification Requests (JSR) : https://github.com/mercyblitz/jsr

#### 7-14 DefferedResult 实现原理
HandlerMethodReturnValueHandler 源码
DeferredResultMethodReturnValueHandler 源码
AsyncContext 源码

#### 7-15 Spring Boot 嵌入式 Servlet 容器限制
- 不支持 web.xml 部署
- 不支持 ServletContainerInitializer 接口
- 注解驱动限制

代码：
spring-boot-servlet 项目
SpringBootServletBootstrap

#### 7-16 Spring Boot 嵌入式 Servlet 容器限制 原理分析
没听懂

#### 7-17 Spring Boot 应用传统 Servlet 容器部署
- SpringBootServletInitializer 
- Spring 3.1 + SPI WebApplicationInitializer
- Servlet 3.0 + SPI ServletContainerInitializer

#### 7-18 扩展 SpringBootServletInitializer
- 使用 Tomcat7 插件
- 使用 Tomcat8 插件
DefaultSpringBootServletInitializer

#### 7-19 构建应用

#### 7-20 渐行渐远的Servlet总结
- Spring Servlet Web
- Spring Boot Servlet Web
- Spring Boot 应用传统 Servlet 容器部署

## 第8章 从 Reactive 到 WebFlux
#### 8-1 从 Reactive 到 WebFlux
- 理解 Reactive
- Reactive Streams 规范
- Reactor 框架运用
- 走向 Spring WebFlux

温故知新
- 反应堆模式（Reactor） 同步非阻塞
- Proactor模式 异步非阻塞
- 观察者模式（Observer）
- 迭代器模式（Iterator）
- Java并发模型

#### 8-2 关于 Reactive 的一些说法
- Reactive 是异步非阻塞编程
- Reactive 能够提升程序性能
- Reactive 能解决传统编程模型遇到的困境

Reactive 实现框架
- RxJava：Reactive Extensions Java
- Reactor：Spring WebFlux Reactive 类库
- Flow API：Java 9 Flow API 实现

#### 8-3 理解阻塞的弊端和并行的复杂
[**Reactor**](http://projectreactor.io/docs/core/release/reference/#_blocking_can_be_wasteful) 认为阻塞可能是浪费的
- 阻塞导致性能瓶颈和浪费资源
- 增加线程可能会引起资源竞争和并发问题
- 并行的方式不是银弹（不能解决所有问题）

**理解阻塞的弊端**
DataLoader.java
**结论：**由于加载过程串行执行的关系，导致消耗实现线性累加。Blocking 模式即串行执行 。

**理解并行的复杂**
ParallelDataLoader.java
**结论：**明显地，程序改造为并行加载后，性能和资源利用率得到提升，消耗时间取最大者。

#### 8-4 Reactor 认为异步不一定能够救赎
[**Reactor**](http://projectreactor.io/docs/core/release/reference/#_asynchronicity_to_the_rescue) 认为异步不一定能够救赎
- Callbacks 是解决非阻塞的方案，然而他们之间很难组合，并且快速地将代码引导至 "Callback Hell"的不归路
- Futures 相对于 Callbacks 好一点，不过还是无法组合，不过   CompletableFuture 能够提升这方面的不足

#### 8-5 理解 Callback Hell
JavaGUI.java
结论：

#### 8-6 理解 Future 阻塞问题
FutureBlockingDataLoader.java
结论：

#### 8-7 理解 Future 链式问题
ChainDataLoader.java
结论：

#### 8-8 Reactive Streams JVM 认为异步系统和资源消费需要特殊处理
观点归纳：
- 
- 
- 

#### 8-9 Reactive Programming 定义
定义：
- 维基百科
- The Reactive Manifesto
- Spring Framework
- ReactiveX
- Reactor
- @andrestaltz

#### 8-10 Reactive Manifesto 定义
关键字：
侧重点：

#### 8-11 维基百科
关键字：
侧重点：
技术连接：

#### 8-12 Spring Framework 定义
关键字：
侧重点：
技术连接：

#### 8-13 ReactiveX 定义
关键字：
侧重点：
技术连接：

#### 8-14 Reactor 定义
关键字：
侧重点：
技术连接：

#### 8-15 andrestaltz 定义
关键字：
侧重点：
技术连接：

#### 8-16 Reactive Programming 特性：编程模型
**Reactive 编程模型**
- 语言模型：响应式编程+函数式编程
- 编发模型：多线程编程
- 对立模型：命令式编程

小结：

#### 8-17 Reactive Programming 特性：数据结构
**Reactive 数据结构**
- 流式
- 序列
- 事件

小结：

**Reactive 设计模式**
- 扩展模式：观察者
- 对立模式：迭代器
- 混合模式：反应堆（Reactor）、Proactor


小结：

#### 8-18 Reactive Programming 特性：并发模型
- 非阻塞
同步
异步

小结：

#### 8-19 Reactive Programming 使用场景
**Reactive Streams JVM**
**Spring Framework**
**ReactiveX**
**Reactor**

#### 8-20 Reactive Streams 规范：定义
关键字：
- 潜在无界性
- 序列
- 异步传递
- 非阻塞背压

见Github：https://github.com/reactive-streams/reactive-streams-jvm

#### 8-21 Reactive Streams 规范：API和事件
- Publisher：数据发布者（上游）
- Subscriber：数据订阅者（下游）
- Subscription：订阅信号
- Processor：Publisher和Subscriber混合体

Subscriber信号事件：
- onSubscribe：当下游订阅时
- onNext：当下游接收数据时
- onComplete：当数据流执行完成时
- onError：当数据流执行错误时

Subscription信号操作：
- request：请求上游元素的数量
- cancel：请求停止发送数据并且清除资源

#### 8-22 Reactive Streams 规范：背压
- 维基百科
关键字
- Reactive Streams JVM
- Reactor

**总结背压**

#### 8-23 Reactor 框架运用 - 核心 API
**核心 API**
- Mono：0-1的异步结果
- Flux：0-N的异步序列
- Scheduler：Reactor调度线程池

#### 8-24 Reactor 框架运用实战（上）
依赖
FluxDemo

#### 8-25 Reactor 框架运用实战（下）
 FluxDemo 背压

#### 8-26 走向 Spring WebFlux
**Reactive Spring Web**
- 背景介绍
- 对比Spring Web MVC

## 第9章 WebFlux 核心
#### 9-1 WebFlux 核心
- Reactor API
- 基本介绍

#### 9-2 官方引入WebFlux的动机分析
- 动机
去Servlet化

#### 9-3 回顾Reactive
- 定义Reactive

#### 9-4 编程模型：注解驱动
- 注解驱动

#### 9-5 Java 函数编程基础
springboot-webflux
- Java函数编程基础

#### 9-6 编程模型：函数式端点 - Functional Endpoints
WebFluxApplication
hello-world

#### 9-7 WebFlux 核心 - 并发模型
- 非阻塞
- 异步
- Spring官方说明

代码：
WebFluxApplication
mvc、mono

#### 9-8 WebFlux 核心 - 核心组件
- HttpHandler API
- WebHandler API
- Web MVC VS WebFlux

#### 9-9 WebFlux 核心处理流程 - 函数式端点组件请求处理流程
见大纲

#### 9-10 WebFlux 核心处理流程 - 注解驱动组件请求处理流程
见大纲

#### 9-11 WebFlux 核心 - 使用场景
#### 9-12 WebFlux 核心 - 课堂总结
#### 9-13 WebFlux 核心 - 课程彩蛋

## [源码](https://github.com/liuhuiAndroid/dive-in-spring-boot)
