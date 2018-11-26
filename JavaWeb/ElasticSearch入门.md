## 第1章 课程介绍
#### 1-1 课程介绍
**什么是ElasticSearch？**
- 基于Apache Lucene构建的开源搜索引擎
- 采用Java编写，提供简单易用的RESTFul API
- 轻松的横向扩展，可支持PB级的结构化或非结构化数据处理

**可用应用场景**
- 海量数据分析引擎
- 站内搜索引擎
- 数据仓库

**一线公司实际应用场景**
- 英国卫报 —— 实时分析公众对文章的回应
- 维基百科、Github —— 站内实时搜素
- 百度 —— 实时日志监控平台

## 第2章 安装
#### 2-2 单实例安装
[**ElasticSearch 官网**](https://www.elastic.co/cn/)
```
官方网站下载地址复制链接，ps：老师的是5.5.2版本
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.5.0.tar.gz
tar -zxvf elasticsearch-6.5.0.tar.gz
cd elasticsearch-6.5.0
java -version # 检查jdk版本1.8以上
sh ./bin/elasticsearch # 启动
started 日志输出代表正常启动
127.0.0.1:9200 可以查看es输出
```
```
CentOS 安装问题见手记
vi bin/elasticsearch 
由于服务器内存太小添加配置:ES_JAVA_OPTS="-Xms1g -Xmx1g"
不允许使用root用户，所以新建用户运行es
adduser seckill
passwd seckill
chmod -R seckill:seckill elasticsearch-6.5.0
su - sekill 
```
```
外网访问es：
在目录config/elasticsearch.yml文件中添加如下：
network.host: 0.0.0.0
http.port: 9200
```

#### 2-3 插件安装
实用插件Head插件：提供UI页面，基本信息查看等功能
**[elasticsearch-head github 地址](https://github.com/mobz/elasticsearch-head)**
```
Github下载地址复制链接
wget https://github.com/mobz/elasticsearch-head/archive/master.zip
unzip master.zip
cd elasticsearch-head-master
node -v # 检查node环境6.0以上
npm install
npm run start # 可以看到服务在9100启动了
localhost:9100 可以查看web页面

解决插件和ES跨域问题：
cd elasticsearch-6.5.0
vi config/elasticsearch.yml
在最后一行添加：
http.cors.enabled: true
http.cors.allow-origin: "*"
./bin/elasticsearch -d # 后台启动ES
cd elasticsearch-head-master
npm run start # 启动插件
localhost:9100 可以查看web页面 green状态
```
```
node安装:
wget https://nodejs.org/dist/v10.13.0/node-v10.13.0-linux-x64.tar.xz # 下载地址见官网：https://nodejs.org/en/
xz -d node-v10.13.0-linux-x64.tar.xz 
tar -xvf node-v10.13.0-linux-x64.tar 
ln -s /usr/node-v10.13.0-linux-x64/bin/node /usr/local/bin/node
ln -s /usr/node-v10.13.0-linux-x64/bin/npm /usr/local/bin/npm
node -v # 查看node版本，安装成功
```

#### 2-4 分布式安装
**一个master、两个slave**
配置之前的为master节点：
```
cd elasticsearch-6.5.0
vi config/elasticsearch.yml
在最后一行添加：
cluster.name: lh
node.name: master
node.master: true
network.host: 127.0.0.1
ps -ef | grep `pwd` # 找到之前启动的es
kill 5531 # 停止es
./bin/elasticsearch -d # 重新启动ES
localhost:9100 查看集群是否正常启动
127.0.0.1:9200 检查集群名称
```
slave 节点
```
mkdir es_slave
cp elasticsearch-6.5.0.tar.gz es_slave
cd es_slave
tar -zxvf elasticsearch-6.5.0.tar.gz
cp -r elasticsearch-6.5.0 es_slave1
cp -r elasticsearch-6.5.0 es_slave2

cd es_slave1
vi config/elasticsearch.yml
在最后一行添加：
cluster.name: lh # 注意要和master保持一致
node.name: slave1
network.host: 127.0.0.1
http.port: 8200
discovery.zen.ping.unicast.hosts: ["127.0.0.1"] # 找到master
./bin/elasticsearch -d # 启动

localhost:9100 刷新可以查看slave1
同样操作slave2
```

## 第3章 基础概念
#### 3-1 基础概念
**集群和节点**
**索引**：含有相同属性的文档集合
**类型**：索引可以定义一个或多个类型，文档必须属于一个类型
**文档**：文档是可以被索引的基本数据单位
索引、类型和文档相当于数据库、表和数据
**分片**：每个索引都有多个分片，每个分片是一个Lucene索引
**备份**：拷贝一份分片就完成了分片的备份

## 第4章 基本用法
#### 4-1 索引创建
**API基本格式**：http://<ip>:<port>/<索引>/<类型>/<文档id>
**常用HTTP动词**：GET/PUT/POST/DELETE
```
可以使用head插件新建索引
访问：http://118.126.111.144:9100/
索引 -> 新建索引 -> 索引名称：book 分片数：5 副本数：1
ps:索引名称需要小写且不能有下划线
```
创建索引
- 非结构化创建
- 结构化创建
使用head插件结构化索引创建：
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-5ad873df7c20222e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
使用postman创建：
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-1c4f6392d2fb808a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
{
    "settings": {
        "number_of_shards": 3,
        "number_of_replicas": 1
    },
    "mappings": {
        "man": {
            "properties": {
                "name":{
                    "type": "text"
                },
                "country":{
                    "type": "keyword"
                },
                "age":{
                    "type": "integer"
                },
                "date":{
                    "type": "date",
                    "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
                }
            }
        }
    }
}
```

#### 4-2 插入
- 指定文档id插入
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-99c1676aa90199d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 自动产生文档id插入
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-f5b7d9eda159d351.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4-3 修改
- 直接修改文档
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-85f7204d42b9c272.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 脚本修改文档
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-f3490403471396f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-312d2117c06eac30.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4-4 删除
- 删除文档
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-868aa2461658a0c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 删除索引
可以使用head插件，点击动作删除
也可以使用postman
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-893556fe91e89ea5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4-5 查询
- 简单查询
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-15555c8554044ac2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 条件查询
查询所有的数据：
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-78ccd47a15c55c6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
获取第一条：
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-2fc30b48b7ab7c15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
关键词查询：
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-7ed40a7c1a722c17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
关键词查询，按照出版日期倒序排列：
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-011ced7b7af87523.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 聚合查询
单个分组聚合：
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-fbcaa6d72dc8aced.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
多个分组聚合：
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-c490b7b8e1f4f606.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
其他聚合：统计计算
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-3185d4fd5060af66.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 第5章 高级查询
- 子条件查询：特定字段查询所指特定值
**Query Context**：在查询过程中，除了判断文档是否满足查询条件外，ES还会计算一个**_score**来标识匹配的程度，旨在判断目标文档和查询条件匹配的有多好。
**Filter Context**：在查询过程中，只判断该文档是否满足条件，只有YES或者NO
- 复合条件查询：以一定的逻辑组合子条件查询
固定分数查询
布尔查询
#### 5-1 query
**Query Context** 常用查询
- 全文本查询：针对文本类型数据
模糊匹配：会匹配到多个词:ElasticSearch、真好
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-7a1758aef9646f16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
习语匹配：只会匹配完整的词
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-c748aa826f340fbe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
多个字段的匹配查询：
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-44be13352dbdb6b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
语法查询：
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-a20f3c0de549b893.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
字段title或author中包含ElasticSearch或者dianwei：
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-0a898e87a5d50cd2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 字段级别查询：针对结构化数据，如数字、日期等
查询字数为1000的数据：
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-d93fb9eec03f915b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
查询字数在100-174的数据：
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-403dc123772b343b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
查询出版日期2008年至今的数据：
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-f2b5e16d0285c64e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 5-2 filter
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-4b86f5094084f0a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 5-3 复合查询
- 固定分数查询
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-2375a4516585f450.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 布尔查询
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-9c0fe81f6340d825.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-dcdc56fb625216a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-77dfc56fa994bbf4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-21cee6a1a64636b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 第6章 Spring Boot集成ES
#### 6-1 SpringBoot集成ElasticSearch
```
<properties>
  <elasticsearch.version>5.5.2</elasticsearch.version>
</properties>

<dependency>
  <groupId>org.elasticsearch.client</groupId>
  <artifactId>transport</artifactId>
  <version>${elasticsearch.version}</version>
</dependency>

1.MyConfig 配置 TransportClient

    @Bean
    public TransportClient client() throws UnknownHostException {
        InetSocketTransportAddress node = new InetSocketTransportAddress(
                InetAddress.getByName("localhost"),
                9300
        );
        Settings settings = Settings.builder().put("cluster.name","wali").build();
        TransportClient client =  new PreBuiltTransportClient(settings);
        client.addTransportAddress(node);
        return client;
    }
```

#### 6-2 查询接口开发
```
    @GetMapping("/get/book/novel")
    public ResponseEntity get(@RequestParam(name = "id", defaultValue = "") String id) {
        if (StringUtils.isEmpty(id)) {
            return new ResponseEntity(HttpStatus.NOT_FOUND);
        }
        GetResponse response = this.client.prepareGet("book", "novel", id).get();
        if (!response.isExists()) {
            return new ResponseEntity(HttpStatus.NOT_FOUND);
        }
        return new ResponseEntity(response.getSource(), HttpStatus.OK);
    }

```

#### 6-3 增加接口开发
```
    @PostMapping("/add/book/novel")
    public ResponseEntity add(BookBean bookBean) {
        try {
            XContentBuilder builder = XContentFactory.jsonBuilder().startObject()
                    .field("title", bookBean.getTitle())
                    .field("author", bookBean.getAuthor())
                    .field("word_count", bookBean.getWord_count())
                    .field("publish_date", bookBean.getPublic_date())
                    .endObject();
            IndexResponse response = this.client.prepareIndex("book", "novel")
                    .setSource(builder).get();
            return new ResponseEntity(response.getId(), HttpStatus.OK);
        } catch (IOException e) {
            log.error(e.getMessage(), e);
            return new ResponseEntity(HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
```

#### 6-4 删除接口开发
```
    @DeleteMapping("/delete/book/novel")
    public ResponseEntity delete(@RequestParam(name = "id", defaultValue = "") String id) {
        if (StringUtils.isEmpty(id)) {
            return new ResponseEntity(HttpStatus.NOT_FOUND);
        }
        DeleteResponse response = this.client.prepareDelete("book", "novel", id).get();
        return new ResponseEntity(response.getResult().toString(), HttpStatus.OK);
    }
```

#### 6-5 更新接口开发
```
    @PutMapping("/update/book/novel")
    public ResponseEntity update(BookBean bookBean) {
        UpdateRequest request = new UpdateRequest("book", "novel", bookBean.getId());
        try {
            XContentBuilder builder = XContentFactory.jsonBuilder().startObject();
            if (bookBean.getAuthor() != null) {
                builder.field("author", bookBean.getAuthor());
            }
            if (bookBean.getTitle() != null) {
                builder.field("title", bookBean.getTitle());
            }
            builder.endObject();
            request.doc(builder);
            UpdateResponse response = this.client.update(request).get();
            return new ResponseEntity(response.getResult().toString(), HttpStatus.OK);
        } catch (IOException | InterruptedException | ExecutionException e) {
            log.error(e.getMessage(), e);
            return new ResponseEntity(HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
```
#### 6-6 复合查询接口开发
```
    @PostMapping("query/book/novel")
    public ResponseEntity query(
            @RequestParam(value = "author", required = false) String author,
            @RequestParam(value = "title", required = false) String title,
            @RequestParam(value = "gt_word_count", defaultValue = "0") int gtWordCount,
            @RequestParam(value = "lt_word_count", required = false) Integer ltWordCount) {
        BoolQueryBuilder boolBuilder = QueryBuilders.boolQuery();
        if (author != null) {
            boolBuilder.must(QueryBuilders.matchQuery("author", author));
        }
        if (title != null) {
            boolBuilder.must(QueryBuilders.matchQuery("title", title));
        }
        RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("word_count").from(gtWordCount);
        if (ltWordCount != null && ltWordCount > 0) {
            rangeQuery.to(ltWordCount);
        }
        boolBuilder.filter(rangeQuery);
        SearchRequestBuilder builder = this.client.prepareSearch("book")
                .setTypes("novel")
                .setSearchType(SearchType.DFS_QUERY_THEN_FETCH)
                .setQuery(boolBuilder)
                .setFrom(0)
                .setSize(10);
        log.info(String.valueOf(builder));
        SearchResponse response = builder.get();
        List<Map<String, Object>> result = new ArrayList<>();
        response.getHits().forEach(s -> result.add(s.getSource()));
        return new ResponseEntity(result, HttpStatus.OK);
    }
```
## [参考](https://blog.csdn.net/u010648555/article/details/81841188)


