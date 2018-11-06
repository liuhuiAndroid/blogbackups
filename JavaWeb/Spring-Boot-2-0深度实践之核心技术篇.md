## 第1章 系列总览
###### 1-2 Spring Boot 2.0 概要
- 易学难精
组件自动装配、外部化配置、嵌入式容器、Spring Boot Starter、Production Ready
- Spring Boot 与 Java EE 规范
Web、SQL、数据校验、缓存、WebSockets、Web Services、Java 管理、消息

###### 1-3 开场白：系列总览
- 核心特性、Web应用、数据相关、功能扩展、运维管理

###### 1-4 核心特性介绍
Spring Boot 三大核心特性
- 组件自动装配
- 嵌入式Web容器
- 生成准备特性

###### 1-5 核心特性之组件自动装配工程部分
组件自动装配
- 激活：@EnableAutoConfiguration
- 配置：/META-INF/spring.factories
- 实现：XXXAutoConfiguration
[Spring Initializr](https://start.spring.io/) Generate Project 填入Group：com.lh，填入Artifact：dive-in-spring-boot，使用idea打开
嵌入式Web容器
- Web Servlet：Tomcat、Jetty和Undertow
- Web Reactive：Netty Web Server
生产准备特性
- 指标：/actuator/metrics
- 建康检查：/actuator/health
- 外部化配置：/actuator/configprops

###### 1-6 Web应用介绍
传统Servlet应用
- Servlet组件：Servlet、Filter、Listener
- Servlet注册：Servlet注解、Spring Bean、RegistrationBean
- 异步非阻塞：异步Servlet、非阻塞Servlet

###### 1-7 传统 Servelt 应用
依赖
```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
Servlet 组件
- Servlet
1. 实现
`@WebServlet`
`HttpServlet`
`注册`
2. URL 映射
`@WebServlet(urlPatterns = "/my/servlet")`
3. 注册
`@ServletComponentScan(basePackages = "com.imooc.diveinspringboot.web.servlet")`
- Filter
- Listener
```
@SpringBootApplication
@ServletComponentScan(basePackages = "com.lh.diveinspringboot.web.servlet")
public class DiveInSpringBootApplication{ ... }

@WebServlet(urlPatterns = "/my/servlet")
public class MyServlet extends HttpServlet {

    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        resp.getWriter().println("Hello,World");
    }

}
```

###### 1-8 异步非阻塞 Servlet 代码示例
结合异步Servlet，从同步方式转换成异步方式
```
@WebServlet(urlPatterns = "/my/servlet", asyncSupported = true)
public class MyServlet extends HttpServlet {

    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        AsyncContext asyncContext = req.startAsync();
        asyncContext.start(() -> {
            try {
                resp.getWriter().println("Hello,World!");
                // 触发完成
                asyncContext.complete();
            } catch (IOException e) {
                e.printStackTrace();
            }
        });
    }

}
```

###### 1-9 Spring Web MVC 应用介绍
- Web MVC视图：模板引擎、内容协商、异常处理等
Web MVC 视图
模板引擎
内容协商
异常处理
- Web MVC REST：资源服务、资源跨域、服务发现等
资源服务
资源跨域
服务发现
- Web MVC核心：核心架构、处理流程、核心组件
核心架构
处理流程
核心组件

###### 1-10 Spring WebFlux 应用
- Reactor基础：Java Lambda、Mono、Flux
- Web Flux核心：Web MVC注解、函数式声明、异步非阻塞
- 使用场景：Web Flux优势和限制

###### 1-11 Web Server 应用
- 切换Web Server
Tomcat -> Jetty， 注意查看依赖变化
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <!-- Exclude the Tomcat dependency -->
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!-- Use Jetty instead -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
```
替换 Servlet 容器，WebFlux，需要注释掉web
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>
```
- 自定义Servlet Web Server
WebServerFactoryCustomizer 有很多实现，比如TomcatServletWebServerFactoryCustomizer
- 自定义Reactive Web Server
ReactiveWebServerFactoryCustomizer

###### 1-12 数据相关介绍
关系型数据
- JDBC：数据源、JdbcTemplate、自动装配
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
```
数据源：javax.sql.DataSource
JdbcTemplate
自动装配：DataSourceAutoConfiguration
- JPA：实体映射关系、实体操作、自动装配
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
```
实体映射关系：@javax.persistence.OneToOne@javax.persistence.OneToMany@javax.persistence.ManyToOne@javax.persistence.ManyToMany
实体操作：javax.persistence.EntityManager
自动装配：HibernateJpaAutoConfiguration
- 事务：Spring事务抽象、JDBC事务处理、自动装配
```
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
        </dependency>
```
Spring 事务抽象：PlatformTransactionManager
JDBC 事务处理：DataSourceTransactionManager
自动装配：TransactionAutoConfiguration

###### 1-13 功能扩展介绍
Spring Boot应用
- SpringApplication：失败分析、应用特性、事件监听等
失败分析：FailureAnalysisReporter
应用特性：SpringApplication Fluent API
- Spring Boot配置：外部化配置、Profile、配置属性
外部化配置：ConfigurationProperty
@Profile
配置属性：PropertySources
- Spring Boot Starter：Starter开发、最佳实践

###### 1-14 运维管理介绍
Spring Boot Actuator
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```
```
self：http://localhost:8080/actuator
health：http://localhost:8080/actuator/health
info：http://localhost:8080/actuator/info
```
- 端点：各类Web和JMX Endpoints
- 健康检查：Health、HeadlthIndicator
- 指标：内建Metrics、自定义Metrics

## 第2章 走向自动装配
#### 2-1 走向自动装配
- Spring Framework手动装配
- Spring Boot自动装配

#### 2-2 Spring Framework 手动装配
[模式注解（Stereotype Annotation)](https://github.com/spring-projects/spring-framework/wiki/Spring-Annotation-Programming-Model#stereotype-annotations)
>A stereotype annotation is an annotation that is used to declare the role that a component plays within the application. For example, the @Repository annotation in the Spring Framework is a marker for any class that fulfills the role or stereotype of a repository (also known as Data Access Object or DAO).
>@Component is a generic stereotype for any Spring-managed component. Any component annotated with @Component is a candidate for component scanning. Similarly, any component annotated with an annotation that is itself meta-annotated with @Component is also a candidate for component scanning. For example, @Service is meta-annotated with @Component.

>模式注解是一种用于声明在应用中扮演“组件”角色的注解。如 Spring Framework 中的 @Repository 标注在任何类上 ，用于扮演仓储角色的模式注解。
@Component 作为一种由 Spring 容器托管的通用模式组件，任何被 @Component 标准的组件均为组件扫描的候选对象。类似地，凡是被 @Component 元标注（meta-annotated）的注解，如 @Service ，当任何组件标注它时，也被视作组件扫描的候选对象

Spring 模式注解装配
- 定义：一种用于声明在应用中扮演“组件”角色的注解
- 举例：数据仓储模式注解@Repository、通用组件模式注解@Component、服务模式注解@Service、Web 控制器模式注解@Controller、配置类模式注解@Configuration等
- 装配：<context:component-scan>或@ComponentScan

<context:component-scan> 方式
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans.xsd
  http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-
context.xsd">
  <!-- 激活注解驱动特性 -->
  <context:annotation-config />
  <!-- 找寻被 @Component 或者其派生 Annotation 标记的类（Class），将它们注册为 Spring Bean -->
  <context:component-scan base-package="com.imooc.dive.in.spring.boot" />
</beans>
```
@ComponentScan 方式
```
@ComponentScan(basePackages = "com.imooc.dive.in.spring.boot")
public class SpringConfiguration {
  ...
}
```

#### 2-3 Spring Framework手动装配自定义模式注解
[Spring Initializr](https://start.spring.io/) 生成dive-in-spring-boot项目
- @Component “派生性”
通过FirstLevelRepository、MyFirstLevelRepository和RepositoryBootstrap理解派生性。
@Component -> @Repository -> FirstLevelRepository
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repository
public @interface FirstLevelRepository {
    String value() default "";
}
```
```
@FirstLevelRepository(value = "myFirstLevelRepository") // Bean 名称
public class MyFirstLevelRepository {
}
```
```
@ComponentScan(basePackages = "com.lh.diveinspringboot.repository")
public class RepositoryBootstrap {
    
    public static void main(String[] args) {
        ConfigurableApplicationContext context = new SpringApplicationBuilder(RepositoryBootstrap.class)
                .web(WebApplicationType.NONE)
                .run(args);

        // myFirstLevelRepository Bean 是否存在
        MyFirstLevelRepository myFirstLevelRepository =
                context.getBean("myFirstLevelRepository",MyFirstLevelRepository.class);

        System.out.println("myFirstLevelRepository Bean : "+myFirstLevelRepository);

        // 关闭上下文
        context.close();
    }
}
```
- @Component “层次性”
通过SecondLevelRepository、MyFirstLevelRepository
@Component -> @Repository -> FirstLevelRepository -> SecondLevelRepository
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@FirstLevelRepository
public @interface SecondLevelRepository {
    String value() default "";
}
```
```
@SecondLevelRepository(value = "myFirstLevelRepository") // Bean 名称
public class MyFirstLevelRepository {
}
```

#### 2-4 @Enable 模块装配两种方式
>Spring Framework 3.1 开始支持“@Enable 模块驱动”。所谓“模块”是指具备相同领域的功能组件集合， 组合所形成一个独立的单元。比如 Web MVC 模块、AspectJ代理模块、Caching（缓存）模块、JMX（Java 管 理扩展）模块、Async（异步处理）模块等。
- Spring @Enable 注解模块装配
- 定义：具备相同领域的功能组件集合，组合所形成一个独立的单元
- 举例：@EnableWebMvc、@EnableAutoConfiguration等
- 两种实现：注解方式、编程方式

**官方实现方式**
- 注解驱动方式
```
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import({DelegatingWebMvcConfiguration.class})
public @interface EnableWebMvc {
}
```
```
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
  ...
}
```
- 接口编程方式
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({CachingConfigurationSelector.class})
public @interface EnableCaching {
    ...
}
```
```
public class CachingConfigurationSelector extends AdviceModeImportSelector<EnableCaching> {

    public String[] selectImports(AdviceMode adviceMode) {
        switch(adviceMode) {
        case PROXY:
            return this.getProxyImports();
        case ASPECTJ:
            return this.getAspectJImports();
        default:
            return null;
        }
    }
}
```

**自定义 @Enable 模块**
- 基于注解驱动实现

- 基于接口驱动实现
HelloWorldImportSelector -> HelloWorldConfiguration -> HelloWorld
```
public class HelloWorldImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        return new String[]{HelloWorldConfiguration.class.getName()};
    }
}
```
```
public class HelloWorldConfiguration {
    @Bean
    public String helloWorld() { // 方法名即 Bean 名称
        return "Hello,World 2018";
    }
}
```
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
//@Import(HelloWorldConfiguration.class)
@Import(HelloWorldImportSelector.class)
public @interface EnableHelloWorld {
}
```
```
@EnableHelloWorld
public class EnableHelloWorldBootstrap {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = new SpringApplicationBuilder(EnableHelloWorldBootstrap.class)
                .web(WebApplicationType.NONE)
                .run(args);

        String helloWorld = context.getBean("helloWorld", String.class);
        System.out.println("helloWorld Bean : " + helloWorld);

        context.close();
    }
}
```
ImportSelector方式比Configuration方式实现Enable模块自定义更加灵活

#### 2-5 Spring条件装配
>从 Spring Framework 3.1 开始，允许在 Bean 装配时增加前置条件判断
- 定义：Bean装配的前置判断
- 举例：@Profile 配置化条件装配、@Conditional 编程条件装配
- 实现：注解方式、编程方式

#### 2-6 基于配置方式实现自定义条件装配 —— @Profile
计算服务，多数据求和 sum
@Profile("Java7") : for 循环
@Profile("Java8") : Lambda
```
public interface CalculateService {
    Integer sum(Integer... values);
}
```
```
@Profile("Java7")
@Service
public class Java7CalculateService implements CalculateService {
    @Override
    public Integer sum(Integer... values) {
        System.out.println("Java 7 for 循环实现 ");
        int sum = 0;
        for (int i = 0; i < values.length; i++) {
            sum += values[i];
        }
        return sum;
    }
}
```
```
@Profile("Java8")
@Service
public class Java8CalculateService implements CalculateService {
    @Override
    public Integer sum(Integer... values) {
        System.out.println("Java 8 Lambda 实现");
        int sum = Stream.of(values).reduce(0, Integer::sum);
        return sum;
    }
}
```
```
@SpringBootApplication(scanBasePackages = "com.lh.diveinspringboot.service")
public class CalculateServiceBootstrap {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = new SpringApplicationBuilder(CalculateServiceBootstrap.class)
                .web(WebApplicationType.NONE)
                .profiles("Java7")
                .run(args);

        // CalculateService Bean 是否存在
        CalculateService calculateService = context.getBean(CalculateService.class);
        System.out.println("calculateService.sum(1...10) : " +
                calculateService.sum(1, 2, 3, 4, 5, 6, 7, 8, 9, 10));

        // 关闭上下文
        context.close();
    }
}
```

#### 2-7 基于编程方式实现条件装配 —— @ConditionalOnSystemProperty
```
@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.TYPE, ElementType.METHOD })
@Documented
@Conditional(OnSystemPropertyCondition.class)
public @interface ConditionalOnSystemProperty {

    /**
     * Java 系统属性名称
     */
    String name();

    /**
     * Java 系统属性值
     */
    String value();
}
```
```
/**
 * 系统属性条件判断
 */
public class OnSystemPropertyCondition implements Condition {
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        Map<String, Object> attributes = annotatedTypeMetadata.getAnnotationAttributes(ConditionalOnSystemProperty.class.getName());
        String propertyName = String.valueOf(attributes.get("name"));
        String propertyValue = String.valueOf(attributes.get("value"));
        String javaPropertyValue = System.getProperty(propertyName);
        return propertyValue.equals(javaPropertyValue);
    }
}
```
```
public class ConditionalOnSystemPropertyBootstrap {

    @Bean
    @ConditionalOnSystemProperty(name = "user.name", value = "we-win")
    public String helloWorld() {
        return "Hello,World.";
    }

    public static void main(String[] args) {
        ConfigurableApplicationContext context = new SpringApplicationBuilder(ConditionalOnSystemPropertyBootstrap.class)
                .web(WebApplicationType.NONE)
                .run(args);

        String helloWorld = context.getBean("helloWorld", String.class);
        System.out.println("helloWorld Bean : " + helloWorld);

        // 关闭上下文
        context.close();
    }
}
```

#### 2-8 Spring Boot 自动装配
- 定义：基于约定大于配置的原则，实现Spring组件自动装配的目的
- 底层装配技术：Spring 模式注解装配、Spring @Enable 模块装配、Spring 条件装配装配、Spring 工厂加载机制

**其中Spring 工厂加载机制：**
1. 实现类： SpringFactoriesLoader
2. 配置资源： META-INF/spring.factories
- 实现：激活自动装配、实现自动装配、配置自动装配实现

#### 2-9 自定义自动装配
参考 META-INF/spring.factories
**实现方法：**
1. 激活自动装配 - @EnableAutoConfiguration
EnableAutoConfigurationBootstrap
```
@EnableAutoConfiguration
public class EnableAutoConfigurationBootstrap {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = new SpringApplicationBuilder(EnableAutoConfigurationBootstrap.class)
                .web(WebApplicationType.NONE)
                .run(args);

        String helloWorld = context.getBean("helloWorld", String.class);
        System.out.println("helloWorld Bean : " + helloWorld);

        // 关闭上下文
        context.close();
    }

}
```

2. 实现自动装配 - XXXAutoConfiguration
HelloWorldAutoConfiguration
```
@Configuration // Spring 模式注解装配
@EnableHelloWorld // Spring @Enable 模块装配
@ConditionalOnSystemProperty(name = "user.name", value = "we-win") // 条件装配
public class HelloWorldAutoConfiguration {
}
```

3. 配置自动装配实现 - META-INF/spring.factories
```
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.lh.diveinspringboot.configuration.HelloWorldAutoConfiguration
```

## 第3章 理解 SpringApplication
#### 3-1 理解 SpringApplication
**[官方文档 SpringApplication](https://docs.spring.io/spring-boot/docs/2.0.5.RELEASE/reference/htmlsingle/#boot-features-spring-application)**
>The SpringApplication class provides a convenient way to bootstrap a Spring application that is started from a main() method. In many situations, you can delegate to the static SpringApplication.run method, as shown in the following example:
```
public static void main(String[] args) {
	SpringApplication.run(MySpringConfiguration.class, args);
}
```
- SpringApplication 准备阶段
- SpringApplication 运行阶段

#### 3-2 基础技术和衍生技术
基础技术 Spring Framework
- Spring 模式注解
- Spring 应用上下文
- Spring 工厂加载机制
- Spring 应用上下文初始器
- Spring Environment 抽象
- Spring 应用事件/监听器

衍生技术 Spring Boot
- SpringApplication
- SpringApplication Builder API
- SpringApplication 运行监听器
- SpringApplication 参数
- SpringApplication 故障分析
- SpringApplication 应用事件/监听器

#### 3-3 合并工程
[**IntelliJ IDEA中创建Web聚合项目**](https://blog.csdn.net/u012129558/article/details/78423511)

#### 3-4 SpringApplication 准备阶段
- 定义：Spring应用引导类，提供便利的自定义行为方法
- 场景：嵌入式Web应用和非Web应用
- 运行：SpringApplication#run(String...)
- 配置：Spring Bean 来源
- 推断：Web 应用类型和主引导类（Main Class）
- 加载：应用上下文初始器和应用事件监听器

**SpringApplication 运行**
```
SpringApplication.run(DiveInSpringBootApplication.class, args)
```
**自定义 SpringApplication**
1. 通过 SpringApplication API 调整
```
		SpringApplication springApplication = new SpringApplication(DiveInSpringBootApplication.class);
		springApplication.setBannerMode(Banner.Mode.CONSOLE);
		springApplication.setWebApplicationType(WebApplicationType.NONE); // 非Web类型
		springApplication.setAdditionalProfiles("prod"); // 线上方式
		springApplication.setHeadless(true);
		springApplication.run(args);

		Set<String> sources = new HashSet();
		sources.add(DiveInSpringBootApplication.class.getName());
		SpringApplication springApplication = new SpringApplication();
		springApplication.setSources(sources);
```
2. 通过 SpringApplicationBuilder API 调整
**[Fluent Builder API](https://docs.spring.io/spring-boot/docs/2.0.5.RELEASE/reference/htmlsingle/#boot-features-fluent-builder-api)**
```
		new SpringApplicationBuilder(DiveInSpringBootApplication.class)
				.bannerMode(Banner.Mode.CONSOLE)
				.web(WebApplicationType.SERVLET)
				.profiles("prod")
				.headless(true)
				.run(args);
```
**配置 Spring Boot Bean 源**
Java 配置 Class 或 XML 上下文配置文件集合，用于 Spring Boot `BeanDefinitionLoader`读取 ，并且将配置源解析加载为Spring Bean 定义
- Java 配置 Class
用于 Spring 注解驱动中 Java 配置类，大多数情况是 Spring 模式注解所标注的类，如 @Configuration。
DiveInSpringBootApplication就是一个配置 Class
@SpringBootApplication 是Java 模式注解
- XML 上下文配置文件
用于 Spring 传统配置驱动中的 XML 文件。

#### 3-5 配置 Spring Boot Bean 源码部分
```
public class DiveInSpringBootApplication {
    public static void main(String[] args) {
        Set<String> sources = new HashSet();
        sources.add(ApplicationConfiguration.class.getName());
        SpringApplication springApplication = new SpringApplication();
        springApplication.setSources(sources);
        ConfigurableApplicationContext context = springApplication.run(args);
        System.out.print("Bean:" + context.getBean(ApplicationConfiguration.class));
    }

    @SpringBootApplication
    public static class ApplicationConfiguration {
    }
}
```

#### 3-6 推断 Web 应用类型
根据当前应用 ClassPath 中是否存在相关实现类来推断 Web 应用的类型，包括：
- Web Reactive： WebApplicationType.REACTIVE
- Web Servlet： WebApplicationType.SERVLET
- 非 Web： WebApplicationType.NONE

参考方法： org.springframework.boot.SpringApplication#deduceWebApplicationType
```
	private WebApplicationType deduceWebApplicationType() {
		if (ClassUtils.isPresent(REACTIVE_WEB_ENVIRONMENT_CLASS, null)
				&& !ClassUtils.isPresent(MVC_WEB_ENVIRONMENT_CLASS, null)
				&& !ClassUtils.isPresent(JERSEY_WEB_ENVIRONMENT_CLASS, null)) {
			return WebApplicationType.REACTIVE;
		}
		for (String className : WEB_ENVIRONMENT_CLASSES) {
			if (!ClassUtils.isPresent(className, null)) {
				return WebApplicationType.NONE;
			}
		}
		return WebApplicationType.SERVLET;
	}
```

#### 3-7 推断引导类
根据 Main 线程执行堆栈判断实际的引导类
参考方法： org.springframework.boot.SpringApplication#deduceMainApplicationClass
```
	private Class<?> deduceMainApplicationClass() {
		try {
			StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
			for (StackTraceElement stackTraceElement : stackTrace) {
				if ("main".equals(stackTraceElement.getMethodName())) {
					return Class.forName(stackTraceElement.getClassName());
				}
			}
		}
		catch (ClassNotFoundException ex) {
			// Swallow and continue
		}
		return null;
	}
```
断点调试堆栈：Thread.currentThread().getStackTrace()

#### 3-8 加载应用上下文初始器
利用 Spring 工厂加载机制，实例化 ApplicationContextInitializer 实现类，并排序对象集合。
-  实现
SpringApplication#getSpringFactoriesInstances
```
	private <T> Collection<T> getSpringFactoriesInstances(Class<T> type,
			Class<?>[] parameterTypes, Object... args) {
		ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
		// Use names and ensure unique to protect against duplicates
		Set<String> names = new LinkedHashSet<>(
				SpringFactoriesLoader.loadFactoryNames(type, classLoader));
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
				classLoader, args, names);
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}
```
- 技术
实现类： org.springframework.core.io.support.SpringFactoriesLoader
配置资源： META-INF/spring.factories
排序： AnnotationAwareOrderComparator#sort

```spring.factories
# Initializers
org.springframework.context.ApplicationContextInitializer=\
com.lh.diveinspringboot.context.HelloWorldApplicationContextInitializer,\
com.lh.diveinspringboot.context.AfterHelloWorldApplicationContextInitializer
```
```
@Order(Ordered.HIGHEST_PRECEDENCE)
public class HelloWorldApplicationContextInitializer<C extends ConfigurableApplicationContext>
        implements ApplicationContextInitializer<C> {

    @Override
    public void initialize(C applicationContext) {
        System.out.println("ConfigurableApplicationContext.id = " + applicationContext.getId());
    }

}
```
```
public class AfterHelloWorldApplicationContextInitializer<C extends ConfigurableApplicationContext>
        implements ApplicationContextInitializer<C>, Ordered {

    @Override
    public void initialize(C applicationContext) {
        System.out.println("After ConfigurableApplicationContext.id = " + applicationContext.getId());
    }

    @Override
    public int getOrder() {
        return Ordered.LOWEST_PRECEDENCE;
    }
}
```

#### 3-9 加载应用事件监听器  
ApplicationListener
利用 Spring 工厂加载机制，实例化 ApplicationListener 实现类，并排序对象集合
```
# Application Listeners
org.springframework.context.ApplicationListener=\
com.lh.diveinspringboot.listener.HelloWorldApplicationListener,\
com.lh.diveinspringboot.listener.AfterHelloWorldApplicationListener
```
```
@Order(Ordered.HIGHEST_PRECEDENCE)
public class HelloWorldApplicationListener implements ApplicationListener<ContextRefreshedEvent> {
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        System.out.println("HelloWorld : " + event.getApplicationContext().getId()
                + " , timestamp : " + event.getTimestamp());
    }
}
```
```
public class AfterHelloWorldApplicationListener implements ApplicationListener<ContextRefreshedEvent>,Ordered {
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        System.out.println("AfterHelloWorld : " + event.getApplicationContext().getId()
                + " , timestamp : " + event.getTimestamp());
    }

    @Override
    public int getOrder() {
        return Ordered.LOWEST_PRECEDENCE;
    }
}
```

#### 3-10 SpringApplication 运行阶段
- **加载：SpringApplication 运行监听器（ SpringApplicationRunListeners ）**
利用 Spring 工厂加载机制，读取 SpringApplicationRunListener 对象集合，并且封装到组合类SpringApplicationRunListeners
```
# Run Listeners
org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener
```
- **运行：SpringApplication 运行监听器（ SpringApplicationRunListeners ）**
SpringApplicationRunListener 监听多个运行状态方法：

| 监听方法        | 阶段说明     |  Spring Boot 起始版本  |
| --------   | -----:   | :----: |
| starting()        |  Spring 应用刚启动      |   1.0    |
| environmentPrepared(ConfigurableEnvironment)        | ConfigurableEnvironment 准备妥当，允许将其调整      |   1.0    |
| contextPrepared(ConfigurableApplicationContext)        | ConfigurableApplicationContext 准备妥当，允许将其调整      |   1.0    |
| contextLoaded(ConfigurableApplicationContext)         | ConfigurableApplicationContext 已装载，但仍未启动      |   1.0    |
| started(ConfigurableApplicationContext)        | ConfigurableApplicationContext 已启动，此时 Spring Bean 已初始化完成      |   2.0    |
| running(ConfigurableApplicationContext)        | Spring 应用正在运行      |   2.0    |
| failed(ConfigurableApplicationContext,Throwable)         | Spring 应用运行失败      |   2.0    |

- **监听：Spring Boot事件、Spring事件**
Spring Boot 通过 SpringApplicationRunListener 的实现类EventPublishingRunListener 利用 Spring Framework 事件API ，广播 Spring Boot 事件。
- **创建：应用上下文、Environment、其他（不重要）**
- **失败：故障分析报告**
- **回调：CommandLineRunner、ApplicationRunner**

#### 3-11 SpringApplication 运行事件/监听器编程模型
- Spring 应用事件
普通应用事件： ApplicationEvent
应用上下文事件： ApplicationContextEvent
- Spring 应用监听器
接口编程模型： ApplicationListener
注解编程模型： @EventListener
- Spring 应用事广播器
接口： ApplicationEventMulticaster
实现类： SimpleApplicationEventMulticaster 
-- 执行模式：同步或异步

SpirngFramework实现方式：
```
public class SpringApplicationEventBootstrap {

    public static void main(String[] args) {
        // 创建上下文
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        // 注册应用事件监听器
        context.addApplicationListener(event -> System.out.println("监听到事件: " + event));
        // 启动上下文
        context.refresh();
        // 发送事件
        context.publishEvent("HelloWorld");
        context.publishEvent("2018");
        context.publishEvent(new ApplicationEvent("幺蛾子") {
        });
        // 关闭上下文
        context.close();
    }
}
```

#### 3-12 SpringApplication 运行监听器
```spring.factories
# Run Listeners
org.springframework.boot.SpringApplicationRunListener=\
com.lh.diveinspringboot.run.HelloWorldRunListener
```
```
public class HelloWorldRunListener implements SpringApplicationRunListener {

    public HelloWorldRunListener(SpringApplication application, String[] args) {

    }

    @Override
    public void starting() {
        System.out.println("HelloWorldRunListener.starting()...");
    }

    @Override
    public void environmentPrepared(ConfigurableEnvironment environment) {

    }

    @Override
    public void contextPrepared(ConfigurableApplicationContext context) {

    }

    @Override
    public void contextLoaded(ConfigurableApplicationContext context) {

    }

    @Override
    public void started(ConfigurableApplicationContext context) {

    }

    @Override
    public void running(ConfigurableApplicationContext context) {

    }

    @Override
    public void failed(ConfigurableApplicationContext context, Throwable exception) {

    }

}
```

#### 3-13 监听 Spring Boot 事件
EventPublishingRunListener 监听方法与 Spring Boot 事件对应关系
| 监听方法        | Spring Boot 事件     |  Spring Boot 起始版本  |
| --------   | -----:   | :----: |
| starting()        |  ApplicationStartingEvent      |   1.5    |
| environmentPrepared(ConfigurableEnvironment)         |  ApplicationEnvironmentPreparedEvent       |   1.0    |
| contextPrepared(ConfigurableApplicationContext)        |        |       |
| contextLoaded(ConfigurableApplicationContext)        |   ApplicationPreparedEvent       |   1.0    |
| started(ConfigurableApplicationContext)         |   ApplicationStartedEvent       |   2.0    |
| running(ConfigurableApplicationContext)         |   ApplicationReadyEvent       |   2.0    |
| failed(ConfigurableApplicationContext,Throwable)        |   ApplicationFailedEvent       |   1.0    |

看源码：
```
# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.ClearCachesApplicationListener,\
org.springframework.boot.builder.ParentContextCloserApplicationListener,\
org.springframework.boot.context.FileEncodingApplicationListener,\
org.springframework.boot.context.config.AnsiOutputApplicationListener,\
org.springframework.boot.context.config.ConfigFileApplicationListener,\
org.springframework.boot.context.config.DelegatingApplicationListener,\
org.springframework.boot.context.logging.ClasspathLoggingApplicationListener,\
org.springframework.boot.context.logging.LoggingApplicationListener,\
org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener
```
```ConfigFileApplicationListener
public class ConfigFileApplicationListener
		implements EnvironmentPostProcessor, SmartApplicationListener, Ordered {
}
```
自己实现：
```application.properties
name = Original intention and persistence
```
```spring.factories
# Application Listeners
org.springframework.context.ApplicationListener=\
com.lh.diveinspringboot.listener.BeforeConfigFileApplicationListener
```
```BeforeConfigFileApplicationListener
public class BeforeConfigFileApplicationListener implements SmartApplicationListener, Ordered {
    @Override
    public boolean supportsEventType(Class<? extends ApplicationEvent> eventType) {
        return ApplicationEnvironmentPreparedEvent.class.isAssignableFrom(eventType)
                || ApplicationPreparedEvent.class.isAssignableFrom(eventType);
    }

    @Override
    public boolean supportsSourceType(Class<?> aClass) {
        return true;
    }

    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        if (event instanceof ApplicationEnvironmentPreparedEvent) {
            ApplicationEnvironmentPreparedEvent preparedEvent = (ApplicationEnvironmentPreparedEvent) event;
            Environment environment = preparedEvent.getEnvironment();
            System.out.println("environment.getProperty(\"name\") : " + environment.getProperty("name"));
        }
        if (event instanceof ApplicationPreparedEvent) {

        }
    }

    @Override
    public int getOrder() {
        return ConfigFileApplicationListener.DEFAULT_ORDER + 1;
    }
}
```

#### 3-14 创建 Spring 应用上下文
**创建 Spring 应用上下文（ ConfigurableApplicationContext ）**
根据准备阶段的推断 Web 应用类型创建对应的 ConfigurableApplicationContext 实例：
- Web Reactive： AnnotationConfigReactiveWebServerApplicationContext
- Web Servlet： AnnotationConfigServletWebServerApplicationContext
- 非 Web： AnnotationConfigApplicationContext
```
	protected ConfigurableApplicationContext createApplicationContext() {
		Class<?> contextClass = this.applicationContextClass;
		if (contextClass == null) {
			try {
				switch (this.webApplicationType) {
				case SERVLET:
					contextClass = Class.forName(DEFAULT_WEB_CONTEXT_CLASS);
					break;
				case REACTIVE:
					contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
					break;
				default:
					contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
				}
			}
			catch (ClassNotFoundException ex) {
				throw new IllegalStateException(
						"Unable create a default ApplicationContext, "
								+ "please specify an ApplicationContextClass",
						ex);
			}
		}
		return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
	}
```
**创建 Environment**
根据准备阶段的推断 Web 应用类型创建对应的 ConfigurableEnvironment 实例：
- Web Reactive： StandardEnvironment
- Web Servlet： StandardServletEnvironment
- 非 Web： StandardEnvironment
```
	private ConfigurableEnvironment getOrCreateEnvironment() {
		if (this.environment != null) {
			return this.environment;
		}
		switch (this.webApplicationType) {
		case SERVLET:
			return new StandardServletEnvironment();
		case REACTIVE:
			return new StandardReactiveWebEnvironment();
		default:
			return new StandardEnvironment();
		}
	}
```
测试代码：
```
@SpringBootApplication
public class SpringApplicationContextBootstrap {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = new SpringApplicationBuilder(SpringApplicationContextBootstrap.class)
                .web(WebApplicationType.NONE)
                .run(args);
        System.out.println("ConfigurableApplicationContext 类型：" + context.getClass().getName());
        System.out.println("Environment 类型：" + context.getEnvironment().getClass().getName());
        context.close();
    }
}
```

## 第4章 Web MVC 核心
#### 4-1 Web MVC 核心 - 开场白
- 理解Spring Web MVC架构
- 认识Spring Web MVC
- 简化Spring Web MVC

#### 4-2 理解 Spring Web MVC 架构
- 基础架构：Servlet
![image.png](https://upload-images.jianshu.io/upload_images/1956963-215c079788e3c0e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- Servlet特点
请求/响应式（Request/Response）
屏蔽网络通讯的细节
完整生命周期
- Servlet职责
处理请求
资源管理
视图渲染
- 核心架构：前段控制器（Front Controller）
![image.png](https://upload-images.jianshu.io/upload_images/1956963-4eadb604f18f34b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
资源：http://www.corej2eepatterns.com/FrontController.htm
实现：Spring Web MVC DispatcherServlet
https://docs.spring.io/spring/docs/1.0.0/javadoc-api/org/springframework/web/servlet/DispatcherServlet.html
- Spring Web MVC 架构
![image.png](https://upload-images.jianshu.io/upload_images/1956963-9f69d7efe3297a5b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4-3 Spring Framework 时代的一般认识
- 实现Controller
```
@Controller
public class HelloWorldController {

    @RequestMapping("")
    public String index() {
        return "index";
    }
    
}
```
- 配置Web MVC 组件
```app-context.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.imooc.web"/>

    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>

    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>

    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

</beans>
```
- 部署DispatcherServlet
```
<web-app>
    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/app-context.xml</param-value>
        </init-param>
    </servlet>

    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```
- 使用可执行 Tomcat Maven 插件
```
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.1</version>
                <executions>
                    <execution>
                        <id>tomcat-run</id>
                        <goals>
                            <goal>exec-war-only</goal>
                        </goals>
                        <phase>package</phase>
                        <configuration>
                            <!-- ServletContext path -->
                            <path>/</path>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```
注意：
```
    <packaging>war</packaging>
```
```
    <dependencies>
        <!-- Servlet 3.1 API 依赖-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
        </dependency>

        <!-- Spring Web MVC 依赖 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
        </dependency>
    </dependencies>
```
测试：
终端切换到diveinspringboot\spring-webmvc路径，运行命令：
```
mvn -Dmaven.test.skip -U clean package # 打包
java -jar target/spring-webmvc-0.1-SNAPSHOT-war-exec.jar # 运行
java -jar target/spring-webmvc-0.1-SNAPSHOT-war-exec.jar --httpPort=8070 # 修改端口
nohup java -jar targer/spring-webmvc-0.1-SNAPSHOT-war-exec.jar # 后台挂起运行

可以在IntelliJ IDEA中使用Remote方式连接测试：
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005
```

#### 4-4 Spring Framework 时代的重新认识
- Web MVC 核心组件
**处理器管理**
映射：HandlerMapping
适配器：HandlerAdapter
执行：HandlerExecutionChain
**页面渲染**
视图解析：ViewResolver
国际化：LocaleResolver、LocaleContextResolver
个性化：ThemeResolver
**异常处理**
异常解析：HandlerExceptionResolver
- Web MVC 注解驱动
- Web MVC 自动装配
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-f129ba117ddf05e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 交互流程
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-1871b1dc428e2f75.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4-5 核心组件流程说明
debugger 方式了解流程

#### 4-6 Web MVC 注解驱动
- 注解配置：@Configuration（Spring模式注解）
- 组件激活：@EnableWebMvc（Spriing模块装配）
- 自定义组件：WebMvcConfigurer（Spring Bean）

从xml方式变成bean方式：
```
@Configuration
@EnableWebMvc
public class WebMvcConfig implements WebMvcConfigurer {

//    <!--<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">-->
//        <!--<property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>-->
//        <!--<property name="prefix" value="/WEB-INF/jsp/"/>-->
//        <!--<property name="suffix" value=".jsp"/>-->
//    <!--</bean>-->
    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setViewClass(JstlView.class);
        viewResolver.setPrefix("/WEB-INF/jsp/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }

}
```

#### 4-7 Web MVC 模块组件说明
@EnableWebMvc源码讲解

#### 4-8 WebMvcConfigurer 注入过程
WebMvcConfigurer 源码讲解
```
@Configuration
@EnableWebMvc
public class WebMvcConfig implements WebMvcConfigurer {
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new HandlerInterceptor() {
            @Override
            public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
                System.out.println("拦截中...");
                return true;
            }
        });
    }

}
```

#### 4-9 Web MVC 常用注解（上）
**Web MVC 注解驱动**
- 模型属性：@ModelAttribute
- 请求头：@RequestHeader
- Cookie：@CookieValue
```
@Controller
public class HelloWorldController {

    @RequestMapping("")
    public String index() {
        return "index";
    }

    @RequestMapping("test")
    public String indexTest(@RequestHeader("Accept-Language") String acceptLanguage,
                            @CookieValue("JSESSIONID") String jsessionId,
                            Model model) {
        model.addAttribute("acceptLanguage", acceptLanguage);
        model.addAttribute("jsessionId", jsessionId);
        model.addAttribute("message", "Hello,World");
        return "index";
    }

    @RequestMapping("error")
    public String indexError(@RequestParam int value,
                             Model model) {
        model.addAttribute("message", "Hello,World");
        return "index";
    }

}
```

#### 4-10 Web MVC 常用注解（下）
- 校验参数：@Valid、@Validated
- 注解处理：@ExceptionHandler
- 切面通知：@ControllerAdvice
```
@ControllerAdvice(assignableTypes = HelloWorldController.class)
public class HelloWorldControllerAdvice {

    @ModelAttribute("acceptLanguage")
    public String acceptLanguage(@RequestHeader("Accept-Language") String acceptLanguage) {
        return acceptLanguage;
    }

    @ModelAttribute("jsessionId")
    public String jsessionId(@CookieValue("JSESSIONID") String jsessionId) {
        return jsessionId;
    }

    @ModelAttribute("message")
    public String message() {
        return "Hello,World 2018";
    }

    @ExceptionHandler(Throwable.class)
    public ResponseEntity<String> onException(Throwable throwable) {
        return ResponseEntity.ok(throwable.getMessage());
    }

}
```

#### 4-11 Web MVC 自动装配
- Servlet依赖：Servlet 3.0+
- Servlet API：ServletContainerInitializer
- Spring适配：SpringServletContainerInitializer
- Spring API：WebApplicationInitializer
- 编程驱动：AbstractDispatcherServletInitializer
- 注解驱动：AbstractAnnotationConfigDispatcherInitializer

SpringServletContainerInitializer#onStartup方法在启动时会回调
源码介绍

#### 4-12 Web MVC 自动装配实现
抛弃web.xml：
```
public class DefaultAnnotationConfigDispatcherServletInitializer extends
        AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[0];
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{DispatcherServletConfiguration.class};
    }

    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}
```
```
@ComponentScan(basePackages = "com.lh.diveinspringboot")
public class DispatcherServletConfiguration {
}
```

#### 4-13 Spring Boot 时代的简化
- 完全自动装配
- 条件装配
- 外部化配置

#### 4-14 完全自动装配
- 自动装配DispatcherServlet：DispatcherServletAutoConfiguration
- 替换@EnableWebMvc：WebMvcAutoConfiguration
- Servlet容器：ServletWebServerFactoryAutoConfiguration

**理解自动装配顺序性**
绝对顺序： @AutoConfigureOrder
相对顺序： @AutoConfigureAfter

#### 4-15 条件装配
- Web类型（ConditionalOnWebApplication ）：
Servlet
- API依赖（ @ConditionalOnClass ）：
Servlet、Spring Web MVC
- Bean依赖（ @ConditionalOnMissingBean 、 @ConditionalOnBean ）：
WebMvcConfigutationSupport

#### 4-16 外部化配置
- Web MVC 配置：WebMvcProperties
- 资源配置：ResourceProperties

#### 4-18 重构 Spring Web MVC 项目
- springboot-webmvc项目
Spring Boot JSP 依赖
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
        </dependency>
```
Spring Boot Maven plugin能够将Spring Boot应用打包为可执行的jar或war文件，然后以通常的方式运行Spring Boot应用。
```
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```
WebMvcProperties$View
```application.properties
spring.mvc.view.prefix = /WEB-INF/jsp/
spring.mvc.view.suffix = .jsp
```
@EnableAutoConfiguration 和 @EnableWebMvc 冲突：
```
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
public class WebMvcAutoConfiguration {
}
```

## 第5章 Web MVC 视图应用
#### 5-1 Web MVC 视图应用
- 模板引擎
- 视图处理
- 视图内容协商
- 视图组件自动装配

#### 5-2 新一代服务端模板引擎Thymeleaf语法和核心要素
- Thymeleaf 依赖
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```
- Thymeleaf 语法
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
    <head>
        <title>Good Thymes Virtual Grocery</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
        <link rel="stylesheet" type="text/css" media="all"
              href="../../css/gtvg.css" th:href="@{/css/gtvg.css}"/>
    </head>
    <body>
        <p th:text="#{home.welcome}">Welcome to our grocery store!</p>
    </body>
</html>
```
- 核心要素
资源定位：模板来源
渲染上下文：变量来源
模板引擎：模板渲染

#### 5-3 Thymeleaf 示例
- 示例：使用Thymeleaf API渲染内容
```
public class ThymeleafTemplateEngineBootstrap {

    public static void main(String[] args) {
        // 构建引擎
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        // 创建渲染上下文
        Context context = new Context();
        context.setVariable("message", "Hello,World");
        // 模板的内容
        String content = "<p th:text=\"${message}\">!!!</p>";
        // 渲染（处理）结果
        String result = templateEngine.process(content, context);
        // 输出渲染（处理）结果
        System.out.println(result);
    }

}
```
- 示例：Thymeleaf 与 Spring 资源整合
```hello-world.html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
    <body>
        <p th:text="${message}">!!!</p>
    </body>
</html>
```
```application.properties
# Thymeleaf 配置
spring.thymeleaf.prefix = classpath:/templates/thymeleaf/
spring.thymeleaf.suffix = .html
## 取消缓存
spring.thymeleaf.cache = false
```
```
public class ThymeleafTemplateEngineBootstrap {

    public static void main(String[] args) throws IOException {
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        Context context = new Context();
        context.setVariable("message", "Hello,World");

        ResourceLoader resourceLoader = new DefaultResourceLoader();
        Resource resource = resourceLoader.getResource("classpath:/templates/thymeleaf/hello-world.html");
        File templateFile = resource.getFile();

        FileInputStream inputStream = new FileInputStream(templateFile);
        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
        IOUtils.copy(inputStream, outputStream);
        inputStream.close();

        String content = outputStream.toString("UTF-8");
        String result = templateEngine.process(content, context);
        System.out.println(result);
    }

}
```

#### 5-4 ThymeleafViewResolver和多ViewResolver处理流程
- Spring Web MVC 视图组件
视图解析器：ViewResolver
视图：View
总控：DispatcherServlet
- Thymeleaf 整合 Spring Web MVC
视图解析器：ThymeleafViewResolver
视图：ThymeleafView
渲染：SpringTemplateEngine
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-a94cde721b21eb05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
源码：DispatcherServlet#resolveViewName

#### 5-5 ThymeleafViewResolver 示例
- 示例：多视图处理器并存
看大纲
代码：
```
@Controller
public class HelloWorldController {

    @GetMapping("/hello-world")
    public String helloWorld() {
        return "hello-world"; // View 逻辑名称
    }

    @ModelAttribute("message")
    public String message() {
        return "HelloWorld";
    }

}
```
可以在ThymeleafProperties查看配置：
```application.properties
# Thymeleaf 配置
spring.thymeleaf.prefix = classpath:/templates/thymeleaf/
spring.thymeleaf.suffix = .html
## 取消缓存
spring.thymeleaf.cache = false
```
```
@SpringBootApplication(scanBasePackages = "com.lh.diveinspringboot")
public class SpringBootViewBootstrap {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootViewBootstrap.class);
    }

}
```

#### 5-6 整合InternalResourceViewResolver示例
实现thymeleaf和jsp混搭：
```
    <packaging>war</packaging>
```
从springboot-webmvc项目拷贝jsp代码和配置。
ThymeleafViewResolver Find Usages查看：
```
			@Bean
			@ConditionalOnMissingBean(name = "thymeleafViewResolver")
			public ThymeleafViewResolver thymeleafViewResolver() {
				ThymeleafViewResolver resolver = new ThymeleafViewResolver();
				resolver.setOrder(Ordered.LOWEST_PRECEDENCE - 5);
				return resolver;
			}
```
InternalResourceViewResolver Find Usages查看：
```
	private int order = Ordered.LOWEST_PRECEDENCE;
```
order值越大，优先级越低。 InternalResourceViewResolver的优先级比ThymeleafViewResolver优先级低。所以需要重新设置：
```
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Bean
    public ViewResolver viewResolver(){
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setViewClass(JstlView.class);
        viewResolver.setPrefix("/WEB-INF/jsp/");
        viewResolver.setSuffix(".jsp");
        viewResolver.setOrder(Ordered.LOWEST_PRECEDENCE - 10);
        viewResolver.setContentType("text/xml;charset=UTF-8");
        return viewResolver;
    }

}
```
Debug DispatcherServlet#resolveViewName
WebMvcConfig

#### 5-7 修复 Maven 多模块 JSP 定位问题 示例
JspServlet#serviceJspFile 断点调试
customizer方法修复：
```
    @Bean
    public WebServerFactoryCustomizer<TomcatServletWebServerFactory> customizer() {
        return factory -> factory.addContextCustomizers((context) -> {
                    String relativePath = "springboot-view/src/main/webapp";
                    // 相对于 user.dir = D:\workspace\dive-in-spring-boot
                    File docBaseFile = new File(relativePath);
                    if (docBaseFile.exists()) { // 路径是否存在
                        // 解决 Maven 多模块 JSP 无法读取的问题
                        context.setDocBase(docBaseFile.getAbsolutePath());
                    }
                }
        );
    }
```

#### 5-8 视图内容协商
Web客户端根据通过不同的请求策略，实现服务端响应对应视图内容输出。
- 核心组件
视图解析器：ContentNegotiatingViewResolver
内容协商管理器：ContentNegotiationManager
内容协商策略：ContentNegotiationStrategy
- ContentNegotiatingViewResolver
InternalResourceViewResolver、BeanNameViewResolver、ThymeleafViewResolver
关联ViewResolver Bean列表
关联ContentNegotiationManager Bean
解析最佳匹配View
- 策略管理 ContentNegotiationManager
由ContentNegotiationConfigurer 对象配置，配置Bean：WebMvcConfigurer
由ContentNegotiationManagerFactoryBean 创建
关联ContentNegotiationStrategy集合

![图片.png](https://upload-images.jianshu.io/upload_images/1956963-3c7f09c0a2c318e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 5-9 视图内容协商代码分析
WebMvcConfigurer#configureContentNegotiation
根据大纲图片流程分析源码

#### 5-10 ViewResolver 冲突说明部分
端点查看ViewResolver.class 名称
this.webApplicationContext.getBeansOfType(ViewResolver.class)

WebMvcAutoConfiguration#viewResolver
```
		@Bean
		@ConditionalOnBean(ViewResolver.class)
		@ConditionalOnMissingBean(name = "viewResolver", value = ContentNegotiatingViewResolver.class)
		public ContentNegotiatingViewResolver viewResolver(BeanFactory beanFactory) {
			ContentNegotiatingViewResolver resolver = new ContentNegotiatingViewResolver();
			resolver.setContentNegotiationManager(
					beanFactory.getBean(ContentNegotiationManager.class));
			// ContentNegotiatingViewResolver uses all the other view resolvers to locate
			// a view so it should have a high precedence
			resolver.setOrder(Ordered.HIGHEST_PRECEDENCE);
			return resolver;
		}
```
WebMvcAutoConfiguration#defaultViewResolver
```
		@Bean
		@ConditionalOnMissingBean
		public InternalResourceViewResolver defaultViewResolver() {
			InternalResourceViewResolver resolver = new InternalResourceViewResolver();
			resolver.setPrefix(this.mvcProperties.getView().getPrefix());
			resolver.setSuffix(this.mvcProperties.getView().getSuffix());
			return resolver;
		}
```

#### 5-11 ViewResolver 内容协商原理
- 示例：多视图处理器内容协商 见大纲
ContentNegotiatingViewResolver#getBestView
```
	@Nullable
	private View getBestView(List<View> candidateViews, List<MediaType> requestedMediaTypes, RequestAttributes attrs) {
		for (View candidateView : candidateViews) {
			if (candidateView instanceof SmartView) {
				SmartView smartView = (SmartView) candidateView;
				if (smartView.isRedirectView()) {
					if (logger.isDebugEnabled()) {
						logger.debug("Returning redirect view [" + candidateView + "]");
					}
					return candidateView;
				}
			}
		}
		for (MediaType mediaType : requestedMediaTypes) {
			for (View candidateView : candidateViews) {
				if (StringUtils.hasText(candidateView.getContentType())) {
					MediaType candidateContentType = MediaType.parseMediaType(candidateView.getContentType());
					if (mediaType.isCompatibleWith(candidateContentType)) {
						if (logger.isDebugEnabled()) {
							logger.debug("Returning [" + candidateView + "] based on requested media type '" +
									mediaType + "'");
						}
						attrs.setAttribute(View.SELECTED_CONTENT_TYPE, mediaType, RequestAttributes.SCOPE_REQUEST);
						return candidateView;
					}
				}
			}
		}
		return null;
	}
```
代码：
```
    viewResolver.setContentType("text/xml;charset=UTF-8");
```
```
    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer.favorParameter(true).favorPathExtension(true);
    }
```
测试：
```
postman：
http://localhost:8080
headers:
Accept:text/html
```

#### 5-12 Web MVC 视图应用总结
**视图组件自动装配**
- 自动装配
Web MVC：WebMvcAutoConfiguration
Thymeleaf Web MVC：ThymeleafWebMvcConfiguration
Thymeleaf Web Flux：ThymeleafWebFluxConfiguration
- 外部化配置
内容协商：WebMvcProperties.Contentnegotiation
Thymeleaf：ThymeleafProperties

