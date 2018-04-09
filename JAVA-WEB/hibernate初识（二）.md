###hibernate中的对象状态
1.对象的三种状态
* 瞬时状态
没有id,没有在session缓存中
* 持久化状态
有id,在session缓存中
* 游离|托管状态
有id,没有在session缓存中

2.三种状态的转换图
![image.png](https://upload-images.jianshu.io/upload_images/1956963-bcd00b049b1ab631.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.持久化: 持久化状态的对象,会在事务提交时,自动同步到数据库中.
我们使用hibernate的原则.就是将对象转换为持久化状态.

###hibernate的一级缓存
1.缓存:提高效率.hibernate中的一级缓存也是为了提高操作数据库的效率.
2.提高效率手段1:提高查询效率
![image.png](https://upload-images.jianshu.io/upload_images/1956963-a003666c362f7979.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3.提高效率手段2:减少不必要的修改语句发送
![image.png](https://upload-images.jianshu.io/upload_images/1956963-22842bbaa9b54a17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###hibernate中的事务
1.如何在hibernate中指定数据库的隔离级别
```
         <!-- 指定hibernate操作数据库时的隔离级别 
			#hibernate.connection.isolation 1|2|4|8		
			0001	1	读未提交
			0010	2	读已提交
			0100	4	可重复读
			1000	8	串行化
		 -->
		 <property name="hibernate.connection.isolation">4</property>
```
2.在项目中如何管理事务
* 业务开始之前打开事务,业务执行之后提交事务. 执行过程中出现异常.回滚事务.
* 在dao层操作数据库需要用到session对象.在service控制事务也是使用session对象完成. 我们要确保dao层和service层使用的使用同一个session对象
* 在hibernate中,确保使用同一个session的问题,hibernate已经帮我们解决了. 我们开发人员只需要调用sf.getCurrentSession()方法即可获得与当前线程绑定的session对象
* 注意1: 调用getCurrentSession方法必须配合主配置中的一段配置
```
<!-- 指定session与当前线程绑定 -->
		 <property name="hibernate.current_session_context_class">thread</property>
```
* 注意2:通过getCurrentSession方法获得的session对象.当事务提交时,session会自动关闭.不要手动调用close关闭.
###hibernate中的批量查询
######1.HQL查询-hibernate Query Language(多表查询,但不复杂时使用)
Hibernate独家查询语言,属于面向对象的查询语言
* 基本查询
```
        String hql = " from Customer "; // 查询所有Customer对象
		//2> 根据HQL语句创建查询对象
		Query query = session.createQuery(hql);
		//3> 根据查询对象获得查询结果
		List<Customer> list = query.list();	// 返回list结果
		//query.uniqueResult();//接收唯一的查询结果
		System.out.println(list);
```
* 条件查询
```
?号占位符

        //1> 书写HQL语句
		String hql = " from Customer where cust_id = ? "; // 查询所有Customer对象
		//2> 根据HQL语句创建查询对象
		Query query = session.createQuery(hql);
		//设置参数
		//query.setLong(0, 1l);
		query.setParameter(0, 1l);
		//3> 根据查询对象获得查询结果
		Customer c = (Customer) query.uniqueResult();
		System.out.println(c);
```
```
命名占位符

        //1> 书写HQL语句
		String hql = " from Customer where cust_id = :cust_id "; // 查询所有Customer对象
		//2> 根据HQL语句创建查询对象
		Query query = session.createQuery(hql);
		//设置参数
		query.setParameter("cust_id", 1l);
		//3> 根据查询对象获得查询结果
		Customer c = (Customer) query.uniqueResult();
		System.out.println(c);
```
* 分页查询
```
        //1> 书写HQL语句
		String hql = " from Customer  "; // 查询所有Customer对象
		//2> 根据HQL语句创建查询对象
		Query query = session.createQuery(hql);
		//设置分页信息 limit ?,?
		query.setFirstResult(1);
		query.setMaxResults(1);
		//3> 根据查询对象获得查询结果
		List<Customer> list =  query.list();
		System.out.println(list);
```
######2.Criteria查询(单表条件查询)
Hibernate自创的无语句面向对象查询
* 基本查询
```
        //查询所有的Customer对象
		Criteria criteria = session.createCriteria(Customer.class);
		List<Customer> list = criteria.list();
```
* 条件查询
```
       //创建criteria查询对象
		Criteria criteria = session.createCriteria(Customer.class);
		//添加查询参数 => 查询cust_id为1的Customer对象
		criteria.add(Restrictions.eq("cust_id", 1l));
		//执行查询获得结果
		Customer c = (Customer) criteria.uniqueResult();
```
* 分页查询
```
        //创建criteria查询对象
		Criteria criteria = session.createCriteria(Customer.class);
		//设置分页信息 limit ?,?
		criteria.setFirstResult(1);
		criteria.setMaxResults(2);
		//执行查询
		List<Customer> list = criteria.list();
```
* 设置查询总记录数
```
        //创建criteria查询对象
		Criteria criteria = session.createCriteria(Customer.class);
		//设置查询的聚合函数 => 总行数
		criteria.setProjection(Projections.rowCount());
		//执行查询
		Long count = (Long) criteria.uniqueResult();
```
######3.原生SQL查询(复杂的业务查询)
* 基本查询
```
返回数组List

        //1 书写sql语句
		String sql = "select * from cst_customer";
		//2 创建sql查询对象
		SQLQuery query = session.createSQLQuery(sql);
		//3 调用方法查询结果
		List<Object[]> list = query.list();
		//query.uniqueResult();
		for(Object[] objs : list){
			System.out.println(Arrays.toString(objs));
		}
```
```
返回对象List

        //1 书写sql语句
		String sql = "select * from cst_customer";
		//2 创建sql查询对象
		SQLQuery query = session.createSQLQuery(sql);
		//指定将结果集封装到哪个对象中
		query.addEntity(Customer.class);
		//3 调用方法查询结果
		List<Customer> list = query.list();
```
* 条件查询
```
        //1 书写sql语句
		String sql = "select * from cst_customer where cust_id = ? ";
		//2 创建sql查询对象
		SQLQuery query = session.createSQLQuery(sql);
		query.setParameter(0, 1l);
		//指定将结果集封装到哪个对象中
		query.addEntity(Customer.class);
		//3 调用方法查询结果
		List<Customer> list = query.list();
```
* 分页查询
```
        //1 书写sql语句
		String sql = "select * from cst_customer limit ?,? ";
		//2 创建sql查询对象
		SQLQuery query = session.createSQLQuery(sql);
		query.setParameter(0, 0);
		query.setParameter(1, 1);
		//指定将结果集封装到哪个对象中
		query.addEntity(Customer.class);
		//3 调用方法查询结果
		List<Customer> list = query.list();
```
