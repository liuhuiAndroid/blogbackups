###一对多|多对一
1.关系表达
* 表中的表达
```
-- ----------------------------
-- Table structure for cst_customer
-- ----------------------------
DROP TABLE IF EXISTS `cst_customer`;
CREATE TABLE `cst_customer` (
  `cust_id` bigint(32) NOT NULL AUTO_INCREMENT COMMENT '客户编号(主键)',
  `cust_name` varchar(32) NOT NULL COMMENT '客户名称(公司名称)',
  `cust_user_id` bigint(32) DEFAULT NULL COMMENT '负责人id',
  `cust_create_id` bigint(32) DEFAULT NULL COMMENT '创建人id',
  `cust_source` varchar(32) DEFAULT NULL COMMENT '客户信息来源',
  `cust_industry` varchar(32) DEFAULT NULL COMMENT '客户所属行业',
  `cust_level` varchar(32) DEFAULT NULL COMMENT '客户级别',
  `cust_linkman` varchar(64) DEFAULT NULL COMMENT '联系人',
  `cust_phone` varchar(64) DEFAULT NULL COMMENT '固定电话',
  `cust_mobile` varchar(16) DEFAULT NULL COMMENT '移动电话',
  PRIMARY KEY (`cust_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Table structure for cst_linkman
-- ----------------------------
DROP TABLE IF EXISTS `cst_linkman`;
CREATE TABLE `cst_linkman` (
  `lkm_id` bigint(32) NOT NULL AUTO_INCREMENT COMMENT '联系人编号(主键)',
  `lkm_name` varchar(16) DEFAULT NULL COMMENT '联系人姓名',
  `lkm_cust_id` bigint(32) NOT NULL COMMENT '客户id',
  `lkm_gender` char(1) DEFAULT NULL COMMENT '联系人性别',
  `lkm_phone` varchar(16) DEFAULT NULL COMMENT '联系人办公电话',
  `lkm_mobile` varchar(16) DEFAULT NULL COMMENT '联系人手机',
  `lkm_email` varchar(64) DEFAULT NULL COMMENT '联系人邮箱',
  `lkm_qq` varchar(16) DEFAULT NULL COMMENT '联系人qq',
  `lkm_position` varchar(16) DEFAULT NULL COMMENT '联系人职位',
  `lkm_memo` varchar(512) DEFAULT NULL COMMENT '联系人备注',
  PRIMARY KEY (`lkm_id`),
  KEY `FK_cst_linkman_lkm_cust_id` (`lkm_cust_id`),
  CONSTRAINT `FK_cst_linkman_lkm_cust_id` FOREIGN KEY (`lkm_cust_id`) REFERENCES `cst_customer` (`cust_id`) ON DELETE NO ACTION ON UPDATE NO ACTION
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;
```
* 对象中的表达
```
public class Customer {
	private Long cust_id;
	private String cust_name;
	private String cust_source;
	private String cust_industry;
	private String cust_level;
	private String cust_linkman;
	private String cust_phone;
	private String cust_mobile;
	//使用set集合,表达一对多关系
	private Set<LinkMan> linkMens = new HashSet<LinkMan>();
}

public class LinkMan {
	private Long lkm_id;
	private Character lkm_gender;
	private String lkm_name;
	private String lkm_phone;
	private String lkm_email;
	private String lkm_qq;
	private String lkm_mobile;
	private String lkm_memo;
	private String lkm_position;
	//表达多对一关系
	private Customer customer;
}
```
* orm元数据中的表达
```
        <!-- 集合,一对多关系,在配置文件中配置 -->
		<!-- 
			name属性:集合属性名
			column属性: 外键列名
			class属性: 与我关联的对象完整类名
		 -->
        <set name="linkMens" >
			<key column="lkm_cust_id" ></key>
			<one-to-many class="LinkMan" />
		</set>

        <!-- 多对一 -->
		<!-- 
			name属性:引用属性名
			column属性: 外键列名
			class属性: 与我关联的对象完整类名
		 -->
        <many-to-one name="customer" column="lkm_cust_id" class="Customer"  >
		</many-to-one>
```
2.操作
```
        //添加操作
		Customer c = new Customer();
		c.setCust_name("百度地图");
		
		LinkMan lm1 = new LinkMan();
		lm1.setLkm_name("罗志祥");
		
		LinkMan lm2 = new LinkMan();
		lm2.setLkm_name("梦幻");
		
		//表达一对多,客户下有多个联系人
		c.getLinkMens().add(lm1);
		c.getLinkMens().add(lm2);
		
		//表达对对对,联系人属于哪个客户
		lm1.setCustomer(c);
		lm2.setCustomer(c);
		
		session.save(c);
 		session.save(lm1);
 		session.save(lm2);
```
```
        //删除操作
		//1> 获得要操作的客户对象
		Customer c = session.get(Customer.class,1l);
		//2> 获得要移除的联系人
		LinkMan lm = session.get(LinkMan.class, 1l);
		//3> 将联系人从客户集合中移除
		c.getLinkMens().remove(lm);
		lm.setCustomer(null);
```
3.进阶操作
* 级联操作
```
         <!-- 
		 	级联操作:	cascade
		 		save-update: 级联保存更新
		 		delete:级联删除
		 		all:save-update+delete
		 	级联操作: 简化操作.目的就是为了少些两行代码.用save-update,不建议使用delete.
		  -->
		<set name="linkMens" inverse="true" cascade="save-update"  >
			<key column="lkm_cust_id" ></key>
			<one-to-many class="LinkMan" />
		</set>
```
使用save-update，添加操作的时候就只需要session.save(c);不需要session.save(lm1);session.save(lm2);
```
        session.save(c);
 		//session.save(lm1);
 		//session.save(lm2);
```
* 关系维护
```
        <!-- inverse属性: 配置关系是否维护. 
		  		true: customer不维护关系
		  		false(默认值): customer维护关系
		  	inverse属性: 性能优化.提高关系维护的性能.
		  	原则: 无论怎么放弃,总有一方必须要维护关系.
		  	一对多关系中: 一的一方放弃.也只能一的一方放弃.多的一方不能放弃.
		  -->
		<set name="linkMens" inverse="true" cascade="all"  >
			<key column="lkm_cust_id" ></key>
			<one-to-many class="LinkMan" />
		</set>
```
```
        //表达一对多,客户下有多个联系人. 
		// 如果客户放弃维护与联系人的关系. 维护关系的代码可以省略
		//c.getLinkMens().add(lm1);
		//c.getLinkMens().add(lm2);
```
###多对多 
