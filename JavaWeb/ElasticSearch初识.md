# 零
- 使用[**ElasticSearch** 中文发行版](https://github.com/medcl/elasticsearch-rtf)安装ElasticSearch
- 运行Elasticsearch
```
./bin/elasticsearch
```
访问 http://localhost:9200/ 测试
测试：`curl "http://elastic:changeme@localhost:9200/?pretty"`
- 安装 X-Pack
X-Pack是一个Elastic Stack的扩展，将安全，警报，监视，报告和图形功能包含在一个易于安装的软件包中。
- 验证 X-Pack
此时访问 http://localhost:9200/ 默认账号密码：elastic changeme
- 安装 [**Kiabna**](https://www.elastic.co/downloads/kibana)
Kibana是一个为 ElasticSearch 提供的数据分析的 Web 接口。可使用它对日志进行高效的搜索、可视化、分析等各种操作。
- 验证 X-Pack
访问 http://localhost:5601 测试
login is currently disabled. administrators should consult the kibana logs for more details.
解决办法：
```
# vim config/kibana.yml  
elasticsearch.username: "elastic"  
elasticsearch.password: "changeme"  //elasticsearch-keystore设置的密码  
```

### 什么是ElasticSearch
- 基于Apache Lucene构建的开源搜索引擎
- 采用Java编写，提供简单易用的RESTFul API
- 轻松的横向扩展，可支持PB级的结构化或非结构化数据处理 

### 应用场景
- 可应用场景
海量数据分析引擎
站内搜索引擎
数据仓库

