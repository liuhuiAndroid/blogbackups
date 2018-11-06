# 一、了解Nginx
### 介绍
Nginx是一个**高效**、**可靠**的web服务和代理中间件
### 环境准备
四项确认
```
1.确认系统网络 
ping www.baidu.com
2.确认yum可用
yum list|grep gcc
3.确认关闭iptables规则
iptables -L 列出所有规则
iptables -F 清除所有规则
iptables -t nat -L 查看nat表所有链的规则
iptables -t nat -F 清除nat表所有链的规则
4.确认停用SELinux
SELinux处于启动状态致使很多服务端口默认是关闭的。
getenforce 如果为disabled就是已经关闭
setenforce 0 关闭
```
两项安装
```
yum -y install gcc gcc-c++ autoconf pcre pcre-devel make automake
yum -y install wget httpd-tools vim
```
一次初始化
```
cd /opt;mkdir app download logs work backup
```

# 二、基础篇
###  什么是Nginx
Nginx是一个开源且高性能、可靠的HTTP中间件、代理服务。

### 常见的中间件服务
HTTPD、IIS、GWS

### Nginx特性_实现优点
- 1、IO多路复用epoll
什么是IO多路复用？
多个描述符的I/O操作都能在一个线程内并发交替地顺序完成，这就叫I/O多路复用，这里的“复用”指的是复用同一个线程。
什么是epoll？
epoll是Linux内核为处理大批量文件描述符而作了改进的poll
- 2、轻量级
功能模块少
代码模块化
- 3、CPU亲和
什么是CPU亲和？
是一种把CPU核心和Nginx工作进程绑定方式，把每个worker进程固定在一个cpu上执行，减少切换cpu的cache miss，获得更好的性能。 
- 4、sendfile
把所有静态文件传输只通过内核空间传递给Socket、响应给用户。

### Nginx快速安装
[官网](http://nginx.org)提供了一个yum源，拷贝到linux的/etc/yum.repos.d/nginx.repo,
注意os改为centos,版本号改为7
```
yum list | grep nginx
yum install nginx -y
nginx -v 查看版本
nginx -V 查看编译参数
```

### Nginx的目录和配置语法
- 1. Nginx安装目录
yum安装都是安装rpm包
rpm -ql nginx

    | 路径  |  类型         |  作用               |
    | --------   | -----:   | :----: |
    |  /etc/logrotate.d/nginx   |  配置文件  |   Nginx日志轮转，用于logrotate服务的日志切割  |
    |  /etc/nginx/nginx.conf   /etc/nginx/conf.d/default.conf |  目录、配置文件  | Nginx主配置文件 |
    |  /etc/nginx/fastcgi_params  /etc/nginx/uwsgi_params  /etc/nginx/scgi_params |  配置文件  | cgi配置相关、fastcgi配置 |
    |  /etc/nginx/mime.types | 配置文件  | 设置http协议的Content-Type与扩展名对应关系 |
    |  /usr/lib/systemd/system/nginx-debug.service /usr/lib/systemd/system/nginx.service /etc/sysconfig/nginx /etc/sysconfig/nginx-debug | 配置文件  | 用于配置出系统守护进程管理器管理方式 |
    |  /usr/lib64/nginx/modules /etc/nginx/modules | 目录  | Nginx模块目录 |
    |  /usr/sbin/nginx /usr/sbin/nginx-debug | 命令  | Nginx服务的启动管理的终端命令 |
    |  /var/cache/nginx | 目录  | Nginx的缓存目录 |
    |  /var/log/nginx | 目录  | Nginx的日志目录 |

- 2. Nginx编译配置参数
nginx -V

    | 编译选项  |  作用               |
    | --------   | -----:   |
    |  --perfix=/etc/nginx   |  安装目的目录或路径  |
    |  --sbin-path=/usr/sbin/nginx   |  安装目的目录或路径  |
    |  --modules-path=/usr/lib64/nginx/modules  |  安装目的目录或路径  |
    |  ...  |  安装目的目录或路径  |
    |  --http-client-body-temp-path=/var/cache/nginx/client_temp  |  执行对应模块时，Nginx所保留的临时性文件  |
    |  ...  |  执行对应模块时，Nginx所保留的临时性文件  |
    |  --user=nginx  |  设定Nginx进程启动的用户和组用户  |
    |  --group=nginx  |  设定Nginx进程启动的用户和组用户  |

- 3. 默认配置语法
Nginx默认配置语法

    |   |    |
    | --------   | -----:   |
    |  user   |  设置nginx服务的系统使用用户  |
    |  work_processes  |  工作进程数  |
    |  error_log  |  nginx的错误日志  |
    |  pid   |  nginx服务启动时候pid  |

    |   |    |    |
    | --------   | -----:   | -----:   |
    |  events  |  worker_connections  |  每个进程允许最大连接数  |
    |  events  |  use  | 工作进程数  |

- 4. 默认配置与默认站点启动
/etc/nginx/nginx.conf配置
/etc/nginx/conf.d/default.conf配置
systemctl start nginx.service 启动nginx服务
systemctl status nginx.service 查看服务当前状态
systemctl restart nginx.service 重新启动服务
systemctl reload nginx.service 重新加载服务配置文件
注：systemctl命令是系统服务管理器指令

### HTTP请求
```
curl http://www.baidu.com
curl -v http://www.baidu.com >/dev/null
注：curl命令是一个利用URL规则在命令行下工作的文件传输工具。-v显示请求详细信息
```

### Nginx日志
Nginx日志类型：error_log 和 access_log
log_format：用来设置日志格式
```
tail -f /var/log/nginx/error.log
tail -n 200 /var/log/nginx/access.log
修改nginx.conf监听头 log_format main '$http_user_agent'
nginx -t -c /etc/nginx/nginx.conf
nginx -s reload -c /etc/nginx/nginx.conf
curl 118.126.111.144
tail -n 200 /var/log/nginx/access.log
```
Nginx变量
    
|   |    |
| --------   | -----:   |
|  HTTP请求变量   |  arg_PARAMETER、http_HEADER、sent_http_HEADER  |
|  内置变量  |  Nginx内置的  |
|  自定义变量  |  自己定义  |

### Nginx模块讲解
- **Nginx官方模块**
###### http_stub_status_module配置
| 编译选项  |  作用               |
| --------   | -----:   |
|  --with-http_stub_status_module   |  Nginx的客户端状态  |
```
在default.conf添加如下配置：

    location /mystatus {
        stub_status;
    }

nginx -tc /etc/nginx/nginx.conf
nginx -s reload -c /etc/nginx/nginx.conf
访问 http://118.126.111.144/mystatus
```

###### http_random_index_module配置
| 编译选项  |  作用               |
| --------   | -----:   |
|  --with-http_random_index_module   |  目录中选择一个随机主页  |
```
location / {
  root /opt/app/code;
  random_index on;
}

nginx -tc /etc/nginx/nginx.conf
systemctl reload nginx
ps -aux | grep nginx 
访问 http://118.126.111.144
```

###### http_sub_module配置
| 编译选项  |  作用               |
| --------   | -----:   |
|  --with-http_sub_module   |  HTTP内容替换  |
```
location / {
  root /opt/app/code;
  index index.html index.htm;
  sub_filter '<a>lh' '<a>hw';
  sub_filter_once off;
}

systemctl reload nginx
ps -aux | grep nginx 
访问 http://118.126.111.144
```
- 第三方模块
nginx + lua

### Nginx的请求限制
| Nginx的请求限制  |     |
| --------   | -----:   |
|  连接频率限制   |  limit_conn_module  |
|  请求频率限制   |  limit_req_module  |
```
limit_conn_zone $binary_remote_addr zone=conn_zone:1m;
limit_req_zone $binary_remote_addr zone=req_zone:1m rate=1r/s;
server {
  location {
    root /opt/app/code;
    #limit_conn conn_zone 1; # 同一时刻只允许一个连接
    #limit_req zone=req_zone burst=3 nodelay; # 3个延迟响应
    #limit_req zone=req_zone burst=3;
    #limit_req zone=req_zone;
    index index.html index.htm
  }
}

nginx -s reload -c /etc/nginx/nginx.conf
ab -n 20 -c 20 http://118.126.111.144/1.html
tail -f /var/log/nginx/error.log
```
### Nginx的访问控制
|   |     |
| --------   | -----:   |
|  基于IP的访问控制   |  http_access_module  |
|  基于用户的信任登录   |  http_auth_basic_module  |
- http_access_module
```
location ~ ^/admin.html {
  root /opt/app/code;
  deny 222.122.111.11;
  allow all;
  index index.html index.htm
}

location ~ ^/admin.html {
  root /opt/app/code;
  allow 222.122.111.0/24;
  deny all;
  index index.html index.htm
}

nginx -tc /etc/nginx/nginx.conf
nginx -s reload -c /etc/nginx/nginx.conf
```
```
http_access_module局限性
方法一：采用别的HTTP头信息控制访问，如：HTTP_X_FORWARD_FOR
方法二：结合geo模块作
方法三：通过HTTP自定义变量传递
```

- http_auth_basic_module
```
rpm -qf /usr/bin/htpasswd 
yum install httpd-tools -y # 安装htpasswd 
htpasswd -c ./auth_conf lh

location ~ ^/admin.html {
  root /opt/app/code;
  auth_basic "Auth access!input your password!";
  auth_basic_user_file /etc/nginx/auth_conf;
  index index.html index.htm
}

nginx -tc /etc/nginx/nginx.conf
nginx -s reload -c /etc/nginx/nginx.conf
```
```
http_auth_basic_module局限性
一、用户信息依赖文件方式
二、操作管理机械，效率低下
三、解决方案
1.Nginx结合LUA实现高效验证
2.Nginx和LDAP打通，利用nginx-auth-ldap模块
```
# 三、场景实践篇
### Nginx作为静态资源web服务_静态资源类型
非服务器动态运行生成的文件
默认路径是：/usr/share/nginx/html/

### Nginx作为静态资源web服务_CDN场景
什么是CDN?
>CDN的全称是Content Delivery Network，即内容分发网络。其基本思路是尽可能避开互联网上有可能影响数据传输速度和稳定性的瓶颈和环节，使内容传输的更快、更稳定。


### Nginx作为静态资源web服务_配置语法
- sendfile 
- tcp_nopush
作用：sendfile开启的情况下，提高网络包的传输效率
- tcp_nodelay
作用：keepalive连接下，提高网络包的传输实时性
- gzip
作用：压缩传输
- http_gzip_static_module
作用：预读gzip功能

### Nginx作为静态资源web服务_场景演示
static_server.conf
关闭和开启gzip压缩，观察网络传输大小
开启gzip_static，gip ./test.img，访问test.img

### Nginx作为静态资源web服务_浏览器缓存原理
HTTP协议定义的缓存机制
校验过期机制
|   |     |
| --------   | -----:   |
|  校验是否过期   |  Expires、Cache-Control(max-age)  |
|  协议中Etag头信息校验   |  Etag  |
|  Last-Modified头信息校验   |  Last-Modified  |

static_server.conf
关闭和开启expires，观察Etag和Last-Modified头，出现304

### Nginx作为静态资源web服务_跨站访问
为什么浏览器禁止跨域访问？
不安全，容易出现CSRF攻击！
Nginx怎么做：Access-Control-Allow-Origin

/opt/app/code/test_origin.html 测试跨域
static_server.conf
```
# 添加跨域
add_header Access-Control-Allow-Origin http://www.baidu.com;
add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,OPTIONS;
```

### Nginx作为静态资源web服务_防盗链
目的：防止资源被盗用
首要方式：区别哪些请求是非正常的用户请求

基于http_refer防盗链配置模块
log_format已经记录了http_refer头
/opt/app/code/test_refer.html测试，观察日志http_refer信息
编辑static_server.conf，
```
valid_referers none blocked 111.22.11.22;
if($invalid_referer){
  return 403;
}

curl -I http://111.22.11.22/wei.png # 没有http_refer信息
curl -e "http://www.baidu.com" -I http://111.22.11.22/wei.png # 有http_refer信息 403
curl -e "111.22.11.22" -I http://111.22.11.22/wei.png # 有http_refer信息 200
```

### Nginx作为代理服务_代理服务
-  正向代理和反向代理？翻墙属于正向代理。
区别在于代理的对象不一样。正向代理代理的对象是客户端，反向代理代理的对象是服务端。

### Nginx作为代理服务_配置语法及反向代理场景
```
fx_proxy.conf realserver.conf 
test_proxy.html

location ~ /test_proxy.html$ {
  proxy_pass http://127.0.0.1:8080;
}
```

### Nginx作为代理服务_正向代理配置场景
```
admin.conf
jesonc.html

location / {
  if( $http_x_forwarded_for !~* "^116\.62\.103\.228"){
  }
  root /opt/app/code;
  index index.html index.htm
}

228服务器做正向代理：
zx_proxy.conf

resolver 8.8.8.8;
location / {
  proxy_pass http://$http_host$request_uri;
}
```
浏览器设置代理服务器为116.62.103.228，比如说SwitchySharp工具

### Nginx作为代理服务_代理配置语法补充
文档在nginx.org中documentation中搜索proxy
- 缓冲区 proxy_buffering
扩展：proxy_buffer_size、proxy_buffers、proxy_busy_buffers_size
- 跳转重定向 proxy_redirect
- 头信息 proxy_set_header
扩展：proxy_hide_header、proxy_set_body
- 超时 proxy_connect_timeout
扩展：proxy_read_timeout、proxy_send_timeout

```
location / {
  proxy_pass http://127.0.0.1:8080;
  include proxy_params;
}

proxy_params文件：
  proxy_redirect default;
  proxy_set_header Host $http_host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_connect_timeout 30;
  proxy_send_timeout 60;
  proxy_read_timeout 60;
  proxy_buffer_size 32k;
  proxy_buffering on;
  proxy_buffers 4 128k;
  proxy_busy_buffers_size 256k;
  proxy_max_temp_file_size 256k;
```

### Nginx作为负载均衡服务_负载均衡与Nginx
- GSLB 全局负载均衡
调度节点和服务节点
- SLB —— Nginx是一个典型的七层负载均衡的SLB
调度节点和服务节点

### Nginx作为负载均衡服务_配置场景
- upstream
```
启动三个不同端口的服务： server1.conf、server2.conf、server3.conf
请求不同端口返回不同背景的html

upstream im {
  server 118.126.111.144:8001;
  server 118.126.111.144:8002;
  server 118.126.111.144:8003;
}

server {
  location / {
    proxy_pass http://im;
    include proxy_params;
  }
}

iptables -I INPUT -p tcp --dport 8002 -j DROP # 测试关闭一个端口，查看负载均衡效果
```

### Nginx作为负载均衡服务_server参数讲解
```
upstream backend {
  server backend1.example.com weight=5;
  server backend2.example.com:8080;
  server unix:/tmp/backend3;

  server backup1.example.com:8080 backup;
  server backup2.example.com:8080 backup;
}
```

后端服务器在负载均衡调度中的状态
|   |     |
| --------   | -----:   |
|  down   |  当前的server暂时不参与负载均衡  |
|  backup   |  预留的备份服务器  |
|  max_fails   |  允许请求失败的次数  |
|  fail_timeout   |  经过max_fails失败后，服务暂停的时间  |
|  max_conns   |  限制最大的接收的连接数  |

### Nginx作为负载均衡服务_backup状态演示
```
upstream im {
  server 118.126.111.144 down;
  server 118.126.111.144:8002 backup;
  server 118.126.111.144:8003 max_fails=1 fail_timeout=10s;
}

iptables -F
iptables -I INPUT -p tcp --dport 8003 -j DROP # 测试关闭一个端口，查看负载均衡效果
iptables -F
```

### Nginx作为负载均衡服务_轮询策略与加权轮询
|   |     |
| --------   | -----:   |
|  轮询   |  按时间顺序逐一分配到不同的后端服务器  |
|  加权轮询   |  weight值越大，分配到的访问几率越高  |
|  ip_hash   |  每个请求按访问IP的hash结果分配，这样来自同一个IP的固定访问一个后端服务器  |
|  url_hash   |  按照访问的URL的hash结果来分配请求，是每个URL定向到同一个后端服务器  |
|  least_conn   |  最少链接数，哪个机器连接数少就分发  |
|  hash关键数值   |  hash自定义的key  |
- 加权轮询
```
upstream im {
  server 118.126.111.144:8001 down;
  server 118.126.111.144:8002 backup;
  server 118.126.111.144:8003 max_fails=1 fail_timeout=10s;
}
```
- 负载均衡策略ip_hash方式
```
upstream im {
  ip_hash;
  server 118.126.111.144:8001;
  server 118.126.111.144:8002;
  server 118.126.111.144:8003;
}
```

- 负载均衡策略url_hash策略和hash关键数值
```
upstream im {
  hash $request_uri;
  server 118.126.111.144:8001;
  server 118.126.111.144:8002;
  server 118.126.111.144:8003;
}
```

### Nginx作为缓存服务_Nginx作为缓存服务
- 缓存类型
服务端缓存（redis、memcache）、代理缓存（Nginx）、客户端缓存（缓存在浏览器上）

### Nginx作为缓存服务_缓存服务配置语法
- proxy_cache配置语法
proxy_cache_path
proxy_cache 
- 缓存过期周期
proxy_cache_valid
- 缓存的维度
proxy_cache_key

### Nginx作为缓存服务_场景配置演示
```
upstream im {
  hash $request_uri;
  server 118.126.111.144:8001;
  server 118.126.111.144:8002;
  server 118.126.111.144:8003;
}

proxy_cache_path /opt/app/cache levels=1:2 keys_zone=lh_cache:10m max_size=10g inactive=60m use_temp_path=off;

server {
  location / {
    proxy_cache lh_cache; # off 关闭缓存
    proxy_pass http://lh;
    proxy_cache_valid 200 304 12h;
    proxy_cache_valid any 10m;
    proxy_cache_key $host$uri$is_args$args;
    add_header Nginx-Cache "$upstream_cache_status";

    proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
    include proxy_params;
  }
}
```
- 如何清理指定缓存？
方式一：rm -rf 缓存目录内容
方式二：第三方扩展模块ngx_cache_purge
- 如何让部分页面不缓存？
proxy_no_cache
```
server {

  if($request_uri ~ ^/(url3|login|register|password\/reset)){
    set $cookie_nocache 1;
  }

  location / {
    proxy_cache lh_cache; # off 关闭缓存
    proxy_pass http://lh;
    proxy_cache_valid 200 304 12h;
    proxy_cache_valid any 10m;
    proxy_cache_key $host$uri$is_args$args;
    proxy_no_cache $cookie_nocache $arg_nocache $arg_comment;
    proxy_no_cache $http_pragma $http_authorization;
    add_header Nginx-Cache "$upstream_cache_status";

    proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
    include proxy_params;
  }
}
```
### Nginx作为缓存服务_分片请求
- 大文件分片请求
http_slice_module
优势：每个子请求收到的数据会形成一个独立文件，一个请求断了，其他请求不受影响。
缺点：当文件很大或者slice很小的时候，可能会导致文件描述符耗尽等情况。
