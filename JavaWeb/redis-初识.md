#### [Redis 官网](https://redis.io/)
### 一、安装Redis
```
如果没有gcc需要在线安装。yum install gcc-c++
安装步骤：
第一步：redis的源码包 redis-3.0.0.tar.gz上传到linux系统。
第二步：解压缩redis。
第三步：编译。进入redis源码目录。make 
第四步：安装。make install PREFIX=/usr/local/redis
可以使用命令：make install -j 4  使用4核cpu加快make
PREFIX参数指定redis的安装目录。一般软件安装到/usr目录下
注意：可能会报错：You need tcl 8.5 or newer in order to run the Redis test
需要安装tcl： yum -y install tcl
第五步：前端启动。在redis的安装目录下直接启动redis-server
第六步：后端启动。
cp redis.conf /usr/local/redis/bin/
修改配置文件redis.conf：daemonize yes
redis.conf备选参数
bind 0.0.0.0 // 所有都才能访问
daemonize yes 
requirepass 123456 // 设置密码
 ./redis-server redis.conf
查看redis进程：ps aux|grep redis
ps -ef | grep redis
第七步：
默认连接Redis：./redis-cli 
自定义连接Redis：./redis-cli -h 192.168.1.1 -p 6379
-h：连接的服务器的地址
-p：服务的端口号
关闭Redis：./redis-cli shutdown
如果设置了密码需要先使用:auth your_password
```
```
redis重启方法：进入redis-cli 执行shutdown save,然后重新启动
在redis的utils目录下有一个install_server.sh,执行可以生成系统服务
/usr/local/redis/redis.conf
/usr/local/redis/redis.log
/usr/local/redis/data
chkconfig --list | grep redis 查看有没有redis服务
systemctl status redis_6379 查看redis状态
systemctl stop redis_6379 关闭redis进程
systemctl start redis_6379 打开redis进程
```

### 二、Redis的数据类型
1. 存储String
set key value
get key
incr key
decr key
2. 存储Hash
相当于一个key对于一个map，map中还有key-value
hset key field value
hgetall key
hget key field
3. 存储List
4. 存储Set
5. 存储SortedSet
6. Key命令
expire key second：设置key的过期时间
ttl key：查看key的有效期
persist key：清除key的过期时间。Key持久化。

###三、Redis的持久化方案
Redis的所有数据都是保存到内存中的。
1. Rdb：快照形式，定期把内存中当前时刻的数据保存到磁盘。Redis默认支持的持久化方案。
```
在redis.conf配置文件中配置
################################ SNAPSHOTTING  ################################
#
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""

save 900 1
save 300 10
save 60 10000
```
2. Aof形式：append only file。把所有对redis数据库操作的命令，增删改操作的命令。保存到文件中。数据库恢复时把所有的命令执行一遍即可。
```
############################## APPEND ONLY MODE ###############################

# By default Redis asynchronously dumps the dataset on disk. This mode is
# good enough in many applications, but an issue with the Redis process or
# a power outage may result into a few minutes of writes lost (depending on
# the configured save points).
#
# The Append Only File is an alternative persistence mode that provides
# much better durability. For instance using the default data fsync policy
# (see later in the config file) Redis can lose just one second of writes in a
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
#
# Please check http://redis.io/topics/persistence for more information.

appendonly no

# The name of the append only file (default: "appendonly.aof")

appendfilename "appendonly.aof"
```

###四、Redis集群的搭建
Redis 集群中内置了 16384 个哈希槽，当需要在 Redis 集群中放置一个 key-value 时，redis 先对 key 使用 crc16 算法算出一个结果，然后把结果对 16384 求余数，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，redis 会根据节点数量大致均等的将哈希槽映射到不同的节点。
1. Redis集群搭建
Redis集群中至少应该有三个节点。要保证集群的高可用，需要每个节点有一个备份机。
Redis集群至少需要6台服务器。
搭建伪分布式。可以使用一台虚拟机运行6个redis实例。需要修改redis的端口号7001-7006
```
1. 使用ruby脚本搭建集群。需要ruby的运行环境。
安装ruby
yum install ruby
yum install rubygems
2. 安装ruby脚本运行使用的包。
gem install redis-3.0.0.gem
cd redis-3.0.0/src
ll *.rb
3. 创建6个redis实例，每个实例运行在不同的端口。需要修改redis.conf配置文件。配置文件中还需要把cluster-enabled yes前的注释去掉。
4. 启动每个redis实例
5. 使用ruby脚本搭建集群
./redis-trib.rb create --replicas 1 192.168.10.179:7001 192.168.10.179:7002 192.168.10.179:7003 192.168.10.179:7004 192.168.10.179:7005 192.168.10.179:7006
```
2. 集群的使用方法
./redis-cli -p 7002 -c
-c：代表连接的是redis集群

###五、Jedis
1. 添加Jedis依赖
```
		<!-- Redis客户端 -->
		<dependency>
			<groupId>redis.clients</groupId>
			<artifactId>jedis</artifactId>
		</dependency>
```
2. 连接单机版
```
	@Test
	public void testJedis() throws Exception {
		// 第一步：创建一个Jedis对象。需要指定服务端的ip及端口。
		Jedis jedis = new Jedis("192.168.10.179", 6379);
		// 第二步：使用Jedis对象操作数据库，每个redis命令对应一个方法。
		String result = jedis.get("hello");
		// 第三步：打印结果。
		System.out.println(result);
		// 第四步：关闭Jedis
		jedis.close();
	}
```
3. 连接单机版使用连接池
```
	@Test
	public void testJedisPool() throws Exception {
		// 第一步：创建一个JedisPool对象。需要指定服务端的ip及端口。
		JedisPool jedisPool = new JedisPool("192.168.10.179", 6379);
		// 第二步：从JedisPool中获得Jedis对象。
		Jedis jedis = jedisPool.getResource();
		// 第三步：使用Jedis操作redis服务器。
		String result = jedis.get("hello");
		System.out.println(result);
		// 第四步：操作完毕后关闭jedis对象，连接池回收资源。
		jedis.close();
		// 第五步：关闭JedisPool对象。
		jedisPool.close();
	}
```
4. 连接集群版
```
	@Test
	public void testJedisCluster() throws Exception {
		// 第一步：使用JedisCluster对象。需要一个Set<HostAndPort>参数。Redis节点的列表。
		Set<HostAndPort> nodes = new HashSet<>();
		nodes.add(new HostAndPort("192.168.10.179", 7001));
		nodes.add(new HostAndPort("192.168.10.179", 7002));
		nodes.add(new HostAndPort("192.168.10.179", 7003));
		nodes.add(new HostAndPort("192.168.10.179", 7004));
		nodes.add(new HostAndPort("192.168.10.179", 7005));
		nodes.add(new HostAndPort("192.168.10.179", 7006));
		JedisCluster jedisCluster = new JedisCluster(nodes);
		// 第二步：直接使用JedisCluster对象操作redis。在系统中单例存在。
		String result = jedisCluster.get("hello");
		// 第三步：打印结果
		System.out.println(result);
		// 第四步：系统关闭前，关闭JedisCluster对象。
		jedisCluster.close();
	}
```

###六、封装
常用的操作Redis的方法提取出一个接口，分别对应单机版和集群版创建两个实现类。
1. 接口定义
```
public interface JedisClient {
	String set(String key, String value);
	String get(String key);
	Boolean exists(String key);
	Long expire(String key, int seconds);
	Long ttl(String key);
	Long incr(String key);
	Long hset(String key, String field, String value);
	String hget(String key, String field);
	Long hdel(String key, String... field);
	Boolean hexists(String key, String field);
	List<String> hvals(String key);
	Long del(String key);
}
```
2. 单机版实现类
```
public class JedisClientPool implements JedisClient {
	
	private JedisPool jedisPool;

	public JedisPool getJedisPool() {
		return jedisPool;
	}

	public void setJedisPool(JedisPool jedisPool) {
		this.jedisPool = jedisPool;
	}

	@Override
	public String set(String key, String value) {
		Jedis jedis = jedisPool.getResource();
		String result = jedis.set(key, value);
		jedis.close();
		return result;
	}

	@Override
	public String get(String key) {
		Jedis jedis = jedisPool.getResource();
		String result = jedis.get(key);
		jedis.close();
		return result;
	}

	@Override
	public Boolean exists(String key) {
		Jedis jedis = jedisPool.getResource();
		Boolean result = jedis.exists(key);
		jedis.close();
		return result;
	}

	@Override
	public Long expire(String key, int seconds) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.expire(key, seconds);
		jedis.close();
		return result;
	}

	@Override
	public Long ttl(String key) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.ttl(key);
		jedis.close();
		return result;
	}

	@Override
	public Long incr(String key) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.incr(key);
		jedis.close();
		return result;
	}

	@Override
	public Long hset(String key, String field, String value) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.hset(key, field, value);
		jedis.close();
		return result;
	}

	@Override
	public String hget(String key, String field) {
		Jedis jedis = jedisPool.getResource();
		String result = jedis.hget(key, field);
		jedis.close();
		return result;
	}

	@Override
	public Long hdel(String key, String... field) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.hdel(key, field);
		jedis.close();
		return result;
	}

	@Override
	public Boolean hexists(String key, String field) {
		Jedis jedis = jedisPool.getResource();
		Boolean result = jedis.hexists(key, field);
		jedis.close();
		return result;
	}

	@Override
	public List<String> hvals(String key) {
		Jedis jedis = jedisPool.getResource();
		List<String> result = jedis.hvals(key);
		jedis.close();
		return result;
	}

	@Override
	public Long del(String key) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.del(key);
		jedis.close();
		return result;
	}

}
```
3. 集群版实现类
```
public class JedisClientCluster implements JedisClient {
	
	private JedisCluster jedisCluster;
	

	public JedisCluster getJedisCluster() {
		return jedisCluster;
	}

	public void setJedisCluster(JedisCluster jedisCluster) {
		this.jedisCluster = jedisCluster;
	}

	@Override
	public String set(String key, String value) {
		return jedisCluster.set(key, value);
	}

	@Override
	public String get(String key) {
		return jedisCluster.get(key);
	}

	@Override
	public Boolean exists(String key) {
		return jedisCluster.exists(key);
	}

	@Override
	public Long expire(String key, int seconds) {
		return jedisCluster.expire(key, seconds);
	}

	@Override
	public Long ttl(String key) {
		return jedisCluster.ttl(key);
	}

	@Override
	public Long incr(String key) {
		return jedisCluster.incr(key);
	}

	@Override
	public Long hset(String key, String field, String value) {
		return jedisCluster.hset(key, field, value);
	}

	@Override
	public String hget(String key, String field) {
		return jedisCluster.hget(key, field);
	}

	@Override
	public Long hdel(String key, String... field) {
		return jedisCluster.hdel(key, field);
	}

	@Override
	public Boolean hexists(String key, String field) {
		return jedisCluster.hexists(key, field);
	}

	@Override
	public List<String> hvals(String key) {
		return jedisCluster.hvals(key);
	}

	@Override
	public Long del(String key) {
		return jedisCluster.del(key);
	}

}
```
4. 封装代码测试
applicationContext-redis.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.2.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.2.xsd
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.2.xsd">
	
	<!-- 连接redis单机版 -->
	<bean id="jedisClientPool" class="cn.study.common.jedis.JedisClientPool">
		<property name="jedisPool" ref="jedisPool"></property>
	</bean>
	<bean id="jedisPool" class="redis.clients.jedis.JedisPool">
		<constructor-arg name="host" value="192.168.10.179"/>
		<constructor-arg name="port" value="6379"/>
	</bean>
	<!-- 连接redis集群 -->
	<!-- <bean id="jedisClientCluster" class="cn.study.common.jedis.JedisClientCluster">
		<property name="jedisCluster" ref="jedisCluster"/>
	</bean>
	<bean id="jedisCluster" class="redis.clients.jedis.JedisCluster">
		<constructor-arg name="nodes">
			<set>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.10.179"></constructor-arg>
					<constructor-arg name="port" value="7001"></constructor-arg>
				</bean> 
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.10.179"></constructor-arg>
					<constructor-arg name="port" value="7002"></constructor-arg>
				</bean> 
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.10.179"></constructor-arg>
					<constructor-arg name="port" value="7003"></constructor-arg>
				</bean> 
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.10.179"></constructor-arg>
					<constructor-arg name="port" value="7004"></constructor-arg>
				</bean> 
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.10.179"></constructor-arg>
					<constructor-arg name="port" value="7005"></constructor-arg>
				</bean> 
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.10.179"></constructor-arg>
					<constructor-arg name="port" value="7006"></constructor-arg>
				</bean> 
			</set>
		</constructor-arg>
	</bean> -->
</beans>
```
```
	@Test
	public void testJedisClient() throws Exception {
		//初始化Spring容器
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring/applicationContext-redis.xml");
		//从容器中获得JedisClient对象
		JedisClient jedisClient = applicationContext.getBean(JedisClient.class);
		String result = jedisClient.get("hello");
		System.out.println(result);
	}
```
**注意：单机版和集群版不能共存，使用单机版时注释集群版的配置。使用集群版，把单机版注释。**

###七、缓存同步
对数据库做增删改操作后只需要把对应缓存删除即可。

###工具
Redis Desktop Manager 
