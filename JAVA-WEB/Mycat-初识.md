>了解到Mycat社区发展不够务实，但是不影响我们学习技术
### 一、Mycat是什么
一个可以用于mysql读写分离和高可用的分布式数据库中间件
### 二、Mycat的下载及安装
[官方网站](http://www.mycat.io/) 和 [github地址](https://github.com/MyCATApache)
安装方法：
```
1. 上传Mycat-server-linux.tar.gz到linux服务器
2. 解压缩
tar -zxvf Mycat-server-1.4-release-20151019230038-linux.tar.gz -C /usr/local
3. 启动mycat
cd /usr/local/mycat/bin/
启动命令：./mycat start
停止命令：./mycat stop
重启命令：./mycat restart
查看状态：./mycat status
```
注意：可以使用mysql的客户端直接连接mycat服务。默认服务端口为8066
### 三、Mycat分片
1. **schema.xml**
2. **server.xml**
```
注意：若是Linux版本的mysql，则需要设置为mysql大小写不敏感，否则可能会发生表找不到的问题。
在mysql的配置文件中my.ini [mysqld] 中增加一行：lower_case_table_names = 1
```
### 四、Mycat读写分离
