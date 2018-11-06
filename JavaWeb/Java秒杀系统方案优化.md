###一、项目概述
业务场景：秒杀
核心问题：秒杀瞬时并发非常大，互联网高并发的场景如何应对？
核心技术：我们需要通过缓存、异步化提高系统的吞吐量 
######项目框架
前端：Thymeleaf、Bootstrap、JQuery
后端：SpringBoot、JSR303、Mybatis
中间件：RabbitMQ、Redis、Druid
######实现步骤
业务逻辑：分布式会话、商品列表页、商品详情页、订单详情页
优化：系统压测、缓存优化、消息队列、接口安全

###二、项目环境搭建
1. idea配置
Maven选择Create from archetype选择创建项目的模板，我们选择org.apache.maven.archetypes:maven-archetype-quickstart。
蓝色是源码目录，灰色是普通文件夹

2. Spring Boot 环境搭建
Spring Boot快速入门和详细文档
[Spring Boot Quick Start](http://projects.spring.io/spring-boot/#quick-start)
[Spring Boot Reference Guide](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#getting-started)

2. 集成Thymeleaf,Result结果封装
- Result结果封装
对返回结果Result封装，避免硬编码。
测试Result
- 集成Thymeleaf
[Thymeleaf 官网](https://www.thymeleaf.org)
[Thymeleaf Github](https://github.com/thymeleaf/thymeleaf)
Spring Boot 集成 Thymeleaf需要先引入依赖，然后加载默认配置文件application.properties
  ```
    spring.thymeleaf.prefix=classpath:/templates/
    spring.thymeleaf.suffix=.html
  ```
测试Thymeleaf

4. 集成Mybatis+druid
- 添加mybatis依赖
[mybatis-spring-boot-autoconfigure](http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/index.html)
添加配置
```
mybatis.type-aliases-package=com.lh.seckill.domain
mybatis.configuration.map-underscore-to-camel-case=true
mybatis.configuration.default-fetch-size=100
mybatis.configuration.default-statement-timeout=3000
mybatis.mapperLocations = classpath:com/lh/seckill/dao/*.xml
```
- 添加druid依赖
[druid github](https://github.com/alibaba/druid)
[druid 中文文档](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter)
添加jdbc和druid依赖
添加druid配置
```
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
    </dependency>
```
```
spring.datasource.url=jdbc:mysql://localhost:3306/seckill?useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true&useSSL=false
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.filters=stat
spring.datasource.maxActive=2
spring.datasource.initialSize=1
spring.datasource.maxWait=60000
spring.datasource.minIdle=1
spring.datasource.timeBetweenEvictionRunsMillis=60000
spring.datasource.minEvictableIdleTimeMillis=300000
spring.datasource.validationQuery=select 'x'
spring.datasource.testWhileIdle=true
spring.datasource.testOnBorrow=false
spring.datasource.testOnReturn=false
spring.datasource.poolPreparedStatements=true
spring.datasource.maxOpenPreparedStatements=20
```
- 访问数据库测试
[mybatis-3 github](https://github.com/mybatis/mybatis-3)
[mybatis-3 中文文档](http://www.mybatis.org/mybatis-3/zh/index.html)
参考mybatis文档，使用不需要配置文件的方法
- 测试事务
@Transactional

5. 集成Jedis+Redis安装+通用缓存Key封装
- 安装Redis
参考Redis初识
- 集成Redis
  1. 添加Jedis依赖
  [jedis](https://github.com/xetorthio/jedis)
  2. 添加Fastjson依赖
[fastjson](https://github.com/alibaba/fastjson)
  3. 添加redis配置
```
#redis
redis.host=192.168.10.179
redis.port=6379
redis.timeout=3
redis.password=123456
redis.poolMaxTotal=10
redis.poolMaxIdle=10
redis.poolMaxWait=3
```
  4. RedisConfig自动读取redis配置
```
@Component
@ConfigurationProperties(prefix="redis")
public class RedisConfig {
    private String host;
    private int port;
    private int timeout;//秒
    private String password;
    private int poolMaxTotal;
    private int poolMaxIdle;
    private int poolMaxWait;//秒
}
```
  Spring Boot 如何注入Bean
```
@Service
public class RedisPoolFactory {

    @Autowired
    RedisConfig redisConfig;

    @Bean
    public JedisPool JedisPoolFactory() {
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        poolConfig.setMaxIdle(redisConfig.getPoolMaxIdle());
        poolConfig.setMaxTotal(redisConfig.getPoolMaxTotal());
        poolConfig.setMaxWaitMillis(redisConfig.getPoolMaxWait() * 1000);
        JedisPool jp = new JedisPool(poolConfig, redisConfig.getHost(), redisConfig.getPort(),
                redisConfig.getTimeout()*1000, redisConfig.getPassword(), 0);
        return jp;
    }

}
```
  5. SampleController中测试redis

- 通用缓存Key封装
模板模式：实现类 -> 抽象类 -> 接口
在redis中的key增加prefix

###四、实现登录功能
1. 数据库设计
2. 明文密码两次MD5处理
  1.用户端：PASS=MD5(明文+固定Salt)
  为了防止明文密码在网络传输
  2.服务端：PASS=MD5(用户输入+随机Salt)
  防止数据库泄露，通过反查就得到密码
引入MD5工具类依赖，编写MD5Util
```
	<dependency>
	    <groupId>commons-codec</groupId>
	    <artifactId>commons-codec</artifactId>
	</dependency>
	  
	<dependency>
	    <groupId>org.apache.commons</groupId>
	    <artifactId>commons-lang3</artifactId>
	    <version>3.6</version>
	</dependency>
```
3. 登录功能
前端用到的技术：bootstrap、jquery-validation、layer
先验证前端参数有没有正确的传递到后端

4. JSR303参数检验+全局异常处理器
- JSR303参数检验
引入依赖，实现自定义参数校验
- 全局异常处理器
@ControllerAdvice + @ExceptionHandler

5. 分布式Session  Session同步？单点登录？
token没有存到数据库中，而是存到redis中，然后放入cookie中
token有效期是最后一次加上你的有效时间
自定义ArgumentResolver给SpringMVC的参数中赋值减少校验登录操作

###五、实现秒杀功能
1. 数据库设计
商品表、订单表、秒杀商品表、秒杀订单表
2. 商品列表页 
3. 商品详情页
商品ID用snowflake算法来自增
添加秒杀状态信息，
4. 秒杀倒计时实现
5. 秒杀功能实现
判断是否登录和是否秒杀到
开启事务：减库存 下订单 写入秒杀订单
减库存：直接更新数据库表库存字段
跳转订单详情页
6. 订单详情页

###六、JMeter压测
1. JMeter入门
[jmeter 官方网站](http://jmeter.apache.org/)
[jmeter github](https://github.com/apache/jmeter)
访问官方网站下载apache-jmeter-4.0.tgz
bin/jmeter.bat 启动jmeter
测试：并发在多少的时候我们的qps/tps是多少
测试步骤
```
1.测试计划、添加、Threads、线程组，选择线程数1000个，Ramp-Up Reriod=0。
2.线程组、配置元件config element、HTTP请求默认值http request defaults
协议：http，服务器IP：localhost，端口号8080
3.线程组、Sampler、HTTP请求
名称：商品列表，HTTP请求方法、路径
/to_list
4.查看运行结果：线程组、监听器、聚合报告、图形结果、表格等
主要观察聚合报告aggregate report
5.点击启动
查看Throughput吞吐量，简单理解成qps
一秒钟执行多少个请求
6.清空表格数据然后重新测试
```
瓶颈在数据库上
cat /etc/motd，修改此文件可以改变linux初始图片
top命令监控服务器cpu和内存等，看load average系统负载

2. 自定义变量模拟多用户
  -1.测试计划 - > 添加配置元件 - > CSV Data Set Config
测试UserController
HTTP请求上修改url并添加参数
application.properties中redis.poolMaxTotal和redis.poolMaxIdle、redis.poolMaxWait改成1000和500、500，redis。timeout=10，否则有时候会报错无法获取jedis，druid中参数也改大一点
  -2.引用变量 {}
添加、配置元件、CSV Data Set Config
配置配置文件，引入。
Variable Names：userId，userToken等
然后在HTTP请求中使用${userToken}引用

3. JMeter命令行使用
  - 1.在windows上录好jmx
引入打jar包依赖，命令行执行mvn clean package打jar包
```
<build>
<finalName>${project.artifactId}</finalName>
<plugins>
<plugin>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
</plugins>
</build>
```
linux下利用nohup后台运行jar文件包程序：
在linux上执行：nohup java -jar seckill.jar &
tail -f nohup.out 查看jar
  - 2.命令行：jmeter.sh -n -t XXX.jmx -l result.jtl
  - 3.把result.jtl导入到jmeter
sz下载
在UserUtil中写一个生成token的方法
vi .jms 修改配置文件路径

4. Redis压测工具redis-benchmark
 - 1. redis-benchmark -h 127.0.0.1 -p 6379 -c 100 -n 100000
  100个并发链接，100000个请求
  测试结果：一秒钟能完成10W个请求
 - 2. redis-benchmark -h 127.0.0.1 -p 6379 -q -d 100
存取大小为100字节的数据包
测试结果：一秒钟能完成7W个请求
 - 3.redis-benchmark -t set,lpush -q -n 100000 
只测试某些操作的性能
 - 4.redis-benchmark -n 100000 -q script load "redis.call('set','foo','bar')"
只测试某些数值存取的性能

5. Spring Boot打war包
 - 1.添加spring-boot-starter-tomcat的provided依赖
```
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <scope>provided</scope>
    </dependency>
```
 - 2.添加maven-war-plugin插件
```
<build>
<finalName>${project.artifactId}</finalName>
<plugins>
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-war-plugin</artifactId>
<configuration>
<failOnMissingWebXml>false</failOnMissingWebXml>
</configuration>
</plugin>
</plugins>
</build>
```
 - 3.packaging改成war
 - 4.Apllication类需要继承SpringBootServletInitializer，重写configer方法
```
@SpringBootApplication
public class MainApplication extends SpringBootServletInitializer {

    public static void main(String[] args) throws Exception {
        SpringApplication.run(MainApplication.class, args);
    }

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(MainApplication.class);
    }
    
}
```
- 5.cmd切换到目录下执行：mvn clean package
war包会生成在target目录下

###七、页面优化技术
1. 页面缓存+URL缓存+对象缓存
- 以商品列表为例做页面缓存，压测
1. 取缓存
2. 手动渲染模板
3. 结果输出
4. 检查redis
```
    @RequestMapping("/to_list")
    @ResponseBody
    public String list(HttpServletRequest request, HttpServletResponse response, Model model, SeckillUser user) {
        //取缓存
        String html = redisService.get(GoodsKey.getGoodsList, "", String.class);
        if(!StringUtils.isEmpty(html)) {
            return html;
        }
        model.addAttribute("user", user);
        //查询商品列表
        List<GoodsVo> goodsList = goodsService.listGoodsVo();
        model.addAttribute("goodsList", goodsList);
        // return "goods_list";
        SpringWebContext ctx = new SpringWebContext(request,response,
                request.getServletContext(),request.getLocale(), model.asMap(), applicationContext);
        //手动渲染
        html = thymeleafViewResolver.getTemplateEngine().process("goods_list", ctx);
        if(!StringUtils.isEmpty(html)) {
            redisService.set(GoodsKey.getGoodsList, "", html);
        }
        return html;
    }
```
- 以商品详情为例URL缓存
其实和页面缓存一样，只是把url参数作为key
- 对象缓存
像SeckillUserService中getByToken方法通过token获取到User对象就是对象缓存。
对SeckillUserService中getById方法也使用对象缓存，在数据库数据变动时，更新或删除缓存。

2. 页面静态化，前后端分离
- 页面静态化：把页面缓存到用户的浏览器上
1. 常用技术AngularJS、Vue.js
2. 优点：利用浏览器的缓存
- 前后端分离
我们用jQuery模拟页面静态化，对商品详情进行修改
改造goods_detail.htm
改造秒杀，修改SeckillController中seckill方法和goods_detail.htm,order_detail.htm
开发者工具，接口304，说明使用缓存了，可以对静态文件配置spring resource handing，火狐查看直接输出已缓存，根本不需要和服务端交互
```
#static
spring.resources.add-mappings=true
spring.resources.cache-period=3600
spring.resources.chain.cache=true 
spring.resources.chain.enabled=true
spring.resources.chain.gzipped=true
spring.resources.chain.html-application-cache=true
spring.resources.static-locations=classpath:/static/
```
改造订单详情
增加拦截器@NeedLogin
解决减库存出现负数的问题：修改sql，reduceStock。数据库本身会加锁，不会出现两个线程更新同一个数据的情况
```
    @Update("update seckill_goods set stock_count = stock_count - 1 where goods_id = #{goodsId} and stock_count > 0")
    int reduceStock(SeckillGoods g);
```
- 超卖问题
1. 数据库加唯一索引：防止用户重复购买
利用数据库的唯一索引，在seckill_order表中加u_uid_gid user_id,goods_id Unique BTREE 索引
判断是否秒杀到修改成也去查缓存，修改OrderService中的getSeckillOrderByUserIdGoodsId方法
2. SQL加库存数量判断：防止库存变成负数

压测检验是否卖超
保证数据一致还是要依靠我们的数据库

3. 静态资源优化
- 1.JS/CSS压缩，减少流量
- 2.多个JS/CSS组合，减少连接数
例如淘宝的[tengine](http://tengine.taobao.org/)
组合多个CSS、JavaScript文件的访问请求变成一个请求
自动去除空白字符和注释从而减小页面的体积
webpack也是可以
- 3.CDN优化
内容分发网络，其实也是一种缓存
例如阿里云CDN

###八、接口优化
[erlang 官网下载](http://www.erlang.org/downloads)   OTP 20.3 Source File
[rabbitmq 官网下载](http://www.rabbitmq.com/download.html)   Generic UNIX binary

```
RabbitMQ安装
1.先安装erlang
yum install ncurses-devel -y
tar -xvf otp_src_20.3.tar.gz -C ./ # 源码在官网下载
cd otp_src_20.3
./configure --prefix=/usr/local/erlang20 --without-javac
make -j 4 && make install
cd /usr/local/erlang20/bin
./erl 验证是否安装成功
2.安装RabbitMQ
安装python：yum install python -y
安装simplejson：yum install xmlto -y
yum install python-simplejson -y
---------------------------
wget https://pypi.python.org/packages/source/s/simplejson/simplejson-3.5.2.tar.gz#md5=10ff73aa857b01472a51acb4848fcf8b --no-check-certificate
tar vxzf simplejson-3.5.2.tar.gz
cd simplejson-3.5.2
python setup.py install
---------------------------
解压
xz -d rabbitmq-server-generic-unix-3.7.4.tar.xz 
tar xf rabbitmq-server-generic-unix-3.7.4.tar 
mv rabbitmq_server-3.7.4/ /usr/local/rabbitmq
cd /usr/local/rabbitmq/sbin
3.启动RabbitMQ
1../rabbitmq-server 启动rabbitMQ server
tail -f rabbit@localhost.log
2.netstat -nap | grep 5672 
3. ./rabbitmqctl stop 关闭rabbitmq
ps -ef | grep rabbit
4. 可以配置环境变量，这样在任何地方都可以使用rabbitmq-server 启动rabbitMQ server
export PATH=$PATH:/usr/local/ruby/bin:/usr/local/erlang20/bin:/usr/local/rabbitmq/sbin
source /etc/profile
-----------------------------------
erl: command not found
因为环境变量不同，将erlang的erl软连接到/usr/bin目录下：
ln -s /usr/local/erlang/bin/erl /usr/bin/erl
```
```
docker 安装 rabbitmq
docker run -d --hostname my-rabbit --name some-rabbit -p 5672:5672 rabbitmq:3
```
Spring Boot集成RabbitMQ
1. 添加依赖spring-boot-starter-amqp，配置
```
	<dependency>  
		<groupId>org.springframework.boot</groupId>  
		<artifactId>spring-boot-starter-amqp</artifactId>  
	</dependency>

#rabbitmq
spring.rabbitmq.host=10.110.3.62
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
spring.rabbitmq.virtual-host=/
#\u6D88\u8D39\u8005\u6570\u91CF
spring.rabbitmq.listener.simple.concurrency= 10
spring.rabbitmq.listener.simple.max-concurrency= 10
#\u6D88\u8D39\u8005\u6BCF\u6B21\u4ECE\u961F\u5217\u83B7\u53D6\u7684\u6D88\u606F\u6570\u91CF
spring.rabbitmq.listener.simple.prefetch= 1
#\u6D88\u8D39\u8005\u81EA\u52A8\u542F\u52A8
spring.rabbitmq.listener.simple.auto-startup=true
#\u6D88\u8D39\u5931\u8D25\uFF0C\u81EA\u52A8\u91CD\u65B0\u5165\u961F
spring.rabbitmq.listener.simple.default-requeue-rejected= true
#\u542F\u7528\u53D1\u9001\u91CD\u8BD5
spring.rabbitmq.template.retry.enabled=true 
spring.rabbitmq.template.retry.initial-interval=1000 
spring.rabbitmq.template.retry.max-attempts=3
spring.rabbitmq.template.retry.max-interval=10000
spring.rabbitmq.template.retry.multiplier=1.0
``` 
2.创建消息接收者
MQReceiver
3.创建消息发送者
MQSender
4.最简单的测试 Direct模式
www.rabbitmq.com/access-control.html，配置vi /usr/local/rabbitmq/etc/rabbitmq/rabbitmq.config，写入[{rabbit, [{loopback_users, []}]}].，否则guest用户无法远程访问。配置好重启。 
./rabbitmqctl stop
./rabbitmq-server
5.了解Topic模式
6.了解Fanout模式
7.了解Headers模式

- 秒杀接口优化
```
思路：减少数据库访问
1.系统初始化，把商品库存数量加载到redis
继承InitializingBean接口实现afterPropertiesSet方法
2.收到请求，Redis预减库存，库存不足，直接返回，否则进入3
3.请求入队。立刻返回排队中
4.请求出队，生成订单，减少库存
5.客户端轮询，是否秒杀成功
```
1. Redis预减库存减少数据库访问
2. 内存标记减少Redis访问
3. RabbitMQ队列缓冲，异步下单，增加用户体验
4. nginx横向扩展
5. 反向代理和负载均衡,nginx也可以配置缓存，开启压缩
LVS：linux内核
- 压测

###九、安全优化
1. 秒杀接口地址隐藏
```
思路：秒杀开始之前，先去请求接口获取秒杀地址
1. 接口改造，带上PathVariable参数
2. 添加生成地址的接口
3. 秒杀收到请求，先验证PathVariable
```
2. 数学公式验证码
```
思路：秒杀开始之前，先输入验证码，分散用户的请求
1. 添加生成验证码的接口
2. 在获取秒杀路径的时候，验证验证码
3. ScriptEngine使用
```
3. 接口限流防刷
```
思路：对接口做限流
1.可以用拦截器减少对业务侵入
```
