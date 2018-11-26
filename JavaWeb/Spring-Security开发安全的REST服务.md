## 第1章 课程导学
#### 1-1 导学
- 企业级的认证和授权
同时支持多种认证方式
同时支持多种前端渠道：浏览器和APP
支持集群环境，跨应用工作，Session控制，控制用户权限，防护与身份认证相关的攻击
- Spring Security
- Spring Social
- Spring Security OAuth

## 第2章 开始开发
#### 2-2 代码结构介绍
- 代码结构
imooc-security：主模块
imooc-security-core：核心业务逻辑
imooc-security-browser：浏览器安全特定代码
imooc-security-app：app相关特定代码
imooc-security-demo：样例程序
```
新建主项目imooc-security
配置主项目pom，依赖和编译插件配置
新建四个module项目，配置各自的pom
```

#### 2-3 Hello Spring Security
```
DemoApplication#hello 接口
启动报错，需要配置数据库连接，还需要增加如下配置：
#集群session存储方式
spring.session.store-type = none
security.basic.enable = false
访问/hello接口测试

mvn clean package -DskipTests 打包
demo项目pom文件添加spring-boot-maven-plugin打包插件，可以打出可执行jar包
tomcat内嵌在可执行jar中，不需要额外的tomcat
在命令行中可通过：java -jar demo.jar 运行程序
```

## 第3章 使用Spring MVC开发RESTful API
#### 3-1 Restful简介
使用Spring MVC编写Restful API
使用Spring MVC处理其它web应用常见的需求和场景
Restful API开发常用辅助框架
- Restful API第一印象
传统写法和REST风格对比
![image.png](https://upload-images.jianshu.io/upload_images/1956963-7ab2b77c5a4a71d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
1. 用URL描述资源
2. 使用HTTP方法描述行为，使用HTTP状态码来表示不同的结果
3. 使用json交互数据
4. RESTful只是一种风格，并不是强制的标准

#### 3-2 查询请求
- 编写第一个Restful API
```
demo项目pom文件引入测试框架：spring-boot-starter-test
test包下UserControllerTest
```
- 编写针对Restful API的测试用例
@RestController 标明此Controller 提供RestAPI
@RequestMapping及其变体。映射http请求url到java方法
@RequestParam 映射请求参数到java方法的参数
@PageableDefault 指定分页参数默认值
- 在Restful API中传递参数
```
@RequestParam 可以设置请求参数名称
可以直接使用对象来接收请求参数，比如UserQueryCondition
可以使用Pageable来接收分页请求参数
可以使用@PageableDefault来指定分页请求参数默认值
代码见UserController#query，使用UserControllerTest#whenQuerySuccess 测试

github上JsonPath项目文档具体介绍了JsonPath表达式语法
```

#### 3-3 用户详情请求
- 编写用户详情服务
@PathVariable映射url片段到java方法的参数
在url声明中使用正则表达式
@JsonView控制json输出内容
- @JsonView使用步骤
使用接口来声明多个视图
在值对象的get方法上指定视图
在Controller方法上指定视图
```
UserController#getInfo 演示@PathVariable
/user/{id:\d+} 正则表达式限制id只能输入数字
UserControllerTest#whenGetInfoFail 测试正则表达式

按照@JsonView使用步骤，给User添加UserSimpleView和UserDetailView视图，并在get方法指定视图，在Controller方法上指定视图，并测试返回数据

测试用例可以保证你重构代码之后，你的程序还是按照期望的结果去运行
```

#### 3-4 用户创建请求
- 处理创建请求
@RequestBody 映射请求体到java方法的参数
日期类型参数的处理
@Valid 注解 BindingResult 验证请求参数的合法性并处理校验结果
```
UserController#create 演示@RequestBody，使用UserControllerTest#whenCreateSuccess 测试

日期类型参数在传递过程中一般都是使用long类型的时间戳

User对象上增加@NotBlank，在UserController#create方法参数中添加@Valid，实现参数校验
BindingResult 使校验失败，还能带着错误信息进入Controller的方法体中，见UserController#create方法演示
```

#### 3-5 修改和删除请求
- 开发用户信息修改和删除服务
常用的验证注解
自定义消息
自定义校验注解
![image.png](https://upload-images.jianshu.io/upload_images/1956963-3e3c6ea103123c97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-7585cded5211beee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
每次先跑测试用例是为了测试用例代码能正常运行
使用 UserControllerTest#whenUpdateSuccess 测试 UserController#update 方法关于@Past参数校验
@NotBlank(message = "密码不能为空")
@Past(message = "生日必须是过去的时间")
每一个注解都可以配置提示消息

自定义校验注解
@MyConstraint 注解指定一个校验器类MyConstraintValidator，其中isValid方法进行校验，在User类上使用@MyConstraint 注解，就实现了校验逻辑，可以使用UserControllerTest#whenUpdateSuccess 进行测试

UserControllerTest#whenDeleteSuccess 测试 UserController#delete 删除逻辑
```

#### 3-6 服务异常处理
- RESTful API 错误处理
Spring Boot 中默认的错误处理机制
自定义异常处理
```
BasicErrorController 是Spring做默认错误处理的控制器，请求头上带了text/html，会走errorHtml方法返回一段html，没带会走error方法返回一个map

Controller方法，如果@Valid校验不通过，且方法中没有BindingResult errors这个字段，PostMan访问接口会得到错误信息，可以使用UserController#create方法测试
如果代码中抛出异常，也会走Spring默认的处理逻辑，可以使用UserController#getInfo方法测试

如何定义浏览器请求错误页面：resources\public\error\404.html 等页面会根据状态码被返回
如何定义客户端请求错误信息：代码中抛出自定义异常，比如UserNotExistException，编写ControllerExceptionHandler处理异常信息
```

#### 3-7 使用切片拦截REST服务
- RESTful API的拦截
过滤器（Filter）
拦截器（Interceptor）
切片（Aspect）
需求：记录所有服务的处理时间
```
编写TimeFilter，调用getInfo方法测试

如何将第三方没有@Component注解的Filter加入到项目？
TimeFilter注释掉@Component注解，当作是第三方Filter，spring项目是有一个web.xml可以配置Filter，springboot项目可以新建一个配置类WebConfig，提供FilterRegistrationBean来配置Filter，作用和web.xml配置Filter一样

Filter是j2ee规范定义的，只能拿到request和response，不能知道哪个控制器的哪个方法处理，Interceptor是spring框架提供，可以知道控制器和方法

编写TimeInterceptor，注意需要配置WebConfig extends WebMvcConfigurerAdapter才能生效，重写addInterceptors方法，注册拦截器才能生效，调用getInfo方法测试，如果getInfo方法抛出异常，会在TimeInterceptor#afterCompletion方法获取到异常，注意@ControllerAdvice注解的ControllerExceptionHandler处理异常在TimeInterceptor之前，如果有ControllerExceptionHandler，TimeInterceptor#afterCompletion方法获取到异常依然是null
拦截器不仅会拦截自己写的Controller，spring的Controller也会被拦截，比如BasicErrorController
```

#### 3-8 使用Filter和Interceptor拦截REST服务
![image.png](https://upload-images.jianshu.io/upload_images/1956963-b0f57433377b52d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
拦截器无法获取控制器方法参数值，想知道参数，只能使用切片
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
编写TimeAspect实现一个简单的切片，测试
```
- RESTful API 的拦截顺序
![image.png](https://upload-images.jianshu.io/upload_images/1956963-c2f8c741f0b0a9d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 3-9 使用REST方式处理文件服务
- 文件的上传和下载
文件上传
文件下载
```
在FileController编写upload和download方法，完成文件的上传和下载
可以使用UserControllerTest#whenUploadSuccess方法测试文件上传，浏览器测试文件下载
```

#### 3-10 使用多线程提高REST服务性能
- 异步处理REST服务
使用Runnable异步处理Rest服务
使用DeferredResult异步处理Rest服务
异步处理配置
![image.png](https://upload-images.jianshu.io/upload_images/1956963-e4067ba3207666f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
使用异步使服务器吞吐量得到极大的提升
```
AsyncController#asyncMain 同步演示

AsyncController#asyncCallable 使用Runnable异步演示
使用Runnable异步处理不能处理所有的场景，其副线程必须由主线程调起，下图场景就不能使用Runnable异步处理完成
```
![image.png](https://upload-images.jianshu.io/upload_images/1956963-8e8f688031c34b6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
MockQueue虚拟队列编写
DeferredResultHolder：在线程1和线程2之间传递DeferredResult对象
QueueListener：监听虚拟消息队列
AsyncController#asyncDeferredResult：用DeferredResult异步处理Rest服务
上面这四个类模拟了上图这种异步处理场景，代码很简单

WebConfig#configureAsyncSupport可以对异步请求拦截器进行配置、超时时间配置和线程池配置
```

#### 3-11 使用Swagger自动生成文档
- 与前端开发并行工作
使用swagger自动生成html文档
使用WireMock快速伪造RESTul服务
```
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.7.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.7.0</version>
</dependency>
配置@EnableSwagger2，访问http://localhost:8080/swagger-ui.html可以查看文档
@ApiOperation修饰Controller方法名称
@ApiModelProperty修饰请求对象参数名称
@ApiParam修饰请求路径参数名称
```

#### 3-12 使用WireMock伪造REST服务
WireMock是一个独立的服务器，前端可以连接WireMock服务器
[**WireMock官网**](http://wiremock.org/)
```
访问官方文档，DOC中Running as a Standalone Process可以下载wiremock-standalone-2.19.0.jar
java -jar wiremock-standalone-2.19.0.jar --port 8081 # 启动WireMock

<dependency>
    <groupId>com.github.tomakehurst</groupId>
    <artifactId>wiremock</artifactId>
</dependency>
MockServer编写配置和测试数据，运行，传输数据到WireMock服务器
访问WireMock服务器的方法可以得到模拟数据,比如：http://localhost:8081/order/1
```

## 第4章 使用Spring Security开发基于表单的登录
#### 4-1 简介
- SpringSecurity 核心功能
认证 —— 你是谁
授权 —— 你能干什么
攻击防护 —— 防止伪造身份
- 内容简介
SpringSecurity基本原理
实现用户名+密码认证（默认内建实现）
实现手机号+短信认证（自定义认证方式）

#### 4-2 SpringSecurity基本原理
- 开启SpringSecurity功能
```
之前使用配置security.basic.enable=false关闭了SpringSecurity，现在需要打开
访问任意接口需要http basic身份验证，用户名为user，密码见springboot启动日志
```
- 覆盖http basic身份验证，采用默认表单登录
```
immoc-security-browser项目，希望实现表单登录，添加如下配置类：
@Configuration
public class BrowserSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
//        http.httpBasic() // 默认httpBasic登录
        http.formLogin() // 表单登录
                .and()
                .authorizeRequests()
                .anyRequest()
                .authenticated(); // 任何请求都需要授权
    }
}
访问任意接口会跳转默认实现的登录页面
```
- SpringSecurity基本原理
```
Spring Security 过滤器链是Spring Security 最核心的东西，是一组Filter，每一个过滤器处理一种认证方式，所有的请求和响应都会经过此过滤器链

UsernamePasswordAuthenticationFilter 处理表单登录，尝试登录
BasicAuthenticationFilter  处理http basic登录，尝试登录

FilterSecurityInterceptor 根据配置（如BrowserSecurityConfig）决定请求能不能访问服务，是否抛出异常
ExceptionTranslationFilter 会捕获FilterSecurityInterceptor过滤器抛出的异常，并根据异常做相应处理
注：绿色的过滤器可以通过配置决定是否生效，其他的过滤器是不能控制的，一定生效，位置不能修改
断点调试上述几个过滤器类，体会SpringSecurity过滤器链流程（快捷键：F8下一步 F9恢复程序）
```
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-9acf3f98319d795d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4-3 自定义用户认证逻辑
- 处理用户信息获取逻辑：用户需要从数据库中读取！——UserDetailsService
```
UserDetailsService 接口根据用户名读取用户信息UserDetails
@Component
public class MyUserDetailsService implements UserDetailsService {
    private Logger logger = LoggerFactory.getLogger(getClass());
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // TODO 注入DAO对象，根据用户名从数据库查询用户信息
        logger.info("登录用户名：" + username);
        // 返回UserDetails实现类
        return new User(username, username + "125",
                // 把逗号隔开的字符串转GrantedAuthority集合，拥有哪些权限
                AuthorityUtils.commaSeparatedStringToAuthorityList("admin"));
    }
}
浏览器登录，用户名随意，密码为用户名+125，实际使用需要查询数据库
```
- 处理用户校验逻辑：比如验证用户是否冻结 —— UserDetails
```
密码是否匹配由spring security实现，我们只需要告诉spring security密码是什么

UserDetails 接口封装了登录信息，校验逻辑可以写在这里
isAccountNonExpired（用户是否过期）、isAccountNonLocked（账户是否锁定冻结）、
isCredentialsNonExpired（密码是否过期）、isEnabled（账户可用，是否删除）

UserDetails 接口实现类可以使用用户表的实体对象，然后实现UserDetails 接口，完善校验逻辑

@Component
public class MyUserDetailsService implements UserDetailsService {
    private Logger logger = LoggerFactory.getLogger(getClass());
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // TODO 注入DAO对象，根据用户名从数据库查询用户信息
        logger.info("登录用户名：" + username);
        // 返回UserDetails实现类，可以使用用户表的实体对象实现UserDetails接口
        // TODO 根据查找到的用户信息判断用户是否被冻结、账户是否删除等
        return new User(username, username + "125",
                true, // 账户可用，是否删除
                true, // 用户是否过期
                true, // 密码是否过期
                false,  // 账户是否锁定冻结
                // 把逗号隔开的字符串转GrantedAuthority集合，拥有哪些权限
                AuthorityUtils.commaSeparatedStringToAuthorityList("admin"));
    }
}
```
- 处理密码加密解密——PasswordEncoder
```
crypto.PasswordEncoder 接口处理加密解密
PasswordEncoder#encode方法用于密码加密（供用户调用）
PasswordEncoder#matches方法判断密码是否匹配（供系统调用）

在BrowserSecurityConfig中配置PasswordEncoder
@Bean
public PasswordEncoder passwordEncoder(){
    return new BCryptPasswordEncoder();
}
```

#### 4-4 个性化用户认证流程（一）
- 自定义登录页面
```
@Configuration
public class BrowserSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin() // 表单登录
                .loginPage("/noc-signIn.html") // 自定义登录页面
                .loginProcessingUrl("/authentication/form") // 自定义表单提交请求
                .and()
                .authorizeRequests()
                .antMatchers("/noc-signIn.html").permitAll() // 需要配置自定义登录页面不需要授权
                .anyRequest()
                .authenticated() // 其他任何请求都需要授权
                .and().csrf().disable(); // 关闭跨站防护
    }
}
编写noc-signIn.html，表单提交到/authentication/form
```
- 处理不同类型的请求
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-bbe7fb45b6cc7b59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
期望效果：如上图所示
修改配置：BrowserSecurityConfig#configure中自定义登录页面到/authentication/require接口
编写接口：BrowserSecurityController#requireAuthentication，路径如上，接口处理请求逻辑

可以使用HttpSessionRequestCache获取之前缓存的请求
工具类DefaultRedirectStrategy可以方便实现跳转
```
- 如何自己配置登录页面
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-87a757866fa6ce61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
SecurityProperties对系统所有的配置进行封装
在core项目对中系统配置封装：SecurityProperties、BrowserProperties
SecurityCoreConfig 使配置读取器SecurityProperties生效

demo项目自定义配置：noc.security.browser.loginPage=/demo-signIn.html
browser项目BrowserSecurityController注入SecurityProperties，完善身份认证跳转逻辑
BrowserSecurityConfig也需要配置自定义登录页面不需要授权
```

#### 4-5 个性化用户认证流程（二）
自定义登录成功处理：比如记录登录成功日志，发积分等
```
spring security默认登录成功是跳转到之前的接口

期望：自定义登录成功处理，返回用户信息
需要实现AuthenticationSuccessHandler

browser项目: NocAuthenticationSuccessHandler#onAuthenticationSuccess实现登录成功逻辑
方法参数Authentication 接口封装了认证信息（认证请求信息和UserDetails）
工具类ObjectMapper可以实现json转化
将自定义登录成功处理器配置到BrowserSecurityConfig：.successHandler(myAuthenticationSuccessHandler)
```
自定义登录失败处理：比如记录登录失败日志，今日错误几次等
```
需要实现AuthenticationFailHandler

browser项目: MyAuthenticationFailHandler#onAuthenticationFailure实现登录失败逻辑
将自定义登录失败处理器配置到BrowserSecurityConfig：.failHandler(myAuthenticationFailHandler)
```
- 改造成根据配置来决定登录返回html还是json
```
LoginType枚举、BrowserProperties添加配置loginType
NocAuthenticationSuccessHandler改继承SavedRequestAwareAuthenticationSuccessHandler，修改登陆成功逻辑
NocAuthenctiationFailureHandler改继承 SimpleUrlAuthenticationFailureHandler，修改登陆失败逻辑

SavedRequestAwareAuthenticationSuccessHandler是spring security默认处理器，默认请求成功跳转之前缓存的页面

测试demo配置noc.security.browser.loginType=REDIRECT和JSON时，
访问：http://localhost:8080/index.html，查看登录成功和登录失败结果
``` 

#### 4-6 认证流程源码级详解
- 认证处理流程说明
![image.png](https://upload-images.jianshu.io/upload_images/1956963-927729d0908a0dc4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
AuthenticationManager负责收集所有的AuthenticationProvider
真正的校验逻辑在AuthenticationProvider中
调用我们实现的UserDetailsService，获取UserDetails
全部校验成功，创建Authentication对象
```
- 认证结果如何在多个请求之间共享？
![image.png](https://upload-images.jianshu.io/upload_images/1956963-0c5cc3cbd461858e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
登录成功之前，把Authentication放入SecurityContext
SecurityContextImpl包装了Authentication
SecurityContextHolder是ThreadLocal的封装，在任何线程都可以读Authentication

SecurityContextPersistenceFilter，
当请求进来检查session中是否有SecurityContext，如果有就拿出来放到线程中，
当请求结束检查线程中是否有SecurityContext，如果有就拿出来放到session中，
这样不同的请求就可以从session中拿到相同的信息
```
![image.png](https://upload-images.jianshu.io/upload_images/1956963-21d566cd563e3f50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 使用SecurityContextHolder获取认证用户信息
```
@GetMapping("/me")
public Object getCurrentUser() {
    return SecurityContextHolder.getContext().getAuthentication();
} 

也可以直接获取Authentication或者UserDetails
@GetMapping("/me/2")
public Object getCurrentUser2(Authentication user) {
    return user;
}

@GetMapping("/me/3")
public Object getCurrentUser3(@AuthenticationPrincipal UserDetails user) {
    return user;
}
```

#### 4-7 图片验证码
- 实现图形验证码功能
开发生成图形验证码接口
在认证流程中加入图形验证码校验
重构代码
- 生成图形验证码
根据随机数生成图片
将随机数存到Session中
再将生成的图片写到接口的响应中
```
core模块
ImageCode 封装图片验证码信息
ValidateCodeController#createCode 完成生成图形验证码三个步骤
工具类 HttpSessionSessionStrategy 可以用来保存session
工具类 ImageIO 方便写图片到输出流

在登陆页面 noc-signIn.html 中添加图形验证码画面
BrowserSecurityConfig#configure配置图形验证码接口不需要授权
访问:http://localhost:8080/noc-signIn.html 查看画面
```
- 验证图形验证码
```
在Spring Security 过滤器链上自定义过滤器，编写自定义校验逻辑
自定义图形验证码过滤器：
public class ValidateCodeFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        // 验证逻辑
        // 调用后面的过滤器
        filterChain.doFilter(request, response);
    }
}
自定义验证码异常：ValidateCodeException
使用自定义登录失败处理器捕获异常：AuthenticationFailureHandler

配置自定义的过滤器到过滤器链上：
BrowserSecurityConfig#configure
@Override
protected void configure(HttpSecurity http) throws Exception {
    ValidateCodeFilter validateCodeFilter = new ValidateCodeFilter();
    validateCodeFilter.setAuthenticationFailureHandler(nocAuthenctiationFailureHandler);
    http.addFilterBefore(validateCodeFilter, UsernamePasswordAuthenticationFilter.class) // 配置自定义的过滤器
            .formLogin() // 表单登录
            .loginPage("/authentication/require") // 自定义登录页面
            .loginProcessingUrl("/authentication/form") // 自定义表单提交请求
            .successHandler(nocAuthenticationSuccessHandler) // 自定义登录成功处理器
            .failureHandler(nocAuthenctiationFailureHandler) // 自定义登录失败处理器
            .and()
            .authorizeRequests()
            .antMatchers("/authentication/require",
                    securityProperties.getBrowser().getLoginPage(),
                    "/code/image").permitAll() // 需要配置自定义登录页面不需要授权
            .anyRequest()
            .authenticated() // 其他任何请求都需要授权
            .and().csrf().disable(); // 关闭跨站防护
}
```

#### 4-8 图片验证码重构
- 重构图形验证码接口
验证码基本参数可配置
验证码拦截的接口可配置
验证码的生成逻辑可配置
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-5485857608ca2d72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 验证码基本参数可配置
```
新建ImageCodeProperties 配置类配置到SecurityProperties中，
中间封装了一层ValidateCodeProperties，考虑到以后还有短信验证码

ValidateCodeController 中注入ImageCodeProperties，使用配置
可以在demo项目中添加自定义配置，测试
```
- 验证码拦截的接口可配置
```
ImageCodeProperties 添加拦截url配置项

ValidateCodeFilter 修改校验逻辑
其中，ValidateCodeFilter 实现InitializingBean接口，在afterPropertiesSet 方法中做url配置初始化
使用工具类AntPathMatcher 做路径匹配
BrowserSecurityConfig#configure 修改ValidateCodeFilter 初始化逻辑

demo项目中添加自定义验证码拦截路径，测试
```
- 验证码的生成逻辑可配置
```
想把一段逻辑变成可配置的，需要声明接口让别人覆盖我们的逻辑
提供接口：ValidateCodeGenerator
提供接口的实现类：ImageCodeGenerator
把ValidateCodeController 关于生成验证码图片的业务逻辑挪到 ImageCodeGenerator中
ValidateCodeController 中注入ValidateCodeGenerator 来使用该逻辑
ValidateCodeBeanConfig 配置类初始化 ValidateCodeGenerator
@ConditionalOnMissingBean(name="imageCodeGenerator") 如果spring容器已经有imageCodeGenerator，就不用此Bean

demo项目DemoImageCodeGenerator 实现ValidateCodeGenerator ，配置Bean名字为imageCodeGenerator交给spring管理
测试，会走demo的生成图形验证码的逻辑

开发思想：以增量的方式实现变化，很重要的开发技巧
```

#### 4-9 添加记住我功能
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-afd5023a284f4408.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-0408e76af7d14784.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 记住我功能具体实现
```
在noc-signIn.html 页面增加记住我CheckBox

在BrowserSecurityConfig 中配置 TokenRepository，用来读写数据库
TokenRepository 需要数据源，demo项目 application.properties 配置数据源
JdbcTokenRepositoryImpl.CREATE_TABLE_SQL 有创建表的脚本，
也可以使用 tokenRepository.setCreateTableOnStartup(true); 自动创建表

BrowserProperties新增配置rememberMeSeconds：记住我功能有效时间

BrowserSecurityConfig#configure 编写上述三个配置
.and()
.rememberMe()
.tokenRepository(persistentTokenRepository())
.tokenValiditySeconds(securityProperties.getBrowser().getRememberMeSeconds())
.userDetailsService(userDetailsService)
其中UserDetailsService在之前已经实现过了

测试，可以看到数据库persistent_logins表存了token信息，重启服务器，不需要重新登录
```
- 记住我功能SpringSecurity源码解析
依照记住我基本原理图，断点调试

#### 4-10 短信验证码接口开发
- 实现短信验证码登录
开发短信验证码接口
校验短信验证码并登录
重构代码
- 开发短信验证码接口
```
ValidateCodeController#createSmsCode 发送短信验证码接口
ValidateCode 封装图片验证码信息，改造ImageCode 继承ValidateCode 
不同的应用使用不同的短信供应商，封装短信验证码发送接口：
SmsCodeSender#send 接口
DefaultSmsCodeSender#send 提供默认实现
ValidateCodeBeanConfig 配置 smsCodeSender
ValidateCodeController 中注入SmsCodeSender，调用其发送短信验证码功能

noc-signIn.html 添加短信登录页面
SmsCodeGenerator 内容复制ImageCodeGenerator，Bean不做成可配置了，因为不像图形验证码有不同的图片，随机生成纯数字随机字符串
短信验证码长度和过期时间可配置，SmsCodeProperties复制ImageCodeProperties，ImageCodeProperties继承SmsCodeProperties，去掉重复代码

ValidateCodeController中createCode方法和createSmsCode方法很类似，可以采用模板方法的方式重构，重构前代码见下图
修改BrowserSecurityConfig 配置/code/* 接口不需要授权

这一节代码逻辑和图片验证码那块代码逻辑几乎一致，
spring常见开发技巧：依赖搜索

分级概念，分层封装：
ValidateCodeProcessor 处理整个验证码的流程，包括生成存储发送，如果整个流程改变，可以重新实现ValidateCodeProcessor接口，重写整个流程
ValidateCodeGenerator 处理具体的生成逻辑，如果仅仅生成验证码逻辑改变，可以重写ValidateCodeGenerator接口
```
```
@RestController
public class ValidateCodeController {

    /**
     * 生成图形验证码
     */
    @GetMapping("/code/image")
    public void createCode(HttpServletRequest request, HttpServletResponse response) throws IOException {
        // 根据随机数生成图片
        ImageCode imageCode = (ImageCode) imageCodeGenerator.generate(new ServletWebRequest(request));
        // 将随机数存到Session中
        sessionStrategy.setAttribute(new ServletWebRequest(request), SESSION_KEY, imageCode);
        // 再将生成的图片写到接口的响应中
        ImageIO.write(imageCode.getImage(), "JPEG", response.getOutputStream());
    }

    /**
     * 生成短信验证码
     */
    @GetMapping("/code/sms")
    public void createSmsCode(HttpServletRequest request, HttpServletResponse response) throws ServletRequestBindingException {
        // 生成短信验证码
        ValidateCode smsCode = imageCodeGenerator.generate(new ServletWebRequest(request));
        // 将短信验证码存到Session中
        sessionStrategy.setAttribute(new ServletWebRequest(request), SESSION_KEY, smsCode);
        // 通过短信服务商发送短信验证码，需要接口封装，不同的应用使用不同的短信供应商
        String mobile = ServletRequestUtils.getRequiredStringParameter(request, "mobile");
        smsCodeSender.send(mobile, smsCode.getCode());
    }

}
```
- 重构后的代码结构
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-c9ae6707fb7efbdd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4-11 短信登录开发
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-e060e2e3f5251d0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 校验短信验证码并登录
```
仿照SpringSecurity密码登录请求流程，写自己的短信登录请求流程
根据上图，需要自己实现如下四个组件
SmsCodeAuthenticationToken 封装登录信息，仿照 UsernamePasswordAuthenticationToken
SmsCodeAuthenticationFilter 拦截登录请求，仿照 UsernamePasswordAuthenticationFilter
SmsCodeAuthenticationProvider 身份验证逻辑，实现 AuthenticationProvider，仿照 DaoAuthenticationProvider
SmsCodeFilter 仿照 ValidateCodeFilter，短信验证码验证逻辑和图形验证码验证逻辑类似

然后需要一段配置，配置上面四个组件，把他们加入spring security中，让他们执行短信登录完整处理
```

#### 4-12 短信登录配置及重构
- 校验短信验证码并登录
```
在SmsCodeAuthenticationSecurityConfig#configure 方法中配置SmsCodeAuthenticationFilter和SmsCodeAuthenticationProvider，
然后仿照图形验证码配置，在BrowserSecurityConfig#configure 方法中配置短信过滤器SmsCodeFilter，
还需要将SmsCodeAuthenticationSecurityConfig 导入 BrowserSecurityConfig，配置
启动测试
```
- 重构代码
```
一、短信验证码和图形验证码，重复代码难以维护，重构代码消除重复
合并SmsCodeFilter、ValidateCodeFilter成为ValidateCodeFilter
注意：重复字符串变成常量

二、配置重构，进一步抽象，如下图
这里是个重点难点额，加油
```
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-fdc5f9076f6fb960.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4-13 小结
- Spring Security基本原理
**Spring Security 过滤器链**
- Spring Security开发基于表单的认证
用户名密码登录请求
短信登录请求

## 第5章 使用Spring Social开发第三方登录
#### 5-1 OAuth协议简介
**OAuth协议简介**
- OAuth协议要解决的问题
应用可以访问用户在微信上的所有数据
用户只有修改密码，才能收回授权
密码泄露的可能性大大提高
- OAuth协议中的各种角色
令牌Token：令牌只有部分权限,有有效期
服务提供商Provider：提供令牌
认证服务器Authorization Server：认证用户身份，提供令牌
资源服务器Resource Server：保存用户资源，验证令牌
资源所有者Resource Owner：用户
第三方应用Client
- OAuth协议运行流程
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-b8832569e54ff2ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- OAuth协议中的授权模式（上图第二步有四种实现）
**授权码模式 authorization code** （功能最完整）
**密码模式 resource owner password credentials**
客户端模式 client credentials
简化模式 implicit
- OAuth 协议授权码模式运行流程
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-b554d596823d46a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 授权码模式特点：
1. **用户同意授权是在认证服务器上完成，不会被伪造**
2. 返回的是授权码，不是令牌，第三方服务器还得申请令牌，安全性高
授权码模式功能最完整，流程最严密

#### 5-2 SpringSocial简介
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-c4e8552cb09329bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-9a48926e4a5ed7fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**SpringSocial 把这个流程封装并实现到SocialAuthenticationFilter中，并加入到过滤器链中，实现了第三方登录**

![图片.png](https://upload-images.jianshu.io/upload_images/1956963-34a5366c158c4d4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**Service Provider接口**是服务提供商的抽象，抽象类AbstractOAuth2ServiceProvider，需要自己实现QQ或微信的实现类
**OAuth2Operations** 封装了第一步到第五步标准的OAuth2协议流程，默认实现类OAuth2Template
**Api** 需要自己封装，因为不同第三方不一样，抽象类AbstractOAuth2ApiBinding

**Connection 接口**，封装前六步获取到的信息，由ConnectionFactory工厂创建出来
**ApiAdapter** 帮助适配不同服务提供商给的不同信息到一个标准数据中
数据库中有**UserConnection表**存放不同用户和第三方数据的对应关系，
由UserConnectionRepository 操作数据库UserConnection表

**Spring Social Core 是整个框架的核心**

#### 5-3 开发QQ登录（上）
#### 5-4 开发QQ登录（中）
#### 5-5 开发QQ登录（下）
#### 5-6 处理注册逻辑
#### 5-7 开发微信登录
#### 5-8 绑定和解绑处理
#### 5-9 单机Session管理
#### 5-10 集群Session管理
#### 5-11 退出登录

## 第6章 Spring Security OAuth开发APP认证框架
#### 6-1 SpringSecurityOAuth简介
- 为什么要开发APP认证框架
之前都是通过Cookie和Session来完成认证，对于应用来说开发繁琐，安全性和客户体验差，而且有些前端技术不支持Cookie，如小程序
**Spring Security OAuth** 是提供给应用认证，通过提供令牌Token来完成认证，相当于服务提供商
**Spring Security OAuth** 快速搭建服务提供商角色功能
- **Spring Security OAuth** 简介
认证服务器存储Token
OAuth2Authentication ProcessingFilter 验证Token
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-eea7282403145acb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 本章内容简介
实现一个标准的OAuth2协议中Provider角色的主要功能
重构之前的三种认证方式的代码，使其支持Token
高级特性

#### 6-2 实现标准的OAuth服务提供商
- 实现认证服务器
```
修改demo项目不依赖browser项目，而是依赖app项目
app项目需要自己的登录成功和登录失败处理器
测试保证启动app项目不报错

NocAuthorizationServerConfig
@EnableAuthorizationServer 注解在类上，就完成了认证服务器

-------------------------------------------------------------
[官方文档 授权码模式](https://tools.ietf.org/html/rfc6749#section-4.1.1)
官方文档查看授权码模式访问参数，启动服务器访问如下地址：
http://localhost:8080/oauth/authorize?response_type=code&client_id=5e0151e3-bd47-48e3-a2ee-7fd07bfdf59a&redirect_uri=http://example.com&scope=all
client_id 在服务器启动日志上有，代表哪个应用在请求授权
redirect_uri 可以拿到授权码
scope 请求什么样的权限
访问网址，会弹出Basic 身份验证，用户名密码在DemoUserDetailsService验证，会得到无权限的错误，需要在DemoUserDetailsService中添加ROLE_USER角色权限

client_id 可以在application.properties中配置，不用自带的自动生成的（会变）：
security.oauth2.client.clientId = noc
security.oauth2.client.clientSecret = noc-secret
获取授权码：访问/oauth/authorize，授权页面，同意会带着授权码跳到redirect_uri上

POST请求/oauth/token，拿着认证信息Authorization(clientId和clientSecret) Basic Auth和授权码等参数发送请求，会得到access_token
参数：grant_type=authorization_code&client_id=noc&code=填授权码&redirect_uri=http://example.com&scope=all

---------------------------------------------------------------
密码模式授权：
POST请求/oauth/token，拿着认证信息Authorization(clientId和clientSecret) Basic Auth和参数发送请求，会得到access_token
参数：grant_type=password、username=、password=、scope=all

如果是自己公司APP或前端，拿着账号密码换token可行，如果不是自家APP，不可行
```
- 实现资源服务器
```
NocResourceServerConfig
@EnableResourceServer 注解在类上，就完成了资源服务器

重启服务，访问demo服务中的/user/me接口，会出异常unauthorized，没有身份认证
请求头上带着Authorization：bearer 5d19a0fa-eda5-4ab0-957b-baf6bfa6aca7（token_type + access_token），发送请求，请求成功
```

#### 6-3 SpringSecurityOAuth核心源码解析
- Spring Security OAuth 核心源码
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-3516b999d7258c48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
TokenEndpoint：负责处理获取令牌的请求/oauth/token
ClientDetailService：根据clientId读取第三方应用Client信息ClientDetails，类似UserDetailService
TokenRequest：由TokenEndpoint创建，封装请求信息，ClientDetails也在里面
TokenGranter：挑选令牌生成逻辑，生成OAuth2Authentication对象
OAuth2Request：ClientDetails和TokenRequest的整合
OAuth2Authentication对象包含请求信息OAuth2Request和授权用户信息Authentication
AuthenticationServerTokenServices 最终生成令牌
DefaultTokenServices#createAccessToken 生成方法
TokenStore：负责令牌存取
TokenEnhancer：令牌增强器
```

#### 6-4 重构用户名密码登录
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-21ffb3deeaae88a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
逻辑：在AuthenticationSuccessHandler中通过AuthorizationServerTokenServices生成OAuth2AccessToken！
在之前登录过程中已经创建好Authentication，主要是需要构建OAuth2Request，思路可以见上图

需要从请求头Authrization中解析clientId
BasicAuthenticationFilter#doFilterInternal 可以解析请求头中的Basic信息，把这段逻辑放到我们的onAuthenticationSuccess方法中
根据clientId，通过ClientDetailsService获取ClientDetails，构建TokenRequest，OAuth2Request，OAuth2Authentication，然后通过AuthorizationServerTokenService获取OAuth2AccessToken，流程很清晰

需要设置安全配置：
NocResourceServerConfig extends ResourceServerConfigurerAdapter
NocResourceServerConfig#configure 类似BrowserSecurityConfig

POST请求 /authentication/form 请求头上带着Authorization，参数username和password，成功获取access_token
访问demo服务中的/user/me接口，请求头上带着Authorization：bearer + access_token，发送请求，请求成功
```

#### 6-5 重构短信登录
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-dc33d851374519cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
需要把Session存储验证码，改成根据devideId从redis或者数据库存取

ValidateCodeRepository接口 负责保存、获取、移除验证码
app和browser分别有一个实现类：RedisValidateCodeRepository、SessionValidateCodeRepository
发送短信验证码，请求头带deviceId，查看redis，短信验证码登录，请求头带deviceId，请求成功

发送短信验证码：GET请求http://localhost:8080/code/sms?mobile=15300000000
短信验证码登录：POST请求http://localhost:8080/authentication/mobile，请求请求头上带着Authorization，参数mobile和smsCode，
```

#### 6-6 重构社交登录
#### 6-7 重构注册逻辑

#### 6-8 令牌配置
- Token 处理
基本的Token参数配置
使用JWT替换默认的Token
扩展和解析JWT的信息
```
认证服务器关于Token配置：
ImoocAuthorizationServerConfig extends AuthorizationServerConfigurerAdapter
当你继承AuthorizationServerConfigurerAdapter这个类，UserDetailsService和AuthenticationManager需要自己配

配置给哪些第三方发令牌：
重写configure(ClientDetailsServiceConfigurer clients)方法，application.yml中的clientId和clientSecret失效，
可以在此configure方法中配置clientId，clientSecret、令牌有效时间、支持的授权模式、权限等

然后修改成可配置：OAuth2ClientProperties、OAuth2Properties、SecurityProperties
在demo项目中配置client，使用密码模式测试
```
- 控制令牌存储
```
使用Redis存储令牌
TokenStoreConfig 负责令牌存取
NocAuthorizationServerConfig#configure 方法中配置 endpoints.tokenStore(tokenStore)
测试，重启服务器，还可以用重启前的令牌访问/user/me 接口
访问demo服务中的/user/me接口，请求头上带着Authorization：bearer + access_token，发送请求，请求成功
查看redis，保存了很多属性
```

#### 6-9 使用JWT替换默认令牌

- JWT（Json Web Token）是一个令牌标准
特点：自包含（本身有信息）、密签、可扩展
- 使用JWT替换默认的Token
```
TokenStoreConfig.JwtTokenConfig 配置JWT 
OAuth2Properties 配置秘钥
NocAuthorizationServerConfig#config 配置 JwtAccessTokenConverter
密码模式获取令牌得到JWT token

https://www.jsonwebtoken.io/ 网站有工具可以解析jwt令牌

扩展jwt令牌内的信息：
TokenStoreConfig.TokenEnhancer
NocJwtTokenEnhancer#enhance
NocAuthorizationServerConfig#config 配置 TokenEnhancer

业务方法获取jwt令牌的额外信息：
demo 项目需要添加依赖 
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.7.0</version>
</dependency>
UserController#getCurrentUserJwt 方法：
获取Header中的token，然后通过
Jwts.parser().setSigningKey("XXX".getBytes("UTF-8")).parseClaimsJws(token).getBody(); 
获取JWT信息解析，访问 /user/me/jwt 接口测试

令牌的刷新：
在用户无感知情况下，可以使用POST请求/oauth/token接口，请求头需要Authorization信息，
参数grant_type=refresh_token refresh_token scope=all，来获取新token
NocAuthorizationServerConfig#configure 方法可以设置 refresh_token 的有效期，一个月， .refreshTokenValiditySeconds
```

#### 6-10 基于JWT实现SSO单点登录1
[**Spring Security OAuth2 开发指南**](https://www.cnblogs.com/xingxueliao/p/5911292.html)
- 基于JWT（Json Web Token）实现SSO（Single Sign On）
淘宝和天猫在不同的服务器上，当淘宝登录成功，刷新天猫，天猫也登录了
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-6bae350d121ae720.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
sso-demo pom项目、sso-server 认证服务器项目、sso-client1 应用A项目、sso-client2 应用B项目

sso-server 认证服务器项目
第一步：
SsoServerApplication

第二步：
// @EnableAuthorizationServer使一个授权服务器在当前的应用程序上下文。
@EnableAuthorizationServer
// AuthorizationServerConfigurerAdapter 类实现 AuthorizationServerConfigurer 它提供了所有必要的方法来配置一个授权服务器。
public class SsoAuthorizationServerConfig extends AuthorizationServerConfigurerAdapter

第三步：配置认证服务器
SsoAuthorizationServerConfig#configure(ClientDetailsServiceConfigurer clients) 配置两个应用
SsoAuthorizationServerConfig#configure(AuthorizationServerEndpointsConfigurer endpoints) 配置使用Jwt令牌
SsoAuthorizationServerConfig#configure(AuthorizationServerSecurityConfigurer security) 认证服务器安全配置

第四步：
application.yml 配置文件
```

#### 6-11 基于JWT实现SSO单点登录2
```
sso-client1 应用A项目
SsoClient1Application
提供 /user 接口查看用户信息
@EnableOAuth2Sso 会帮我们完成跳转到授权服务器，需要配置 application.properties
static/index.html 访问client2页面
sso-client2 应用B项目，代码和配置同上，测试

----细化场景----
一、优化登录体验
sso-server 修改basic登录为表单登录
public class SsoSecurityConfig extends WebSecurityConfigurerAdapter
SsoSecurityConfig#configure(HttpSecurity http) 方法配置表单登录

二、用户名密码从数据库获取
public class SsoUserDetailsService implements UserDetailsService
SsoSecurityConfig 配置加密器PasswordEncoder
SsoSecurityConfig#configure方法使用自己的userDetailsService和加密器来进行身份验证
SsoUserDetailsService#loadUserByUsername 方法 配置加密器和密码

三、授权页面隐藏，自动提交确认授权
Spring 源码 WhitelabelApprovalEndpoint 类实现授权页面，处理授权请求
我们自己实现 @RestController 覆盖掉 @FrameworkEndpoint 的处理
SsoApprovalEndpoint 代码照搬 WhitelabelApprovalEndpoint，添加 @RestController 
修改访问授权页面 SsoApprovalEndpoint.TEMPLATE ，隐藏页面内容，并修改为直接提交confirmationForm表单
简单粗暴 哈哈哈哈

这个基于JWT实现SSO单点登录逻辑很清晰
```

## 第7章 使用Spring Security控制授权
#### 7-1 SpringSecurity授权简介
- Spring Security 对授权的定义
控制Url能不能访问，是一个安全问题，由Spring Security管理
控制权限能不能看见，是一个用户体验问题，自己开发权限模块
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-d55cea46eadbf32b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**简单场景**
- 只区分是否登录
```
BrowserSecurityConfig#config方法中 
.authorizeRequests().antMatchers(XXXXX).permitAll() // 控制允许所有人访问
.anyRequest().authenticated() // 控制需要登录
```
- 只区分简单角色
```
.antMatchers(HttpMethod.GET,"/user/*").hasRole("ADMIN")，// 可配置请求需要权限
UserDetailsService 实现类中new User时给了权限ROLE_ADMIN，才可以访问

ExpressionUrlAuthorizationConfigurer.hasRole(role)添加了前缀ROLE_
```

#### 7-2 SpringSecurity源码解析
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-cbc6345155695b04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
绿色的过滤器是与身份认证相关的
AnonymousAuthenticationFilter 源码分析
AnonymousAuthenticationFilter#doFilter 判断前面过滤器是否成功身份认证
如果没有认证成功，createAuthentication -> AnonymousAuthenticationToken 
AnonymousAuthenticationToken 传到FilterSecurityInterceptor，进行授权
```
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-db70edd6aa4afa38.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
三个比较关键的类
FilterSecurityInterceptor 授权判断的主入口
AccessDecisionManager 接口，有一个抽象实现和三个具体实现
AccessDecisionVoter 投票者，处理判断逻辑，请求投过还是不过

AbstractAccessDecisionManager 抽象实现管理一组 AccessDecisionVoter，判断最终过不过
AffirmativeBased 一个通过，请求通过，Spring Security默认使用这一种
ConsensusBased 比较通过和不通过个数，看哪种多
UnanimousBased 一个不通过，请求不通过

SecurityConfig 配置信息，封装成一组ConfigAttribute对象，对应url需要的权限
Authentication 封装用户拥有哪些权限
FilterSecurityInterceptor 中封装的请求信息
FilterSecurityInterceptor 将这三份信息传给 AccessDecisionManager，交给投票者投票汇总投票结果

Spring Security 3.0 新增 WebExpressionVoter 投票器来投票
```

#### 7-3 权限表达式
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-576824214577fbf6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
hasRole方法内会在权限前添加ROLE_
hasAuthority完全匹配字符串

表达式联合起来使用：
.access("hasRole('ADMIN') and hasIpAddress('xxxx')")
```
- 安全模块封装
权限模块提供AuthorizeConfigProvider并且提供默认实现
别的模块（用户模块）可以有自己的实现
应用A也可以有自己的实现
然后由权限模块AuthorizeConfigManager收集这些实现配置
```
AuthorizeConfigProvider#config 方法封装授权配置
权限模块的实现：NocAuthorizeConfigProvider
AuthorizeConfigManager#config 方法合成授权配置
ImoocAuthorizeConfigManager 收集Provider

BrowserSecurityConfig 使用AuthorizeConfigManager
NocResourceServerConfig 使用AuthorizeConfigManager
demo项目 DemoAuthorizeConfigProvider 配置自己的权限
控制页面权限同理：.antMatchers("/demo.html")
```
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-84dc057e8db99b78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 7-4 基于数据库Rbac数据模型控制权限
角色众多，权限复杂的场景
权限规则随着公司和业务的发展不断变化
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-4bd3550f353f5b5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
noc-security-authorize 项目实现基于RBAC的标准实现
RbacService#hasPermission 判断是否能访问
RbacServiceImpl#hasPermission 根据用户名，读取用户所拥有权限的所有URL
AntPathMatcher 来判断url是否匹配

Spring Security 需要写权限表达式，调用RbacService#hasPermission 方法，判断是否通过
DemoAuthorizeConfigProvider#config
config.anyRequest().access("@rbacService.hasPermission(request,authentication)")
这个配置需要最后才调用 @Order(Integer.MAX_VALUE)
```

## 第8章 课程总结
#### 8-1 课程总结
```
imooc-security-authorize 项目代码实现
doc 中有项目总结
application-example.properties 配置项说明
db.sql 数据库文件
extends.txt 扩展点总结说明
readme.txt 怎么使用安全模块

根据readme.txt，演示test项目如何使用我们的安全模块
```
- 如何控制别人写代码
他想干一件什么事，需要实现哪个接口
他想调整一种效果，需要配置什么参数

## 附录
-  [源码](https://github.com/liuhuiAndroid/noc-security)
- IDEA 快捷键
```
F7            Step Into 相当于eclipse的f5就是  进入到代码
F8            Step Over 相当于eclipse的f6      跳到下一步
F9            resume programe 恢复程序
Alt+F10       show execution point 显示执行断点
Alt+shift+F7  Force Step Into 这个是强制进入代码
Shift+F8      Step Out  相当于eclipse的f8跳到下一个断点，也相当于eclipse的f7跳出函数
Atl+F9        Run To Cursor 运行到光标处
ctrl+shift+F9   debug运行java类
ctrl+shift+F10  正常运行java类
alt+F8          debug时选中查看值
```
