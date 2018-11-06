### 二
模块化开发
- 基础模块
- 业务层共用模块
- 应用单独的模块

为什么模块化开发
- 高内聚，低耦合
- 并行开发，提高开发效率
- 轮子重复使用

如何划分模块
- 业务层次 dao service
- 功能划分
- 重复使用

![image.png](https://upload-images.jianshu.io/upload_images/1956963-42970ddb81422fa1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

数据库设计
管理端 manager
- 产品表
编号、名称、收益率、锁定期、状态、起投金额、投资步长、备注、创建时间、创建者、更新时间、更新者
销售端 seller
- 订单表
订单编号、渠道编号、产品编号、用户编号、外部订单编号、类型、状态、金额、备注、创建时间、更新时间

### 三
```
POST测试新增产品
http://localhost:8081/manager/products
Content-Type application/json
{
	"id" : "001",
	"name" : "金融1号",
	"thresholdAmount" : 10,
	"stepAmount" : 1,
	"lockTerm" : 0,
	"rewardRate" : 3.86,
	"status" : "AUDITING",
	"memo" : null,
	"createAt" : null,
	"updateAt" : null,
	"createUser":  null,
	"updateUser" : null
}

GET测试查询产品
http://localhost:8081/manager/products/001

GET测试查询产品
http://localhost:8081/manager/products?minRewardRate=3&maxRewardRate=5&pageNum=0
```
错误处理
- 用户友好的错误说明
- 统一处理、简化业务代码
- 异常标准化
[Spring Boot Reference Guide](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-error-handling)

# 四
[swagger-ui](http://localhost:8081/manager/swagger-ui.html)

# 五
JSONRPC

# 六

# 七

# 八

# 九
