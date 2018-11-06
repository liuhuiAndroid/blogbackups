###一、什么是Nginx
Nginx 是一个高性能的HTTP和反向代理服务器，也是一个IMAP/POP3/SMTP服务器。

###二、Nginx应用场景
1. **Http服务器**
Nginx是一个Http服务可以独立提供Http服务。可以做网页静态服务器。
2. **虚拟主机**
可以实现在一台服务器虚拟出多个网站。例如个人网站使用的虚拟主机。
3. **反向代理，负载均衡**
当网站的访问量达到一定程度后，单台服务器不能满足用户的请求时，需要用多台服务器集群可以使用Nginx做反向代理。并且多台服务器可以平均分担负载，不会因为某台服务器负载高宕机而某台服务器闲置的情况。

###三、Nginx安装
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
访问nginx：192.168.10.179 默认是80端口
注意：是否关闭防火墙
```

###四、配置虚拟主机和反向代理
1. 虚拟主机：就是在一台服务器启动多个网站。
2. 反向代理：反向代理服务器决定哪台服务器提供服务。
Nginx实现反向代理
1 - 需要安装两个Tomcat，分别运行在8080和8081端口
2 - Nginx重新加载配置文件
3 - 配置域名：在Hosts文件中添加域名和IP的映射关系
3. 具体配置
nginx/conf/nginx.conf
```
    # HTTPS server
    # 通过端口区分虚拟主机
    server {
        listen       81;
        server_name  localhost;
        location / {
            root   html1;
            index  index.html index.htm;
        }
    }
    # 通过域名区分虚拟主机
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
        server 192.168.10.179:8080;
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
        server 192.168.10.179:8081;
        server 192.168.10.179:8082 weight=2;
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

###五、Nginx的高可用
未完待续
