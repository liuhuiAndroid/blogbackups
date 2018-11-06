###一、[MyBatis](http://www.mybatis.org/mybatis-3/) 介绍
>MyBatis is a first class persistence framework with support for custom SQL, stored procedures and advanced mappings. MyBatis eliminates almost all of the JDBC code and manual setting of parameters and retrieval of results. MyBatis can use simple XML or Annotations for configuration and map primitives, Map interfaces and Java POJOs to database records.

**MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs映射成数据库中的记录。**

###二、[MyBatis](http://www.mybatis.org/mybatis-3/) 入门
1. jdbc程序
```
	public static void main(String[] args) {
		Connection connection = null;
		PreparedStatement preparedStatement = null;
		ResultSet resultSet = null;
		try {
			// 加载数据库驱动
			Class.forName("com.mysql.jdbc.Driver");
			// 通过驱动管理类获取数据库链接
			connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8", "root", "root");
			// 定义sql语句 ?表示占位符
			String sql = "select * from user where username = ?";
			// 获取预处理statement
			preparedStatement = connection.prepareStatement(sql);
			// 设置参数，第一个参数为sql语句中参数的序号（从1开始），第二个参数为设置的参数值
			preparedStatement.setString(1, "张小明");
			// 向数据库发出sql执行查询，查询出结果集
			resultSet = preparedStatement.executeQuery();
			// 遍历查询结果集
			while (resultSet.next()) {
				System.out.println(resultSet.getString("id") + "  " + resultSet.getString("username"));
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			// 释放资源
			if (resultSet != null) {
				try {
					resultSet.close();
				} catch (SQLException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
			if (preparedStatement != null) {
				try {
					preparedStatement.close();
				} catch (SQLException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
			if (connection != null) {
				try {
					connection.close();
				} catch (SQLException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		}
	}
}
```
2. **使用jdbc操作数据库存在的问题**
1 . 数据库连接创建、释放频繁造成系统资源浪费，从而影响系统性能。如果使用数据库连接池可解决此问题。
2 . Sql语句在代码中硬编码，造成代码不易维护，实际应用中sql变化的可能较大，sql变动需要改变java代码。
3 . 使用preparedStatement向占有位符号传参数存在硬编码，因为sql语句的where条件不一定，可能多也可能少，修改sql还要修改代码，系统不易维护。
4 . 对结果集解析存在硬编码，sql变化导致解析代码变化，系统不易维护，如果能将数据库记录封装成POJO对象解析比较方便。

3. **MyBatis的入门程序**
![image.png](https://upload-images.jianshu.io/upload_images/1956963-fef3889ea9a3525c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
log4j.properties
```
log4j.rootLogger=DEBUG, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```
SqlMapConfig.xml  mybatis核心配置文件
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 和spring整合后 environments配置将废除    -->
    <environments default="development">
        <environment id="development">
            <!-- 使用jdbc事务管理 -->
            <transactionManager type="JDBC" />
            <!-- 数据库连接池 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8" />
                <property name="username" value="root" />
                <property name="password" value="root" />
            </dataSource>
        </environment>
    </environments>
    <!-- 加载映射文件 -->
    <mappers>
        <mapper resource="sqlmap/User.xml"/>
    </mappers>
</configuration>
```
User.java
```
public class User implements Serializable {
	private static final long serialVersionUID = 1L;
	private Integer id;
	private String username;
	private String sex;
	private Date birthday;
	private String address;
	get/set ...
}
```
User.xml    sql映射文件
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace：命名空间-->
<mapper namespace="test">
    <!-- id:statement的id 或者叫做sql的id-->
    <!-- parameterType:指定输入参数类型 -->
    <!-- resultType:指定输出结果类型，应该填写pojo的全路径 -->
    <!-- #{}：输入参数的占位符，相当于jdbc的？ -->
    <select id="findUserById" parameterType="int"
        resultType="com.study.mybatis.pojo.User">
        SELECT * FROM `user` WHERE id  = #{id}
    </select>
    <!-- //根据用户名称模糊查询用户列表
	#{}表示一个占位符号    select * from user where id = ？     ? ==  '五'
	${}表示拼接sql串    select * from user where username like '%五%'  -->
	<select id="findUserByUsername" parameterType="String" resultType="com.study.mybatis.pojo.User">
		select * from user where username like "%"#{haha}"%"
	</select>
	<!-- 添加用户 -->
	<insert id="insertUser" parameterType="com.study.mybatis.pojo.User">
		<!-- selectKey 标签实现主键返回 -->
		<!-- keyColumn:主键对应的表中的哪一列 -->
		<!-- keyProperty：主键对应的pojo中的哪一个属性 -->
		<!-- order：设置在执行insert语句前执行查询id的sql，还是在执行insert语句之后执行查询id的sql -->
		<!-- resultType：设置返回的id的类型 -->
		<selectKey keyColumn="id" keyProperty="id" resultType="Integer" order="AFTER">
			<!-- mysql的函数，返回auto_increment自增列新记录id值 -->
			select LAST_INSERT_ID()
		</selectKey>
		insert into user (username,birthday,address,sex) 
		values (#{username},#{birthday},#{address},#{sex})
	</insert>
	<!-- 更新 -->
	<update id="updateUserById" parameterType="com.study.mybatis.pojo.User">
		update user 
		set username = #{username},sex = #{sex},birthday = #{birthday},address = #{address}
		where id = #{id}
	</update>
	<!-- 删除 -->
	<delete id="deleteUserById" parameterType="Integer">
		delete from user 
		where id = #{vvvvv}
	</delete>
</mapper>
```
测试程序
```
public class Test {

	@Test
	public void testMybatis() throws Exception {
		//加载核心配置文件
		String resource = "SqlMapConfig.xml";
		InputStream in = Resources.getResourceAsStream(resource);
		//创建SqlSessionFactory
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
		//创建SqlSession
		SqlSession sqlSession = sqlSessionFactory.openSession();
		//执行Sql语句 
		User user = sqlSession.selectOne("test.findUserById", 10);
		System.out.println(user);
	}
	
	//根据用户名称模糊查询用户列表
	@Test
	public void testfindUserByUsername() throws Exception {
		//加载核心配置文件
		String resource = "SqlMapConfig.xml";
		InputStream in = Resources.getResourceAsStream(resource);
		//创建SqlSessionFactory
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
		//创建SqlSession
		SqlSession sqlSession = sqlSessionFactory.openSession();
		//执行Sql语句 
		List<User> users = sqlSession.selectList("test.findUserByUsername", "五");
		for (User user2 : users) {
			System.out.println(user2);
		}
	}
	
	//添加用户
	@Test
	public void testInsertUser() throws Exception {
		//加载核心配置文件
		String resource = "sqlMapConfig.xml";
		InputStream in = Resources.getResourceAsStream(resource);
		//创建SqlSessionFactory
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
		//创建SqlSession
		SqlSession sqlSession = sqlSessionFactory.openSession();
		//执行Sql语句 
		User user = new User();
		user.setUsername("鱼虾");
		user.setBirthday(new Date());
		user.setAddress("地址");
		user.setSex("男");
		int i = sqlSession.insert("test.insertUser", user);
		sqlSession.commit();
		System.out.println(user.getId());
	}
	
	//更新用户
	@Test
	public void testUpdateUserById() throws Exception {
		//加载核心配置文件
		String resource = "SqlMapConfig.xml";
		InputStream in = Resources.getResourceAsStream(resource);
		//创建SqlSessionFactory
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
		//创建SqlSession
		SqlSession sqlSession = sqlSessionFactory.openSession();
		//执行Sql语句 
		User user = new User();
		user.setId(27);
		user.setUsername("武侠");
		user.setBirthday(new Date());
		user.setAddress("地址2");
		user.setSex("女");
		int i = sqlSession.update("test.updateUserById", user);
		sqlSession.commit();
	}
	
	//删除
	@Test
	public void testDelete() throws Exception {
		//加载核心配置文件
		String resource = "SqlMapConfig.xml";
		InputStream in = Resources.getResourceAsStream(resource);
		//创建SqlSessionFactory
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
		//创建SqlSession
		SqlSession sqlSession = sqlSessionFactory.openSession();
		sqlSession.delete("test.deleteUserById", 27);
		sqlSession.commit();
	}

}
```
4. **MyBatis 解决jdbc编程的问题**
1 \.数据库连接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库连接池可解决此问题。
解决：在SqlMapConfig.xml中配置数据连接池，使用连接池管理数据库链接。
2 \.Sql语句写在代码中造成代码不易维护，实际应用sql变化的可能较大，sql变动需要改变java代码。
解决：将Sql语句配置在XXXXmapper.xml文件中与java代码分离。
3 \.向sql语句传参数麻烦，因为sql语句的where条件不一定，可能多也可能少，占位符需要和参数一一对应。
解决：Mybatis自动将java对象映射至sql语句，通过statement中的parameterType定义输入参数的类型。
4 \.对结果集解析麻烦，sql变化导致解析代码变化，且解析前需要遍历，如果能将数据库记录封装成pojo对象解析比较方便。
解决：Mybatis自动将sql执行结果映射至java对象，通过statement中的resultType定义输出结果的类型。

5. **MyBatis 与 Hibernate不同**
这个问题得深入研究 暂时不了解

###三、Dao的开发方法
使用MyBatis开发Dao，通常有两个方法，即原始Dao开发方法和Mapper动态代理开发方法。
1. **原始Dao的开发方法**
原始Dao开发方法需要程序员编写Dao接口和Dao实现类。
在入门程序基础上开发:
Dao接口
```
public interface UserDao {
	public User selectUserById(Integer id);
}
```
Dao实现类
```
public class UserDaoImpl implements UserDao {

	private SqlSessionFactory sqlSessionFactory;
	public UserDaoImpl(SqlSessionFactory sqlSessionFactory) {
		this.sqlSessionFactory = sqlSessionFactory;
	}
	
	public User selectUserById(Integer id){
		SqlSession sqlSession = sqlSessionFactory.openSession();
		User user = sqlSession.selectOne("test.findUserById", id);
		sqlSession.close();
		return user;
	}
}
```
Dao测试
```
public class TestDao {
	public SqlSessionFactory sqlSessionFactory;
	
	@Before
	public void before() throws Exception {
		//加载核心配置文件
		String resource = "SqlMapConfig.xml";
		InputStream in = Resources.getResourceAsStream(resource);
		//创建SqlSessionFactory
		sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
	}
	
	@Test
	public void testSelectById() throws Exception {
		UserDao userDao = new UserDaoImpl(sqlSessionFactory);
		User user = userDao.selectUserById(26);
		System.out.println(user);
	}
}
```
原始Dao开发中存在以下问题
- Dao方法体存在重复代码：通过SqlSessionFactory创建SqlSession，调用SqlSession的数据库操作方法
- 调用SqlSession的数据库操作方法需要指定Statement的id，这里存在硬编码，不得于开发维护。

2. **接口的动态代理方式**
**Mybatis官方推荐使用Mapper代理方法开发Mapper接口**
Mapper接口开发方法只需要程序员编写Mapper接口（相当于Dao接口），由Mybatis框架根据接口定义创建接口的动态代理对象，代理对象的方法体同上边Dao接口实现类方法。
Mapper接口开发需要遵循以下规范：
1、Mapper.xml文件中的namespace与mapper接口的类路径相同。
2、Mapper接口方法名和Mapper.xml中定义的每个statement的id相同 
3、Mapper接口方法的输入参数类型和Mapper.xml中定义的每个sql 的ParameterType的类型相同
4、Mapper接口方法的输出参数类型和Mapper.xml中定义的每个sql的ResultType的类型相同

UserMapper.xml
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace：命名空间-->
<mapper namespace="com.study.mybatis.mapper.UserMapper">
    <!-- id:statement的id 或者叫做sql的id-->
    <!-- parameterType:声明输入参数的类型 -->
    <!-- resultType:声明输出结果的类型，应该填写pojo的全路径 -->
    <!-- #{}：输入参数的占位符，相当于jdbc的？ -->
    <select id="findUserById" parameterType="int"
        resultType="com.study.mybatis.pojo.User">
        SELECT * FROM `user` WHERE id  = #{id}
    </select>
    
    <!-- //根据用户名称模糊查询用户列表
	#{}    select * from user where id = ？    占位符  ? ==  '五'
	${}    select * from user where username like '%五%'  字符串拼接  -->
	<select id="findUserByUsername" parameterType="String" resultType="com.study.mybatis.pojo.User">
		select * from user where username like "%"#{haha}"%"
	</select>
	
	<!-- 添加用户 -->
	<insert id="insertUser" parameterType="com.study.mybatis.pojo.User">
		<selectKey keyProperty="id" resultType="Integer" order="AFTER">
			select LAST_INSERT_ID()
		</selectKey>
		insert into user (username,birthday,address,sex) 
		values (#{username},#{birthday},#{address},#{sex})
	</insert>
	
	<!-- 更新 -->
	<update id="updateUserById" parameterType="com.study.mybatis.pojo.User">
		update user 
		set username = #{username},sex = #{sex},birthday = #{birthday},address = #{address}
		where id = #{id}
	</update>
	
	<!-- 删除 -->
	<delete id="deleteUserById" parameterType="Integer">
		delete from user 
		where id = #{vvvvv}
	</delete>
	
</mapper>
```
UserMapper 接口文件
```
public interface UserMapper {
	public User findUserById(Integer id);	
}
```
加载UserMapper.xml文件
SqlMapConfig.xml
```
    <!-- 加载映射文件 Mapper的位置 -->
    <mappers>
        <!-- 注意是扫描包 -->
        <package name="com.study.mybatis.mapper" />
    </mappers>
```
测试
```
public class TestMapper {

	@Test
	public void testMapper() throws Exception {
		//加载核心配置文件
		String resource = "SqlMapConfig.xml";
		InputStream in = Resources.getResourceAsStream(resource);
		//创建SqlSessionFactory
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
		//创建SqlSession
		SqlSession sqlSession = sqlSessionFactory.openSession();
		//SqlSEssion帮我生成一个实现类  （给接口）
		UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
		User user = userMapper.findUserById(10);
		System.out.println(user);
	}
	
}
```

###四、SqlMapConfig.xml文件说明
1. SqlMapConfig.xml中配置的内容和顺序
**properties**（属性）
settings（全局配置参数）
**typeAliases**（类型别名）
typeHandlers（类型处理器）
objectFactory（对象工厂）
plugins（插件）
environments（环境集合属性对象）
environment（环境子属性对象）
transactionManager（事务管理）
dataSource（数据源）
**mappers**（映射器）
2. **properties**
db.properties
```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8
jdbc.username=root
jdbc.password=root
```
SqlMapConfig.xml
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	
	<!-- 是用resource属性加载外部配置文件 -->
	<properties resource="db.properties">
		<!-- 在properties内部用property定义属性 -->
		<!-- 如果外部配置文件有该属性，则内部定义属性被外部属性覆盖 -->
		<property name="jdbc.username" value="root123" />
		<property name="jdbc.password" value="root123" />
	</properties>
	
    <!-- 和spring整合后 environments配置将废除    -->
    <environments default="development">
        <environment id="development">
            <!-- 使用jdbc事务管理 -->
            <transactionManager type="JDBC" />
            <!-- 数据库连接池 -->
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}" />
                <property name="url" value="${jdbc.url}" />
                <property name="username" value="${jdbc.username}" />
                <property name="password" value="${jdbc.password}" />
            </dataSource>
        </environment>
    </environments>
    <!-- 加载映射文件 Mapper的位置 -->
    <mappers>
        <package name="com.study.mybatis.mapper" />
    </mappers>
    
</configuration>
```
3. **typeAliases**
```
	<typeAliases>
		<!-- 单个别名定义 -->
		<typeAlias alias="user" type="com.study.mybatis.pojo.User" />
		<!-- 批量别名定义，扫描整个包下的类，别名为类名（大小写不敏感） -->
		<package name="com.study.mybatis.pojo" />
	</typeAliases>
```
在mapper.xml配置文件中，就可以使用设置的别名了
别名大小写不敏感
4. **mappers**
```
    <mappers>
    	<!-- 使用相对于类路径的资源 -->
        <mapper resource="sqlmap/User.xml" />
        <!-- 使用mapper接口类路径 -->
        <!-- 此种方法要求mapper接口名称和mapper映射文件名称相同，且放在同一个目录中 -->
        <mapper class="com.study.mybatis.mapper.UserMapper"/>
        <!-- 注册指定包下的所有mapper接口 -->
        <!-- 此种方法要求mapper接口名称和mapper映射文件名称相同，且放在同一个目录中。 -->
        <package name="com.study.mybatis.mapper"/>
    </mappers>
```

###五、输入映射和输出映射
1. **输入参数映射**
parameterType
- 传递简单类型
- 传递POJO对象
- 传递POJO包装对象
```
    <!-- 根据用户名模糊查询 -->
	<select id="findUserByQueryVo" parameterType="QueryVo" resultType="User">
		select * from user where username like "%"#{user.username}"%"
	</select>
```
```
public interface UserMapper {

	public User findUserById(Integer id);
	
	public List<User> findUserByQueryVo(QueryVo vo);
	
}
```
测试：
```
	@Test
	public void testMapperQueryVo() throws Exception {
		//加载核心配置文件
		String resource = "SqlMapConfig.xml";
		InputStream in = Resources.getResourceAsStream(resource);
		//创建SqlSessionFactory
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
		//创建SqlSession
		SqlSession sqlSession = sqlSessionFactory.openSession();
		//SqlSEssion帮我生成一个实现类  （给接口）
		UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
		QueryVo vo = new QueryVo();
		User user = new User();
		user.setUsername("五");
		vo.setUser(user);
		List<User> userList = userMapper.findUserByQueryVo(vo);
		for (User u : userList) {
			System.out.println(u);
		}
	}
```
2. **返回值映射**
- 输出简单类型
sql：select count(*) from user
```
     <select id="countUser" resultType="Integer">
	 	select count(1) from user
	 </select>
```
```
	public Integer countUser();
```
```
	@Test
	public void testCountUser() throws Exception {
		//加载核心配置文件
		String resource = "sqlMapConfig.xml";
		InputStream in = Resources.getResourceAsStream(resource);
		//创建SqlSessionFactory
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
		//创建SqlSession
		SqlSession sqlSession = sqlSessionFactory.openSession();
		UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
		Integer i = userMapper.countUser();
		System.out.println(i);
	}
```
- 输出POJO对象

- 输出POJO列表

- ResultMap
ResultType可以指定将查询结果映射为POJO，但需要POJO的属性名和sql查询的列名一致方可映射成功。
如果sql查询字段名和POJO的属性名不一致，可以通过ResultMap将字段名和属性名作一个对应关系 ，ResultMap实质上还需要将查询结果映射到POJO对象中。
ResultMap可以实现将查询结果映射为复杂类型的POJO，比如在查询结果映射对象中包括POJO和list实现一对一查询和一对多查询。
```
public class Order implements Serializable{

	private static final long serialVersionUID = 1L;

	private Integer id;
	private Integer userId;
	private String number;
	private Date createtime;
	private String note;
	private User user;

	get/set ...
}
```
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.study.mybatis.mapper.OrderMapper">
	 
	<!-- 解决sql查询列(user_id)和Order类属性(userId)不一致的问题 -->
	<resultMap type="Order" id="order">
		<result column="user_id" property="userId"/>
	</resultMap>
	
	<select id="selectOrderList" resultMap="order">
		SELECT id, user_id, number, createtime, note from orders
	</select>
	
</mapper>
```
```
public interface OrderMapper {

	List<Order> selectOrderList();
	
}
```
```
	@Test
	public void testOrderList() throws Exception {
		//加载核心配置文件
		String resource = "SqlMapConfig.xml";
		InputStream in = Resources.getResourceAsStream(resource);
		//创建SqlSessionFactory
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
		//创建SqlSession
		SqlSession sqlSession = sqlSessionFactory.openSession();
		OrderMapper mapper = sqlSession.getMapper(OrderMapper.class);
		List<Order> ordersList = mapper.selectOrderList();
		for (Order order : ordersList) {
			System.out.println(order);
		}
	}
```

###六、动态sql
1. If标签
2. Where标签
3. Sql片段
```
	<!-- Sql片段 -->
	<sql id="selector">
		select * from user
	</sql>
	
	 <select id="selectUserBySexAndUsername" parameterType="User" resultType="User">
	 	<!-- 如果要使用别的Mapper.xml配置的sql片段，可以在refid前面加上对应的Mapper.xml的namespace -->
	 	<include refid="selector"/>
	 	<!-- 如果不加<where>, 需要加上Where 1=1 -->
	 	<where>
		 	<if test="sex != null and sex != ''">
		 		 and sex = #{sex} 
		 	</if>
		 	<if test="username != null and username != ''">
			 	 and username = #{username}
		 	</if>
	 	</where>
	 </select>
```
```
	public List<User> selectUserBySexAndUsername(User user);
```
```
	@Test
	public void testfindUserBySexAndUsername() throws Exception {
		//加载核心配置文件
		String resource = "sqlMapConfig.xml";
		InputStream in = Resources.getResourceAsStream(resource);
		//创建SqlSessionFactory
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
		//创建SqlSession
		SqlSession sqlSession = sqlSessionFactory.openSession();
		UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
		User user = new User();
		user.setSex("1");
		user.setUsername("张小明");
		List<User> users = userMapper.selectUserBySexAndUsername(user);
		for (User user2 : users) {
			System.out.println(user2);
		}
	}
```
4. foreach标签
向sql传递数组或List，mybatis使用foreach解析，如：select * from user where id in(1,10,24)
```
public class QueryVo implements Serializable{
	private static final long serialVersionUID = 1L;
	private User user;
	List<Integer> idsList;
	Integer[] ids;
	
	public User getUser() {
		return user;
	}
	public void setUser(User user) {
		this.user = user;
	}
	public List<Integer> getIdsList() {
		return idsList;
	}
	public void setIdsList(List<Integer> idsList) {
		this.idsList = idsList;
	}
	public Integer[] getIds() {
		return ids;
	}
	public void setIds(Integer[] ids) {
		this.ids = ids;
	}
}
```
```
	 <select id="selectUserByIds" parameterType="QueryVo" resultType="User">
	 	<include refid="selector"/>
	 	<where>
	 		<foreach collection="list" item="id" separator="," open="id in (" close=")">
	 			#{id}
	 		</foreach>
	 	</where>
	 </select>
```
```
三选一
//	public List<User> selectUserByIds(Integer[] ids); // parameterType="Integer" collection="array"
//	public List<User> selectUserByIds(List<Integer> ids); // parameterType="Integer" collection="list"
//	public List<User> selectUserByIds(QueryVo vo); // parameterType="QueryVo" collection="idsList"
```
```
	@Test
	public void testfindUserIDs() throws Exception {
		//加载核心配置文件
		String resource = "sqlMapConfig.xml";
		InputStream in = Resources.getResourceAsStream(resource);
		//创建SqlSessionFactory
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
		//创建SqlSession
		SqlSession sqlSession = sqlSessionFactory.openSession();
		UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
		List<Integer> idsList  = new ArrayList<>();
		idsList.add(16);
		idsList.add(22);
		idsList.add(24);
		Integer[] idsInteger = new Integer[3];
		idsInteger[0] = 16;
		idsInteger[1] = 22;
		idsInteger[2] = 24;
		QueryVo queryVo = new QueryVo();
		queryVo.setIdsList(idsList);
		List<User> users = userMapper.selectUserByIds(queryVo);
		for (User user2 : users) {
			System.out.println(user2);
		}
	}
```

###七、关联查询
1. **一对一关联**
`
SELECT
	o.id,
	o.user_id userId,
	o.number,
	o.createtime,
	o.note,
	u.username,
	u.address
FROM
	order o
LEFT JOIN user u ON o.user_id = u.id
`
- 方法一：使用resultType
```
public class OrderUser extends Order{
    user的属性
    ...  ...
}
```
- 方法二：使用resultMap
```
public class Order implements Serializable{
    private User user;
    ...
}
```
```
	<resultMap type="Order" id="orderUser">
	 	<result column="id" property="id"/>
	 	<result column="user_id" property="userId"/>
	 	<result column="number" property="number"/>
	 	<!-- 一对一 -->
	 	<association property="user" javaType="User">
	 		<id column="user_id" property="id"/>
	 		<result column="username" property="username"/>
	 	</association>
	 </resultMap>
	<select id="selectOrder" resultMap="orderUser">
	 	SELECT 
	 	o.id,
	    o.user_id, 
	    o.number,
	 	o.createtime,
	 	u.username 
	 	FROM orders o 
	 	left join user u 
	 	on o.user_id = u.id
	 </select>
```
```
	public List<Order> selectOrder();
```
```
	@Test
	public void testOrderUser() throws Exception {
		String resource = "SqlMapConfig.xml";
		InputStream in = Resources.getResourceAsStream(resource);
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
		SqlSession sqlSession = sqlSessionFactory.openSession();
		OrderMapper orderMapper = sqlSession.getMapper(OrderMapper.class);
		List<Order> selectOrderList = orderMapper.selectOrder();
		for (Order orders : selectOrderList) {
			System.out.println(orders);
		}
	}
```
2. **一对多关联**
`
SELECT
	u.id,
	u.username,
	u.birthday,
	u.sex,
	u.address,
	o.id oid,
	o.number,
	o.createtime,
	o.note
FROM
	user u
LEFT JOIN order o ON u.id = o.user_id
`
```
public class User implements Serializable{
	private List<Order> ordersList;
	... ...
}
```
```
	<resultMap type="User" id="user">
		<id column="user_id" property="id"/>
		<result column="username" property="username"/>
		<!-- 一对多 -->
		<collection property="ordersList" ofType="Order">
			<id column="id" property="id"/>
			<result column="number" property="number"/>
		</collection>
	</resultMap>
	<select id="selectUserList" resultMap="user">
		SELECT 
	 	o.id,
	    o.user_id, 
	    o.number,
	 	o.createtime,
	 	u.username 
	 	FROM user u
	 	left join orders o 
	 	on o.user_id = u.id
	</select>
```
```
	public List<User> selectUserList();
```
```
	@Test
	public void testUserList() throws Exception {
		String resource = "SqlMapConfig.xml";
		InputStream in = Resources.getResourceAsStream(resource);
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
		SqlSession sqlSession = sqlSessionFactory.openSession();
		OrderMapper orderMapper = sqlSession.getMapper(OrderMapper.class);
		List<User> users = orderMapper.selectUserList();
		for (User user : users) {
			System.out.println(user);
		}
	}
```
###八、[MyBatis](http://www.mybatis.org/mybatis-3/)整合Spring

MyBatis Spring的配置文件sqlmapConfig.xml
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<!-- 设置别名 -->
	<typeAliases>
		<!-- 2. 指定扫描包，会把包内所有的类都设置别名，别名的名称就是类名，大小写不敏感 -->
		<package name="com.study.mybatis.pojo" />
	</typeAliases>
	
	<mappers>
		<package name="com.study.mybatis.mapper"/>
	</mappers>
</configuration>
```
applicationContext.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd">

	<context:property-placeholder location="classpath:db.properties"/>
	
	<!-- 数据库连接池 -->
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
		destroy-method="close">
		<property name="driverClassName" value="${jdbc.driver}" />
		<property name="url" value="${jdbc.url}" />
		<property name="username" value="${jdbc.username}" />
		<property name="password" value="${jdbc.password}" />
		<property name="maxActive" value="10" />
		<property name="maxIdle" value="5" />
	</bean>
	
	<!-- Mybatis的工厂 -->
	<bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"/>
		<!-- 核心配置文件的位置 -->
		<property name="configLocation" value="classpath:sqlMapConfig.xml"/>
	</bean>
	
	<!-- Mapper动态代理开发   扫描 -->
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<!-- 基本包 -->
		<property name="basePackage" value="com.study.mybatis.mapper"/>
	</bean>
	
</beans>
```
db.properties
```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8
jdbc.username=root
jdbc.password=root
```
log4j.properties
```
log4j.rootLogger=DEBUG, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```
先使用下面介绍的Mybatis 自动生成代码插件生成Mapper和POJO
测试代码：
```
public class TestMybatis {
	
	private ApplicationContext context;

	@Before
	public void setUp() throws Exception {
		this.context = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
	}

	@Test
	public void testInsert() {
		// 获取Mapper
		UserMapper userMapper = this.context.getBean(UserMapper.class);
		User user = new User();
		user.setUsername("鱼虾");
		user.setSex("1");
		user.setBirthday(new Date());
		user.setAddress("上海");
		userMapper.insert(user);
	}

	@Test
	public void testSelectByExample() {
		// 获取Mapper
		UserMapper userMapper = this.context.getBean(UserMapper.class);
		// 创建User对象扩展类，用户设置查询条件
		UserExample example = new UserExample();
		example.createCriteria().andUsernameLike("%张%");
		// 查询数据
		List<User> list = userMapper.selectByExample(example);
		System.out.println(list.size());
	}

	@Test
	public void testSelectByPrimaryKey() {
		// 获取Mapper
		UserMapper userMapper = this.context.getBean(UserMapper.class);
		User user = userMapper.selectByPrimaryKey(1);
		System.out.println(user);
	}
	
}
```

###九、Mybatis 自动生成代码插件
- ##### [mybatis generator github](https://github.com/mybatis/generator)
首先需要在pom.xml文件添加mybatis-generator依赖包以及插件
```
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.3.2</version>
</dependency>
```
在build的plugins中添加：
```
    <build>
        <plugins>
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.2</version>
                <configuration>
                    　　<!-- mybatis用于生成代码的配置文件 -->
                    　　<configurationFile>src/main/resources/generatorConfig.xml</configurationFile>
                    <verbose>true</verbose>
                    <overwrite>true</overwrite>
                </configuration>
                <dependencies>
                    <!--此处添加一个mysql-connector-java依赖可以防止找不到jdbc Driver-->
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>6.0.6</version>
                        <scope>runtime</scope>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
```
配置generatorConfig.xml文件
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
    PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
    "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <context id="DB2Tables" targetRuntime="MyBatis3">
        <commentGenerator>
            <property name="suppressDate" value="false"/>
            <!-- 是否自动生成的注释 -->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>

        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC&amp;nullNamePatternMatchesAll=true"
                        userId="root"
                        password="root">
        </jdbcConnection>

        <!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，为 true时把JDBC DECIMAL 和
			NUMERIC 类型解析为java.math.BigDecimal -->
        <javaTypeResolver >
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>

        <!-- targetProject:生成PO类的位置 -->
        <javaModelGenerator targetPackage="com.study.mybatis.pojo" targetProject="src/main/java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <!-- targetProject:mapper映射文件生成的位置 -->
        <sqlMapGenerator targetPackage="com.study.mybatis.mapper"
                         targetProject="src/main/java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
        </sqlMapGenerator>

        <!-- targetPackage：mapper接口生成的位置 -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.study.mybatis.mapper"
                             targetProject="src/main/java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
        </javaClientGenerator>

        <!--<table schema="DB2ADMIN" tableName="ALLTYPES" domainObjectName="Customer" >
            <property name="useActualColumnNames" value="true"/>
            <generatedKey column="ID" sqlStatement="DB2" identity="true" />
            <columnOverride column="DATE_FIELD" property="startDate" />
            <ignoreColumn column="FRED" />
            <columnOverride column="LONG_VARCHAR_FIELD" jdbcType="VARCHAR" />
        </table>-->
        <!-- 指定数据库表 -->
        <table schema="" tableName="user"></table>
        <table schema="" tableName="orders"></table>
    </context>
</generatorConfiguration>
```
到此为止，所有的配置已完毕。
- ##### [mybatis-generator-plugin github](https://github.com/itfsw/mybatis-generator-plugin)
非官方的对代码生成插件的拓展：
可在generatorConfig.xml中进行配置，具体需要参考github：
```
        <!-- 查询单条数据插件 -->
        <plugin type="com.itfsw.mybatis.generator.plugins.SelectOneByExamplePlugin"/>
        <!-- Example Criteria 增强插件 -->
        <plugin type="com.itfsw.mybatis.generator.plugins.ExampleEnhancedPlugin"/>
        <!-- 数据Model属性对应Column获取插件 -->
        <plugin type="com.itfsw.mybatis.generator.plugins.ModelColumnPlugin"/>
        <!-- 查询结果选择性返回插件 -->
        <plugin type="com.itfsw.mybatis.generator.plugins.SelectSelectivePlugin" />
```
###拓展：使用pagehelper进行分页操作
#### [Mybatis-PageHelper](https://github.com/pagehelper/Mybatis-PageHelper)
#### [pagehelper-spring-boot](https://github.com/pagehelper/pagehelper-spring-boot)
**PageHelper.startPage 静态方法调用**
在你需要进行分页的 MyBatis 查询方法前调用 PageHelper.startPage 静态方法即可，紧跟在这个方法后的第一个MyBatis 查询方法会被进行分页。
```
//获取第1页，10条内容，默认查询总数count
PageHelper.startPage(1, 10);
```

###可能遇到的问题
* Mapper映射文件不存在异常
在pom文件添加资源配置
```
<project>
	<build>
		<resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
	</build>
</project>
```

###资料
[MyBatis官方文档](http://www.mybatis.org/mybatis-3/)
[MyBatis中文文档](http://www.mybatis.org/mybatis-3/zh/index.html)
