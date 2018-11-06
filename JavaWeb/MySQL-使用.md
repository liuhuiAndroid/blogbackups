###一、基础操作
1. 创建数据库
create database mall_test character set utf8;
2. 删除数据库
drop database mall_test;
3. 使用数据库
use mall_test;
4. 查看当前正在操作的库
select database();
5. 创建一张表
create table 表名(
	字段名 类型(长度) [约束],
	字段名 类型(长度) [约束],
	字段名 类型(长度) [约束]
);
6. 查看数据库表
show tables;
7. 查看表的结构
desc user;
8. 删除一张表
drop table user;

###二、Delete与Truncate的区别
Delete删除的时候是一条一条的删除记录，它配合事务，可以将删除的数据找回。
Truncate删除，它是将整个表摧毁，然后再创建一张一模一样的表。它删除的数据无法找回。

###三、查询
select  要查询的字段
from  要查询的表
where
group by 
having  分组后带有条件只能使用having
order by 必须放到最后面

###四、数据库连接池druid
[**Github druid**](https://github.com/alibaba/druid)
[**druid常见问题**](https://github.com/alibaba/druid/wiki/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)
31\. **如何在Spring Boot中集成Druid连接池和监控？**
使用Druid Spring Boot Starter，文档地址：[https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter)
[**druid 源码分析与学习**](https://blog.csdn.net/herriman/article/details/51759479)


