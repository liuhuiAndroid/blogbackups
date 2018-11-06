###一、Linux上Solr服务搭建
* 搭建步骤：
```
1.上传solr-4.10.3.tar.gz到Linux上
2.解压solr到/usr/local/solr目录
mkdir /usr/local/solr
tar -zxvf solr-4.10.3.tar.gz -C /usr/local/solr
3.将solr的war包拷贝到tomcat
cp solr-4.10.3/dist/solr-4.10.3.war apache-tomcat-7.0.47/webapps/
mv solr-4.10.3.war solr.war
4.启动tomcat来解压solr.war，然后关闭tomcat，删除solr.war
5.把solr-4.10.3/example/lib/ext目录下的所有的jar包，添加到solr工程中
cp solr-4.10.3/example/lib/ext/* apache-tomcat-7.0.47/webapps/solr/WEB-INF/lib/
6.创建一个solrhome。/example/solr目录就是一个solrhome。复制此目录
cp solr-4.10.3/example/solr ./solrhome -r
7.关联solr及solrhome。需要修改solr工程的web.xml文件。
 vi /usr/local/solr/apache-tomcat-7.0.47/webapps/solr/WEB-INF/web.xml
    <env-entry>
       <env-entry-name>solr/home</env-entry-name>
       <env-entry-value>/usr/local/solr/solrhome</env-entry-value>
       <env-entry-type>java.lang.String</env-entry-type>
    </env-entry>
8.启动Tomcat，访问http://192.168.10.179:8080/solr/
```
* 配置业务域
```
1.把中文分析器添加到工程中。
把IKAnalyzer2012FF_u1.jar添加到solr工程的lib目录下
cp IKAnalyzer2012FF_u1.jar /usr/local/solr/apache-tomcat-7.0.47/webapps/solr/WEB-INF/lib/
把扩展词典、配置文件放到solr工程的WEB-INF/classes目录下。
mkdir /usr/local/solr/apache-tomcat-7.0.47/webapps/solr/WEB-INF/classes
cp mydict.dic ext_stopword.dic IKAnalyzer.cfg.xml /usr/local/solr/apache-tomcat-7.0.47/webapps/solr/WEB-INF/classes
2.配置一个FieldType，制定使用IKAnalyzer
vi solrhome/collection1/conf/schema.xml 
<fieldType name="text_ik" class="solr.TextField">
  <analyzer class="org.wltea.analyzer.lucene.IKAnalyzer"/>
</fieldType>
3.配置业务域，type制定使用自定义的FieldType。设置业务系统Field
<field name="item_title" type="text_ik" indexed="true" stored="true"/>
<field name="item_sell_point" type="text_ik" indexed="true" stored="true"/>
<field name="item_price"  type="long" indexed="true" stored="true"/>
<field name="item_image" type="string" indexed="false" stored="true" />
<field name="item_category_name" type="string" indexed="true" stored="true" />
<field name="item_keywords" type="text_ik" indexed="true" stored="false" multiValued="true"/>
<copyField source="item_title" dest="item_keywords"/>
<copyField source="item_sell_point" dest="item_keywords"/>
<copyField source="item_category_name" dest="item_keywords"/>
4.重启Tomcat
```
###二、[SolrCloud](http://lucene.apache.org/solr/guide/6_6/solrcloud.html)介绍
>Apache Solr includes the ability to set up a cluster of Solr servers that combines fault tolerance and high availability. Called SolrCloud, these capabilities provide distributed indexing and search capabilities.
SolrCloud is flexible distributed search and indexing, without a master node to allocate nodes, shards and replicas. Instead, Solr uses ZooKeeper to manage these locations, depending on configuration files and schemas. Queries and updates can be sent to any server. Solr will use the information in the ZooKeeper database to figure out which servers need to handle the request.

**Apache Solr包含了建立一个包含容错和高可用性的Solr服务器集群的能力，这些功能被称为SolrCloud。提供了分布式索引和搜索功能。
SolrCloud是灵活的分布式搜索和索引，没有一个主节点来分配节点、碎片和副本。相反，Solr使用ZooKeeper来管理这些位置，这取决于配置文件和模式。查询和更新可以发送到任何服务器。Solr将使用ZooKeeper数据库中的信息来确定哪些服务器需要处理请求。**

###三、Solr集群的系统架构
![image.png](https://upload-images.jianshu.io/upload_images/1956963-abb34eeb71182004.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###四、Solr集群搭建
* Zookeeper集群搭建
```
1.把zookeeper-3.4.6.tar.gz上传到服务器。
2.解压缩zookeeper到/usr/local/solr/solrcloud/，并复制三份。
tar -zxvf /root/zookeeper-3.4.6.tar.gz -C ./
mv zookeeper-3.4.6/ zookeeper01
cp -r zookeeper01/ zookeeper02
cp -r zookeeper01/ zookeeper03
3.在每个zookeeper目录下创建一个data目录。在每个data目录下创建一个myid文件，内容是每个实例的id。
mkdir zookeeper01/data
cd zookeeper01/data/  
echo 1 >> myid
echo 2 >> zookeeper02/data/myid
echo 3 >> zookeeper03/data/myid
4.修改配置文件。把conf目录下的zoo_sample.cfg文件改名为zoo.cfg
mv zookeeper01/conf/zoo_sample.cfg zookeeper01/conf/zoo.cfg

dataDir=/usr/local/solr/solrcloud/zookeeper01/data
clientPort=2181
server.1=192.168.10.179:2881:3881
server.2=192.168.10.179:2882:3882
server.3=192.168.10.179:2883:3883
5.启动每个zookeeper实例
vi start-zookeeper-all.sh
cd zookeeper01/bin
./zkServer.sh start
cd ../..
cd zookeeper02/bin
./zkServer.sh start
cd ../..
cd zookeeper03/bin
./zkServer.sh start
cd ../..

chmod u+x start-zookeeper-all.sh 
./start-zookeeper-all.sh
6.查看zookeeper的状态：
zookeeper01/bin/zkServer.sh status
```

* Solr集群的搭建
```
1.创建四个tomcat实例。每个tomcat运行在不同的端口。8180、8280、8380、8480
 tar -zxvf /root/apache-tomcat-7.0.47.tar.gz -C ./
mv apache-tomcat-7.0.47/ tomcat01
cp -r tomcat01 tomcat02
cp -r tomcat01 tomcat03
cp -r tomcat01 tomcat04
vi tomcat01/conf/server.xml 
2.部署solr的war包。把单机版的solr工程复制到集群中的tomcat中。
3.为每个solr实例创建一个对应的solrhome。使用单机版的solrhome复制四份。
4.需要修改solr的web.xml文件。把solrhome关联起来。
5.配置solrCloud相关的配置。每个solrhome下都有一个solr.xml，把其中的ip及端口号配置好。
  <solrcloud>
    <str name="host">192.168.10.179</str>
    <int name="hostPort">8180</int>
    <str name="hostContext">${hostContext:solr}</str>
    <int name="zkClientTimeout">${zkClientTimeout:30000}</int>
    <bool name="genericCoreNodeNames">${genericCoreNodeNames:true}</bool>
  </solrcloud>
6.让zookeeper统一管理配置文件。需要把solrhome/collection1/conf目录上传到zookeeper。上传任意solrhome中的配置文件即可。
cd solr-4.10.3/example/scripts/cloud-scripts/
./zkcli.sh -zkhost 192.168.25.154:2181,192.168.25.154:2182,192.168.25.154:2183 -cmd upconfig -confdir /usr/local/solr-cloud/solrhome01/collection1/conf -confname myconf

可能会报错 Could not find or load main class org.apache.solr.cloud.ZkCLI
解决报错：cd solr-4.10.3/example/
java -jar start.jar

可以在zookeeper的bin目录下使用zkCli.sh命令查看zookeeper上的配置文件：
 ./zkCli.sh 
 ls /configs/myconf

使用以下命令连接指定的zookeeper服务：
./zkCli.sh -server 192.168.10.179:2183           quit退出
7.修改tomcat/bin目录下的catalina.sh 文件，关联solr和zookeeper。
vi tomcat01/bin/catalina.sh
把此配置添加到配置文件中：
JAVA_OPTS="-DzkHost=192.168.10.179:2181,192.168.10.179:2182,192.168.10.179:2183"
8.启动每个tomcat实例。要保证zookeeper集群是启动状态。
vi start-tomcat-all.sh
tomcat01/bin/startup.sh
tomcat02/bin/startup.sh
tomcat03/bin/startup.sh
tomcat04/bin/startup.sh 
chmod u+x start-tomcat-all.sh 
./start-tomcat-all.sh
9.访问集群
```
![image.png](https://upload-images.jianshu.io/upload_images/1956963-79293a9d6ebbb7aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
10.创建新的Collection进行分片处理。
http://192.168.10.179:8180/solr/admin/collections?action=CREATE&name=collection2&numShards=2&replicationFactor=2
11.删除不用的Collection。
http://192.168.10.179:8180/solr/admin/collections?action=DELETE&name=collection1
```
![image.png](https://upload-images.jianshu.io/upload_images/1956963-a620ea8632e7451c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###五、使用SolrJ管理Solr集群
**1. 添加/更新文档**
```
	@Test
	public void testAddDocument() throws Exception {
		//创建一个集群的连接，应该使用CloudSolrServer创建。
		CloudSolrServer solrServer = new CloudSolrServer("192.168.10.179:2181,192.168.10.179:2182,192.168.10.179:2183");
		//zkHost：zookeeper的地址列表
		//必须设置一个defaultCollection属性。
		solrServer.setDefaultCollection("collection2");
		//创建一个文档对象
		SolrInputDocument document = new SolrInputDocument();
		//向文档中添加域
		document.setField("id", "solrcloud01");
		document.setField("item_title", "测试");
		document.setField("item_price", 123);
		//把文件写入索引库
		solrServer.add(document);
		//提交
		solrServer.commit();
	}
```
**2. 查询文档**
```
	@Test
	public void testQueryDocument() throws Exception {
		//创建一个CloudSolrServer对象
		CloudSolrServer cloudSolrServer = new CloudSolrServer("192.168.10.179:2181,192.168.10.179:2182,192.168.10.179:2183");
		//设置默认的Collection
		cloudSolrServer.setDefaultCollection("collection2");
		//创建一个查询对象
		SolrQuery query = new SolrQuery();
		//设置查询条件
		query.setQuery("*:*");
		//执行查询
		QueryResponse queryResponse = cloudSolrServer.query(query);
		//取查询结果
		SolrDocumentList solrDocumentList = queryResponse.getResults();
		System.out.println("总记录数：" + solrDocumentList.getNumFound());
		//打印
		for (SolrDocument solrDocument : solrDocumentList) {
			System.out.println(solrDocument.get("id"));
			System.out.println(solrDocument.get("title"));
			System.out.println(solrDocument.get("item_title"));
			System.out.println(solrDocument.get("item_price"));
		}
	}
```
###六、电商搜索案例实现
```
	<!-- 集群版solrJ -->
	<bean id="cloudSolrServer" class="org.apache.solr.client.solrj.impl.CloudSolrServer">
		<constructor-arg index="0" value="192.168.10.179:2181,192.168.10.179:2182,192.168.10.179:2183"></constructor-arg>
		<property name="defaultCollection" value="collection2"></property>
	</bean>
```
