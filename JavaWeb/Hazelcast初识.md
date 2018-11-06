###一、什么是[Hazelcast](https://hazelcast.org/)
>Hazelcast作为一个高度可扩展的数据分发和集群平台，提供了高效的、可扩展的分布式数据存储、数据缓存。

###二、Hazelcast的使用
```
<dependency>
    <groupId>com.hazelcast</groupId>
    <artifactId>hazelcast</artifactId>
    <version>3.8.6</version>
</dependency>

<dependency>
    <groupId>com.hazelcast</groupId>
    <artifactId>hazelcast-spring</artifactId>
    <version>3.8.6</version>
</dependency>
```
客户端可以安装Hazelcast Clients查看缓存

配置
```
hazelcast.xml

application.yml
spring:
  cache:
    type: hazelcast
```
```
@SpringBootApplication
@EnableCaching
@EntityScan("com.lh.entity")
@EnableScheduling
public class SellerApp {

    public static void main(String[] args){
        SpringApplication.run(SellerApp.class);
    }

}
```
```
@Cacheable(cacheNames = CACHE_NAME)
@CachePut(cacheNames = CACHE_NAME, key = "#product.id")
@CacheEvict(cacheNames = CACHE_NAME)
```

```
@Autowired
HazelcastInstance hazelcastInstance;

Map map = hazelcastInstance.getMap(CACHE_NAME);
```
