###一、[MyBatis](http://www.mybatis.org/mybatis-3/) 介绍
>MyBatis is a first class persistence framework with support for custom SQL, stored procedures and advanced mappings. MyBatis eliminates almost all of the JDBC code and manual setting of parameters and retrieval of results. MyBatis can use simple XML or Annotations for configuration and map primitives, Map interfaces and Java POJOs to database records.

**MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs映射成数据库中的记录。**

###二、[MyBatis](http://www.mybatis.org/mybatis-3/) 入门
1. **使用jdbc操作数据库存在的问题**
1 . 数据库连接创建、释放频繁造成系统资源浪费，从而影响系统性能。如果使用数据库连接池可解决此问题。
2 . Sql语句在代码中硬编码，造成代码不易维护，实际应用中sql变化的可能较大，sql变动需要改变java代码。
3 . 使用preparedStatement向占有位符号传参数存在硬编码，因为sql语句的where条件不一定，可能多也可能少，修改sql还要修改代码，系统不易维护。
4 . 对结果集解析存在硬编码，sql变化导致解析代码变化，系统不易维护，如果能将数据库记录封装成POJO对象解析比较方便。

2. **MyBatis的入门程序**

###三、Dao的开发方法
1. **原始dao的开发方法**

2. **接口的动态代理方式**

###四、SqlMapConfig.xml文件说明

###五、输入映射和输出映射
1. **输入参数映射**

2. **返回值映射**

###六、动态sql

###七、关联查询
1. **一对一关联**

2. **一对多关联**

###八、[MyBatis](http://www.mybatis.org/mybatis-3/)整合spring

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
