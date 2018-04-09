### CentOS安装
>安装vmware软件
>官网下载iso文件安装到vmware中
>使用SecureCRT终端仿真程序

### vmware设置网络
>首先在vmware中，查看NAT网络模式中的虚拟路由器的网段和IP地址
>接下来设置windows的vmnet8的ip地址和虚拟机中centos的ip地址，即可联网
>奇怪CentOS7用NAT的时候虚拟机无法ping通主机，改用桥接可以

###配置DNS
```
vi /etc/resolv.conf
nameserver 8.8.8.8
nameserver 8.8.4.4
```

### EditText有FTP插件

### 修改ip地址
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes     #是否开机启用
BOOTPROTO=static   #ip地址设置为静态
IPADDR=192.168.0.101
NETMASK=255.255.255.0
GATEWAY=192.168.0.1
```
service network restart 


### 关闭iptables并设置其开机启动/不启动
```
service iptables stop
chkconfig iptables on
chkconfig iptables off

#CentOS7关闭防火墙
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
```

### 安装JDK
```
1.上传jdk-7u45-linux-x64.tar.gz到Linux上
2.解压jdk到/usr/local目录
tar -zxvf jdk-7u45-linux-x64.tar.gz -C /usr/local/
3.设置环境变量，在/etc/profile文件最后追加相关内容
vi /etc/profile
export JAVA_HOME=/usr/local/jdk1.7.0_45
export PATH=$PATH:$JAVA_HOME/bin
4.刷新环境变量
source /etc/profile
5.测试java命令是否可用
java -version
```

### 安装Tomcat
```
1.上传apache-tomcat-7.0.68.tar.gz到Linux上
2.解压tomcat
tar -zxvf apache-tomcat-7.0.68.tar.gz -C /usr/local/
3.启动tomcat
/usr/local/apache-tomcat-7.0.68/bin/startup.sh
4.查看tomcat进程是否启动
jps
ps -ef|grep tomcat
5.查看tomcat进程端口
netstat -anpt | grep 2465
6.通过浏览器访问tomcat
http://192.168.0.101:8080/
#####################
tail -f tomcat/logs/catalina.out 查看tomcat日志
```
### 安装多个Tomcat
```
####好像只需要改端口号就行 没这么复杂
1.编辑环境变量：vi /etc/profile
export JAVA_HOME=/usr/local/jdk1.7.0_45
export PATH=$PATH:$JAVA_HOME/bin
#########first tomcat##########
CATALINA_BASE=/usr/local/tomcat
CATALINA_HOME=/usr/local/tomcat
TOMCAT_HOME=/usr/local/tomcat
export CATALINA_BASE CATALINA_HOME TOMCAT_HOME
#########first tomcat##########
#########second tomcat##########
CATALINA_2_BASE=/usr/local/tomcat2
CATALINA_2_HOME=/usr/local/tomcat2
TOMCAT_2_HOME=/usr/local/tomcat2
export CATALINA_2_BASE CATALINA_2_HOME TOMCAT_2_HOME
#########second tomcat##########
保存退出。再输入：source /etc/profile才能生效。
2.来到第二个tomcat的bin目录下
打开catalina.sh ，找到下面红字，
 # OS specific support.  $var _must_ be set to either true or false.
在下面增加如下代码
export CATALINA_BASE=$CATALINA_2_BASE
export CATALINA_HOME=$CATALINA_2_HOME

来到第二个tomcat的conf目录下
打开server.xml更改端口：
修改server.xml配置和第一个不同的启动、关闭监听端口。
修改后示例如下：
　 <Server port="9005" shutdown="SHUTDOWN">　               端口：8005->9005
<!-- Define a non-SSL HTTP/1.1 Connector on port 8080 -->
    <Connector port="9080" maxHttpHeaderSize="8192"　       端口：8080->9080
maxThreads="150" minSpareThreads="25" maxSpareThreads="75"
               enableLookups="false" redirectPort="8443" acceptCount="100"
               connectionTimeout="20000" disableUploadTimeout="true" />
<!-- Define an AJP 1.3 Connector on port 8009 -->
    <Connector port="9009"                                  端口：8009->9009
               enableLookups="false" redirectPort="8443" protocol="AJP/1.3" />
```
### 安装MySQL
```
1.上传MySQL-server-5.5.48-1.linux2.6.x86_64.rpm、MySQL-client-5.5.48-1.linux2.6.x86_64.rpm到Linux上
2.使用rpm命令安装MySQL-server-5.5.48-1.linux2.6.x86_64.rpm，缺少perl依赖
rpm -ivh MySQL-server-5.5.48-1.linux2.6.x86_64.rpm
3.安装perl依赖，上传6个perl相关的rpm包
rpm -e perl-*
4.再安装MySQL-server，rpm包冲突
rpm -ivh MySQL-server-5.5.48-1.linux2.6.x86_64.rpm
5.卸载冲突的rpm包
rpm -e mysql-libs-5.1.73-5.el6_6.x86_64 --nodeps
6.再安装MySQL-client和MySQL-server
rpm -ivh MySQL-client-5.5.48-1.linux2.6.x86_64.rpm
rpm -ivh MySQL-server-5.5.48-1.linux2.6.x86_64.rpm
7.启动MySQL服务，然后初始化MySQL
service mysql start
/usr/bin/mysql_secure_installation
8.测试MySQL
mysql -u root -p
```
##### yum安装mysql
[CentOS 7.0下使用yum安装MySQL](https://www.linuxidc.com/Linux/2016-09/134940.htm)
```
CentOS7默认数据库是mariadb,配置等用着不习惯,因此决定改成mysql,但是CentOS7的yum源中默认好像是没有mysql的。为了解决这个问题，我们要先下载mysql的repo源。

1.下载mysql的repo源
$ wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
2.安装mysql-community-release-el7-5.noarch.rpm包
$ sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
安装这个包后，会获得两个mysql的yum repo源：/etc/yum.repos.d/mysql-community.repo，/etc/yum.repos.d/mysql-community-source.repo。
3.安装mysql
$ sudo yum install mysql-server
根据提示安装就可以了,不过安装完成后没有密码,需要重置密码
4.重置mysql密码
$ mysql -u root
登录时有可能报这样的错：ERROR 2002 (HY000): Can‘t connect to local MySQL server through socket ‘/var/lib/mysql/mysql.sock‘ (2)，原因是/var/lib/mysql的访问权限问题。下面的命令把/var/lib/mysql的拥有者改为当前用户：
$ sudo chown -R root:root /var/lib/mysql
重启mysql服务
$ service mysqld restart
接下来登录重置密码：
$ mysql -u root  //直接回车进入mysql控制台
mysql > use mysql;
mysql > update user set password=password('123456') where user='root';
mysql > exit;
```
```
1. 检查安装情况
查看有没有安装过：
yum list installed MySQL* （有存在要卸载yum remove MySQL*）
rpm -qa | grep mysql*
查看有没有安装包：
yum list mysql*
2. 安装 MySQL
安装 MySQL 客户端：
yum -y install mysql
安装 MySQL 服务器端：
yum -y install mysql-server  mysql-devel
注意：安装前需要添加mysql社区repo通过输入命令：sudo rpm -Uvh http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
才能像安装MySQL的常规方法一样安装mysql： yum install mysql mysql-server mysql-libs mysql-server
检查mysql是否安装成功：
rpm -q mysql

3. 配置和启动
数据库字符集设置：
MySQL 配置文件/etc/my.cnf中加入default-character-set=utf8
查看是否生成了mysqld服务, 并设置随机启动
# chkconfig --list |grep mysql 
启动 MySQL 服务： 
service mysqld start或者/etc/init.d/mysqld start
设置开机启动：
添加开机启动：chkconfig --add mysqld;
开机启动：chkconfig mysqld on;
查看开机启动设置是否成功chkconfig --list | grep mysql* mysqld 0:关闭 1:关闭 2:启用 3:启用 4:启用 5:启用 6:关闭停止：
停止 MySQL 服务： 
service mysqld stop

4.MySQL 登陆等权限设置
登录 MySQL 数据库：
mysql -u root password 123456
首次使用创建root管理员和密码：
mysql -u root -p输入密码即可？
mysql -u root;
use mysql ;
update user set password=password("123456") where user="root";
flush privileges;

忘记密码：
service mysqld stop;
mysqld_safe --user=root --skip-grant-tables;
这一步骤执行的时候不会出现新的命令行，你需要重新打开一个窗口执行下面的命令
mysql -u root;
use mysql ;
update user set password=password("123456") where user="root";
flush privileges;

如果需要外网可以访问需要再做如下设置：
授权用户可以从远程登陆
grant all PRIVILEGES on *.* to root@'%'  identified by 'pwd123';
flush privileges ;

5. 远程访问，开放防火墙的端口号 MySQL
 一般开发测试我直接把防火墙关闭，生产的开发对应端口即可：
su root
service iptables stop #关闭防火墙
service iptables status #验证是否关闭
chkconfig iptables off #关闭防火墙的开机自动运行
chkconfig –list | grep iptables #验证防火墙的开机自动运行
vim /etc/sysconfig/selinux # 禁用selinux，将SELINUX=disabled
6. Linux MySQL 的几个重要目录
数据库目录 /var/lib/mysql/
配置文件 /usr/share /mysql（mysql.server命令及配置文件）
相关命令 /usr/bin（mysqladmin mysqldump等命令）
启动脚本 /etc/rc.d/init.d/（启动脚本文件mysql的目录）

7. 删除 MySQL  数据库
如果使用的是 yum 安装的 mysql，需要删除的话，就使用如下命令：
yum -y remove mysql*
然后将 /var/lib/mysql文件夹下的所有文件都删除干净，最后再重新执行上面的安装步骤。
```
[参考 Linux下yum安装MySQL](http://blog.csdn.net/jerome_s/article/details/52883234)
[参考 linux 安装mysql数据库](http://www.cnblogs.com/nzplearnSite/p/5002775.html)

```
常用命令
1. 创建数据库
create database test;
2. 创建用户
create user lh identified by '1234';
3. 给用户赋予test数据库的权限
grant all on test.* to lh;
```

### 安装Zookeeper
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

### 安装dubbo
```
第一步：安装tomcat
第二步：部署监控中心
[root@localhost ~]# cp dubbo-admin-2.5.4.war apache-tomcat-7.0.47/webapps/dubbo-admin.war 
第三步：启动tomcat
第四步：访问http://192.168.25.167:8080/dubbo-admin/
用户名：root
密码：root

如果监控中心和注册中心在同一台服务器上，可以不需要任何配置。
如果不在同一台服务器，需要修改配置文件：
/root/apache-tomcat-7.0.47/webapps/dubbo-admin/WEB-INF/dubbo.properties
dubbo.registry.address=zookeeper://127.0.0.1:2181 #注册中心的地址
dubbo.admin.root.password=root #root用户的密码
dubbo.admin.guest.password=guest
```
### 本地YUM源制作
```
1.官网下载DVD.iso
2.配置好这台服务器的IP地址
3.上传CentOS-7-x86_64-DVD-1708.iso到服务器
4.将CentOS-7-x86_64-DVD-1708.iso镜像挂载到某个目录
mkdir /var/iso
mount -o loop CentOS-7-x86_64-DVD-1708.iso /var/iso
5.修改本机上的YUM源配置文件，将源指向自己
备份原有的YUM源的配置文件
cd /etc/yum.repos.d/
rename .repo .repo.bak *
vi CentOS-Local.repo

[base]
name=CentOS-Local
baseurl=file:///var/iso
gpgcheck=1
enabled=1   #很重要，1才启用
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#添加上面内容保存退出
6.清除YUM缓冲
yum clean all
7.列出可用的YUM源
yum repolist
8.安装相应的软件
yum install -y httpd
9.开启httpd使用浏览器访问[<u>http://192.168.0.100:80</u>](http://192.168.0.100:80)（如果访问不通，检查防火墙是否开启了80端口或关闭防火墙）
service httpd start
10.将YUM源配置到httpd（Apache Server）中，其他的服务器即可通过网络访问这个内网中的YUM源了
cp -r /var/iso/ /var/www/html/CentOS-7
11.取消先前挂载的镜像
umount /var/iso
12.在浏览器中访问http://192.168.0.100/CentOS-7/
13.让其他需要安装RPM包的服务器指向这个YUM源，准备一台新的服务器，备份或删除原有的YUM源配置文件
cd /etc/yum.repos.d/
rename .repo .repo.bak *
vi CentOS-Local.repo

[base]
name=CentOS-Local
baseurl=file:///var/iso
gpgcheck=1
enabled=1   #很重要，1才启用
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

添加上面内容保存退出
14.在这台新的服务器上执行YUM的命令
yum clean all
yum repolist
15.安装相应的软件
yum install -y gcc
16、加入依赖包到私有yum的repository
进入到repo目录
执行命令：  createrepo  .

用了阿里的yum源地址:
http://mirrors.aliyun.com/repo/Centos-7.repo替换/etc/yum.repos.d/CentOS-Base.repo 就可以了
或者使用如下方法配置阿里云yum源：
备份下原来的yum源
mv CentOS-Base.repo CentOS-Base.repo_bak
阿里云yum源：
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all
yum makecache
```

### 安装nginx
```
第一步：安装gcc的环境。
yum install gcc-c++
第二步：安装第三方的开发包PCRE、zlib、openssl
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel
第三步：解压缩压缩包
tar -zxvf nginx-1.8.0.tar.gz  -C /usr/local/
第四步：进入nginx-1.8.0目录，使用configure命令创建makeFile文件。
./configure
注意：启动nginx之前，上边将临时文件目录指定为/var/temp/nginx，需要在/var下创建temp及nginx目录
mkdir /var/temp/nginx/client -p
第五步：make
第六步：make install
可以看到/usr/local/nginx下有三个目录
第七步：进入sbin目录
启动nginx ：./nginx 
使用 ps aux|grep nginx可以看到端口

关闭nginx： ./nginx -s stop 或者./nginx -s quit
重启nginx： 先关闭后启动。
刷新配置文件：./nginx -s reload
访问nginx：192.168.10.200 默认是80端口
注意：是否关闭防火墙
```
```
   # HTTPS server
    #通过端口区分虚拟主机
    server {
        listen       81;
        server_name  localhost;
        location / {
            root   html1;
            index  index.html index.htm;
        }
    }
    #通过域名区分虚拟主机
    server {
        listen       80;
        server_name  www.liuh80.com;
        location / {
            root   html1;
            index  index.html index.htm;
        }
    }
    server {
        listen       80;
        server_name  www.liuh81.com;
        location / {
            root   html2;
            index  index.html index.htm;
        }
    }
    # 反向代理
    upstream tomcat1 {
        server 192.168.10.200:8080;
    }
    server {
        listen       80;
        server_name  www.liuhui.com;
        location / {
            proxy_pass   http://tomcat1;
            index  index.html index.htm;
        }
    }
    upstream tomcat2 {
        #负载均衡
        server 192.168.10.200:8081;
        server 192.168.10.200:8082 weight=2;
    }
    server {
        listen       80;
        server_name  www.liuhui2.com;

        location / {
            proxy_pass   http://tomcat2;
            index  index.html index.htm;
        }
    }
```

### 安装keepalived
```
第一步：安装环境
yum -y install kernel-devel*
yum -y install openssl-*
yum -y install popt-devel
yum -y install lrzsz
yum -y install openssh-clients
yum -y install libnl libnl-devel popt
第二步：解压
tar -zxvf keepalived-1.2.15.tar.gz -C /usr/local
第三步：cd keepalived-1.2.15
执行配置命令
./configure --prefix=/usr/local/keepalived
第四步：
编译
make
安装
make install
至此安装成功

第五步：拷贝执行文件
cp /usr/local/keepalived/sbin/keepalived /usr/sbin/
第六步：将init.d文件拷贝到etc下,加入开机启动项
cp /usr/local/keepalived/etc/rc.d/init.d/keepalived /etc/init.d/keepalived
第七步：将keepalived文件拷贝到etc下，加入网卡配置
cp /usr/local/keepalived/etc/sysconfig/keepalived /etc/sysconfig/ 
第八步：创建keepalived文件夹
mkdir -p /etc/keepalived
第九步：将keepalived配置文件拷贝到etc下
cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf
第十步：添加可执行权限
chmod +x /etc/init.d/keepalived

第六步：加入开机启动
chkconfig --add keepalived	#添加时必须保证/etc/init.d/keepalived存在
chkconfig keepalived on
添加完可查询系统服务是否存在：chkconfig --list

第七步：启动keepalived
启动：service keepalived start
停止：service keepalived stop
重启：service keepalived restart

第八步：配置日志文件
1.将keepalived日志输出到local0：
vi /etc/sysconfig/keepalived
KEEPALIVED_OPTIONS="-D -d -S 0"
2.在/etc/rsyslog.conf里添加:
local0.*  /var/log/keepalived.log
3.重新启动keepalived和rsyslog服务：
service rsyslog restart 
service keepalived restart

第九步：4.打开防火墙的通讯地址
iptables -A INPUT -d 224.0.0.18 -j ACCEPT
/etc/rc.d/init.d/iptables save

```
### 安装redis
```
如果没有gcc需要在线安装。yum install gcc-c++
安装步骤：
第一步：redis的源码包 redis-3.0.0.tar.gz上传到linux系统。
第二步：解压缩redis。
第三步：编译。进入redis源码目录。make 
第四步：安装。make install PREFIX=/usr/local/redis
PREFIX参数指定redis的安装目录。一般软件安装到/usr目录下
注意：可能会报错：You need tcl 8.5 or newer in order to run the Redis test
需要安装tcl： yum -y install tcl
第五步：前端启动。在redis的安装目录下直接启动redis-server
第六步：后端启动。
cp redis.conf /usr/local/redis/bin/
修改配置文件redis.conf：daemonize yes
 ./redis-server redis.conf
查看redis进程：ps aux|grep redis
```
### Solr服务搭建
```
第一步：把solr 的压缩包上传到Linux系统
第二步：解压solr。
第三步：安装Tomcat，解压缩即可。
第四步：把solr部署到Tomcat下。
第五步：解压缩war包。启动Tomcat解压。
第六步：把/usr/local/solr-4.10.3/example/lib/ext目录下的所有的jar包，添加到solr工程中。
[root@localhost ext]# cd /usr/local/solr-4.10.3/example/lib/ext
[root@localhost ext]# cp * /usr/local/tomcat/webapps/solr/WEB-INF/lib/
第七步：创建一个solrhome。/example/solr目录就是一个solrhome。复制此目录到/usr/local/solr/solrhome
[root@localhost example]# cd /usr/local/solr-4.10.3/example
[root@localhost example]# cp -r solr /usr/local/solr/solrhome

solrhome目录建立文件solr.xml，内容：
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<solr> 
</solr>
第八步：关联solr及solrhome。需要修改solr工程的web.xml文件。
 <env-entry>
       <env-entry-name>solr/home</env-entry-name>
       <env-entry-value>/usr/local/solr/solrhome</env-entry-value>
       <env-entry-type>java.lang.String</env-entry-type>
    </env-entry>
第九步：启动Tomcat
访问http://192.168.25.154:8080/solr/
和windows下的配置完全一样。

```
###Tips
[SecureCRT配色推荐和永久设置](http://blog.csdn.net/zq710727244/article/details/53909801)
