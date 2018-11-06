###什么是[Solr](http://lucene.apache.org/solr/)
Solr是基于Lucene的全文搜索服务器。Solr提供了比Lucene更为丰富的查询语言，同时实现了可配置、可扩展，并对索引、搜索性能进行了优化。 

###使用[Solr](http://lucene.apache.org/solr/)的原因
单独使用Lucene实现站内搜索需要开发的工作量较大，主要表现在：索引维护、索引性能优化、搜索性能优化等，因此不建议采用。
基于Solr实现站内搜索扩展性较好并且可以减少程序员的工作量，因为Solr提供了较为完备的搜索引擎解决方案，因此在门户、论坛等系统中常用此方案。

###[Solr使用指南](https://wiki.apache.org/solr/FrontPage)
###[Solr](http://lucene.apache.org/solr/)的安装及配置
######1. 官网下载安装包并解压，目录结构如下
![image.png](https://upload-images.jianshu.io/upload_images/1956963-59e31d399df50d1b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

######2. Solr整合tomcat
- 1 创建一个Solr home目录
![image.png](https://upload-images.jianshu.io/upload_images/1956963-0066e011228aa079.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 2 把solr的war包复制到tomcat 的webapp目录下
- 3 把solr.war解压并删除
- 4 把\solr-4.10.3\example\lib\ext目录下的所有的jar包添加到solr工程中
- 5 配置solrHome和solrCore
1 . 创建一个solrhome1。\solr-4.10.3\example\solr目录就是一个标准的solrhome。
2 . 在solrhome下有一个文件夹叫做collection1这就是一个solrcore。就是一个solr的实例。一个solrcore相当于mysql中一个数据库。Solrcore之间是相互隔离。
- 6 修改web.xml，告诉solr服务器solrHome的位置。
```
  <!-- People who want to hardcode their "Solr Home" directly into the
       WAR File can set the JNDI property here...
   -->
    <env-entry>
       <env-entry-name>solr/home</env-entry-name>
       <env-entry-value>/put/your/solr/home/here</env-entry-value>
       <env-entry-type>java.lang.String</env-entry-type>
    </env-entry>
```
- 7 启动tomcat,访问http://localhost:8080/solr/

######3. 配置中文分析器
* Schema.xml分析
Solr数据表配置文件，它定义了加入索引的数据的数据类型。
**1 . FieldType域类型定义**
**2 . Field定义**
在fields结点内定义具体的Field，filed定义包括name,type（为之前定义过的各种FieldType）,indexed（是否被索引）,stored（是否被储存），multiValued（是否存储多个值）等属性。
**3 . uniqueKey**
Solr中默认定义唯一主键key为id域: <uniqueKey>id</uniqueKey>
Solr在删除、更新索引时使用id域进行判断，也可以自定义唯一主键。注意在创建索引时必须指定唯一约束。
**4 . copyField复制域**
copyField复制域，可以将多个Field复制到一个Field中，以便进行统一的检索
**5 . dynamicField**
动态字段就是不用指定具体的名称，只要定义字段名称的规则，例如定义一个 dynamicField，name 为*_i，定义它的type为text，那么在使用这个字段的时候，任何以_i结尾的字段都被认为是符合这个定义的。

* 安装中文分词器
使用IKAnalyzer中文分析器
1 . 把IKAnalyzer2012FF_u1.jar添加到solr/WEB-INF/lib目录下。
2 . 复制IKAnalyzer的配置文件和自定义词典和停用词词典到solr的classpath下。
3 . 在schema.xml中添加一个自定义的fieldType，使用中文分析器。并定义field，指定field的type属性为text_ik。
4 . 重启tomcat测试
![image.png](https://upload-images.jianshu.io/upload_images/1956963-907f55a572b429c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###使用Solr的后台管理索引库
######1. 添加/更新文档
######2. 批量导入数据
使用dataimport插件批量导入数据。
* 1 . 把dataimport插件依赖的jar包以及mysql的数据库驱动添加到solrcore（collection1\lib）中。
solr-dataimporthandler-4.10.3.jar
solr-dataimporthandler-extras-4.10.3.jar
mysql-connector-java-5.1.7-bin.jar
* 2 . 配置solrconfig.xml文件，添加一个requestHandler。
```
  <requestHandler name="/dataimport" 
	class="org.apache.solr.handler.dataimport.DataImportHandler">
    <lst name="defaults">
      <str name="config">data-config.xml</str>
    </lst>
  </requestHandler>
```
* 3 . 创建一个data-config.xml，保存到collection1\conf\目录下
```
<?xml version="1.0" encoding="UTF-8" ?>  
<dataConfig>   
<dataSource type="JdbcDataSource"   
		  driver="com.mysql.jdbc.Driver"   
		  url="jdbc:mysql://localhost:3306/lucene"   
		  user="root"   
		  password="root"/>   
	<document>   
		<entity name="product" query="SELECT pid,name,catalog_name,price,description,picture FROM products ">
			 <field column="pid" name="id"/> 
			 <field column="name" name="product_name"/> 
			 <field column="catalog_name" name="product_catalog_name"/> 
			 <field column="price" name="product_price"/> 
			 <field column="description" name="product_description"/> 
			 <field column="picture" name="product_picture"/> 
		</entity>   
	</document>   
</dataConfig>
```
* 4 . 设置业务系统Field
如果不使用Solr提供的Field可以针对具体的业务需要自定义一套Field，如下是商品信息Field：
```
   <!--product-->
   <field name="product_name" type="text_ik" indexed="true" stored="true"/>
   <field name="product_price"  type="float" indexed="true" stored="true"/>
   <field name="product_description" type="text_ik" indexed="true" stored="false" />
   <field name="product_picture" type="string" indexed="false" stored="true" />
   <field name="product_catalog_name" type="string" indexed="true" stored="true" />
   <field name="product_keywords" type="text_ik" indexed="true" stored="false" multiValued="true"/>
   <copyField source="product_name" dest="product_keywords"/>
   <copyField source="product_description" dest="product_keywords"/>
```
* 5 . 在数据库中准备测试数据，重启tomcat，在后台管理页面点击“execute”按钮导入数据
######3. 删除文档
* 1 . 删除制定ID的索引 
```
<delete>
	<id>8</id>
</delete>
<commit/>
```
* 2 . 删除查询到的索引数据 
```
<delete>
	<query>product_catalog_name:幽默杂货</query>
</delete>
<commit/>
```
* 3 . 删除所有索引数据
```
<delete>
	<query>*:*</query>
</delete>
<commit/>
```
######4. 修改文档
######5. 查询文档
* q - 查询字符串，必须的，如果查询所有使用*:*。
* fq - （filter query）过滤查询，作用：在q查询符合结果中同时是fq查询符合的
* sort - 排序，
* start - rows 分页显示使用
* fl - 指定返回那些字段内容，用逗号或空格分隔多个。
* df - 指定一个搜索Field
* wt - 指定输出格式
* hl - 是否高亮 ,设置高亮Field，设置格式前缀和后缀。
![image.png](https://upload-images.jianshu.io/upload_images/1956963-54ea19d99c9ab478.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###使用SolrJ管理索引库
* **什么是SolrJ**
SolrJ是操作Solr的Java客户端,它提供了增加、修改、删除、查询Solr索引的Java接口。
* **依赖的jar包**
![image.png](https://upload-images.jianshu.io/upload_images/1956963-a5c2bb0be4d48d04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-35352d4bcfc1ee29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
主要步骤：
和Solr服务器建立连接。使用HttpSolrServer对象建立连接。
创建一个SolrInputDocument对象，然后添加域。
将SolrInputDocument添加到索引库。
提交。

**1. 添加/更新文档**
```
	@Test
	public void addDocument() throws Exception {
		//和solr服务器创建连接
		//参数：solr服务器的地址
		SolrServer solrServer = new HttpSolrServer("http://localhost:8080/solr");
		//创建一个文档对象
		SolrInputDocument document = new SolrInputDocument();
		//向文档中添加域
		//第一个参数：域的名称，域的名称必须是在schema.xml中定义的
		//第二个参数：域的值
		document.addField("id", "c0001");
		document.addField("title_ik", "使用solrJ添加的文档");
		document.addField("content_ik", "文档的内容");
		document.addField("product_name", "商品名称");
		//把document对象添加到索引库中
		solrServer.add(document);
		//提交修改
		solrServer.commit();
	}
```
**2. 删除文档**
* 根据id删除
```
	//删除文档，根据id删除
	@Test
	public void deleteDocumentByid() throws Exception {
		//创建连接
		SolrServer solrServer = new HttpSolrServer("http://localhost:8080/solr");
		//根据id删除文档
		solrServer.deleteById("c0001");
		//提交修改
		solrServer.commit();
	}
```
* 根据查询删除
查询语法完全支持Lucene的查询语法。
```
	//根据查询条件删除文档
	@Test
	public void deleteDocumentByQuery() throws Exception {
		//创建连接
		SolrServer solrServer = new HttpSolrServer("http://localhost:8080/solr");
		//根据查询条件删除文档
		solrServer.deleteByQuery("*:*");
		//提交修改
		solrServer.commit();
	}
```

**3. 修改文档**
在SolrJ中修改没有对应的update方法，只有add方法，只需要添加一条新的文档，和被修改的文档id一致就，可以修改了。本质上就是先删除后添加。

**4. 查询文档**
* 简单查询
```
	//查询索引
	@Test
	public void queryIndex() throws Exception {
		//创建连接
		SolrServer solrServer = new HttpSolrServer("http://localhost:8080/solr");
		//创建一个query对象
		SolrQuery query = new SolrQuery();
		//设置查询条件
		query.setQuery("*:*");
		//执行查询
		QueryResponse queryResponse = solrServer.query(query);
		//取查询结果
		SolrDocumentList solrDocumentList = queryResponse.getResults();
		//共查询到商品数量
		System.out.println("共查询到商品数量:" + solrDocumentList.getNumFound());
		//遍历查询的结果
		for (SolrDocument solrDocument : solrDocumentList) {
			System.out.println(solrDocument.get("id"));
			System.out.println(solrDocument.get("product_name"));
			System.out.println(solrDocument.get("product_price"));
			System.out.println(solrDocument.get("product_catalog_name"));
			System.out.println(solrDocument.get("product_picture"));
		}
	}
```
* 复杂查询
其中包含查询、过滤、分页、排序、高亮显示等处理
```
	//复杂查询索引
	@Test
	public void queryIndex2() throws Exception {
		//创建连接
		SolrServer solrServer = new HttpSolrServer("http://localhost:8080/solr");
		//创建一个query对象
		SolrQuery query = new SolrQuery();
		//设置查询条件
		query.setQuery("钻石");
		//过滤条件
		query.setFilterQueries("product_catalog_name:幽默杂货");
		//排序条件
		query.setSort("product_price", ORDER.asc);
		//分页处理
		query.setStart(0);
		query.setRows(10);
		//结果中域的列表
		query.setFields("id","product_name","product_price","product_catalog_name","product_picture");
		//设置默认搜索域
		query.set("df", "product_keywords");
		//高亮显示
		query.setHighlight(true);
		//高亮显示的域
		query.addHighlightField("product_name");
		//高亮显示的前缀
		query.setHighlightSimplePre("<em>");
		//高亮显示的后缀
		query.setHighlightSimplePost("</em>");
		//执行查询
		QueryResponse queryResponse = solrServer.query(query);
		//取查询结果
		SolrDocumentList solrDocumentList = queryResponse.getResults();
		//共查询到商品数量
		System.out.println("共查询到商品数量:" + solrDocumentList.getNumFound());
		//遍历查询的结果
		for (SolrDocument solrDocument : solrDocumentList) {
			System.out.println(solrDocument.get("id"));
			//取高亮显示
			String productName = "";
			Map<String, Map<String, List<String>>> highlighting = queryResponse.getHighlighting();
			List<String> list = highlighting.get(solrDocument.get("id")).get("product_name");
			//判断是否有高亮内容
			if (null != list) {
				productName = list.get(0);
			} else {
				productName = (String) solrDocument.get("product_name");
			}
			System.out.println(productName);
			System.out.println(solrDocument.get("product_price"));
			System.out.println(solrDocument.get("product_catalog_name"));
			System.out.println(solrDocument.get("product_picture"));
		}
	}
```

###电商搜索案例实现
springmvc.xml
```
        <!-- 配置SolrJ -->
        <bean id="solrServer" class="org.apache.solr.client.solrj.impl.HttpSolrServer">
        	<constructor-arg value="http://localhost:8080/solr/collection1"/>
        </bean>
```

###资料
[Solr环境](https://download.csdn.net/download/w_1231/10313104)
