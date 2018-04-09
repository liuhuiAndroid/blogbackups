### 商品详情页添加redis缓存的例子
* 安装redis
```
如果没有gcc需要在线安装。yum install gcc-c++
安装步骤：
第一步：redis的源码包 redis-3.0.0.tar.gz上传到linux系统。
第二步：解压缩redis。
第三步：编译。进入redis源码目录。make 
第四步：安装。make install PREFIX=/usr/local/redis
PREFIX参数指定redis的安装目录。一般软件安装到/usr目录下
注意：可能会报错：You need tcl 8.5 or newer in order to run the Redis test
需要安装tcl： yum -y install tcl
第五步：前端启动。在redis的安装目录下直接启动redis-server
第六步：后端启动。
cp redis.conf /usr/local/redis/bin/
修改配置文件redis.conf：daemonize yes
 ./redis-server redis.conf
查看redis进程：ps aux|grep redis
```
* 添加jar包
```
		<dependency>
			<groupId>redis.clients</groupId>
			<artifactId>jedis</artifactId>
		</dependency>
```
applicationContext-redis.xml
```
	<!-- 连接redis单机版 -->
	<bean id="jedisClientPool" class="cn.e3mall.common.jedis.JedisClientPool">
		<property name="jedisPool" ref="jedisPool"></property>
	</bean>
	<bean id="jedisPool" class="redis.clients.jedis.JedisPool">
		<constructor-arg name="host" value="192.168.10.179"/>
		<constructor-arg name="port" value="6379"/>
	</bean>
```
* 添加缓存
```
	@Test
	public void testAdd() throws Exception {
		//初始化spring容器
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring/applicationContext-redis.xml");
		//从容器中获得JmsTemplate对象。
		JedisClient jedisClient = applicationContext.getBean(JedisClient.class);
		try {
			jedisClient.set("REDIS_ITEM_PRE" + ":" + "itemId" + ":BASE", "测试");
			//设置过期时间
			jedisClient.expire("REDIS_ITEM_PRE" + ":" + "itemId" + ":BASE", 3600);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
```
* 从缓存中获取数据
```
	@Test
	public void testGet() throws Exception {
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring/applicationContext-redis.xml");
		//从容器中获得JmsTemplate对象。
		JedisClient jedisClient = applicationContext.getBean(JedisClient.class);
		//查询缓存
		try {
			String string = jedisClient.get("REDIS_ITEM_PRE" + ":" + "itemId" + ":BASE");
			System.out.println(string);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
```
