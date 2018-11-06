###一、什么是Dubbo
>Dubbo是一个分布式服务框架，致力于提供高性能和透明化的RPC远程服务调用方案，以及SOA服务治理方案。

[Dubbo Github](https://github.com/apache/incubator-dubbo)
[Dubbo用户手册](http://dubbo.apache.org/books/dubbo-user-book/)
[Dubbo开发手册](http://dubbo.apache.org/books/dubbo-dev-book/)
[Dubbo管理手册](http://dubbo.apache.org/books/dubbo-admin-book/)

###二、Dubbo的架构
![image.png](https://upload-images.jianshu.io/upload_images/1956963-77958aaf8513162a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###三、Zookeeper安装
Dubbo需要使用Zookeeper作为注册中心
```
第一步：安装jdk
第二步：把zookeeper的压缩包上传到linux系统。
第三步：解压缩压缩包
tar -zxvf zookeeper-3.4.6.tar.gz
第四步：进入zookeeper-3.4.6目录，创建data文件夹。
第五步：把zoo_sample.cfg改名为zoo.cfg
[root@localhost conf]# mv zoo_sample.cfg zoo.cfg
第六步：修改data属性：dataDir=/root/zookeeper-3.4.6/data
第七步：启动zookeeper
[root@localhost bin]# ./zkServer.sh start
关闭：[root@localhost bin]# ./zkServer.sh stop
查看状态：[root@localhost bin]# ./zkServer.sh status

注意：需要关闭防火墙。
service iptables stop
永久关闭修改配置开机不启动防火墙：
chkconfig iptables off
如果不能成功启动zookeeper，需要删除data目录下的zookeeper_server.pid文件。
```

###三、Dubbo安装
1. 把dubbo-admin.war拷贝到Tomcat的webapps下，然后重启项目
2. 访问http://192.168.10.179:8080/dubbo-admin/ 
用户名：root
密码：root
3. 注意
如果监控中心和注册中心在同一台服务器上，可以不需要任何配置。
如果不在同一台服务器，需要修改配置文件：
tomcat/webapps/dubbo-admin/WEB-INF/dubbo.properties
```
#注册中心地址
dubbo.registry.address=zookeeper://127.0.0.1:2181
#root用户密码
dubbo.admin.root.password=root
dubbo.admin.guest.password=guest
```
