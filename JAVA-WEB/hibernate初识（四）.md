###查询-HQL语法
###查询-Criteria语法
######1.语法
* 基本语法
```
Criteria c = session.createCriteria(Customer.class);
List<Customer> list = c.list();
```
* 条件语法
```
Criteria c = session.createCriteria(Customer.class);
//c.add(Restrictions.idEq(2l));
c.add(Restrictions.eq("cust_id",2l));
List<Customer> list = c.list();
```
* 分页语法
```
Criteria c = session.createCriteria(Customer.class);
//limit ?,? 
c.setFirstResult(0);
c.setMaxResults(2);
List<Customer> list = c.list();
```
* 排序语法
```
Criteria c = session.createCriteria(Customer.class);
c.addOrder(Order.asc("cust_id"));
//c.addOrder(Order.desc("cust_id"));
List<Customer> list = c.list();
```
* 统计语法
```
Criteria c = session.createCriteria(Customer.class);
//设置查询目标
c.setProjection(Projections.rowCount());
List list = c.list();
```
######2.离线查询
使用DetachedCriteria来构造查询条件就不必为了查询条件的变化而去频繁改动查询语句
```
    @Test
	public void function(){
		//Service/web层
		DetachedCriteria dc  = DetachedCriteria.forClass(Customer.class);
		dc.add(Restrictions.idEq(6l));//拼装条件(全部与普通Criteria一致)
		//----------------------------------------------------
		Session session = HibernateUtils.openSession();
		Transaction tx = session.beginTransaction();
		//----------------------------------------------------
		Criteria c = dc.getExecutableCriteria(session);
		List list = c.list();
		System.out.println(list);
		//----------------------------------------------------
		tx.commit();
		session.close();
	}
```
```
离线查询示例：分页查询
public void pageQuery(PageBean pageBean) {
		int currentPage = pageBean.getCurrentPage();
		int pageSize = pageBean.getPageSize();
		DetachedCriteria detachedCriteria = pageBean.getDetachedCriteria();
		
		//查询total---总数据量
		detachedCriteria.setProjection(Projections.rowCount());//指定hibernate框架发出sql的形式----》select count(*) from bc_staff;
		List<Long> countList = (List<Long>) this.getHibernateTemplate().findByCriteria(detachedCriteria);
		Long count = countList.get(0);
		pageBean.setTotal(count.intValue());
		
		//查询rows---当前页需要展示的数据集合
		detachedCriteria.setProjection(null);//指定hibernate框架发出sql的形式----》select * from bc_staff;
  		//指定hibernate框架封装对象的方式,使查询到的list与数据库字段对应
		detachedCriteria.setResultTransformer(DetachedCriteria.ROOT_ENTITY);
		int firstResult = (currentPage - 1) * pageSize;
		int maxResults = pageSize;
		List rows = this.getHibernateTemplate().findByCriteria(detachedCriteria, firstResult, maxResults);
		pageBean.setRows(rows);
	}
```
###查询优化
######1.类级别查询
* get方法:没有任何策略.调用即立即查询数据库加载数据.
* load方法:应用类级别的加载策略
```
<class name="Customer" table="cst_customer" lazy="true" >
1.lazy(默认值):true, 查询类时,会返回代理对象.在使用属性时,才会根据关联的session查询数据库.加载数据，不使用不加载数据。
2.lazy:false. load方法会与get方法没有任何区别.调用时即加载数据.
3.结论:为了提高效率.建议使用延迟加载(懒加载)
4.注意:使用懒加载时要确保,调用属性加载数据时,session还是打开的.不然会抛出异常
```
######2.关联级别查询
* 集合策略
```
    <!-- 
		lazy属性: 决定是否延迟加载
			true(默认值): 延迟加载,懒加载
			false: 立即加载
			extra: 极其懒惰
		fetch属性: 决定加载策略.使用什么类型的sql语句加载集合数据
			select(默认值): 单表查询加载
			join: 使用多表查询加载集合
			subselect:使用子查询加载集合
	 -->
```
* 关联属性策略
```
        <!-- 
		fetch 决定加载的sql语句
			select: 使用单表查询
			join : 多表查询
		lazy  决定加载时机
			false: 立即加载
			proxy: 由customer的类级别加载策略决定.
		 -->
		<many-to-one name="customer" column="lkm_cust_id" class="Customer" fetch="join" lazy="proxy"  >
		</many-to-one>
```
* 结论:为了提高效率.fetch的选择上应选择select. lazy的取值应选择 true. 全部使用默认值.
* no-session问题解决: 扩大session的作用范围.
######3.批量抓取
使用批量抓取可以减少sql语句的发送，提高检索效率。
```
      <!-- batch-size: 抓取集合的数量为10.
	 		抓取客户的集合时,一次抓取几个客户的联系人集合.
	  -->
      <set name="linkMens" batch-size="10"  >
			<key column="lkm_cust_id" ></key>
			<one-to-many class="LinkMan" />
	  </set>
```
