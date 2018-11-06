# 四、深度学习篇
### Nginx动静分离_动静分离
- 什么是动静分离？
通过中间件将动态请求和静态请求分离。
- 为什么要动静分离？
分离资源，减少不必要的请求消耗，减少请求延时。
- 动静分离场景
![image.png](https://upload-images.jianshu.io/upload_images/1956963-7925d436dfeb51fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

test_mysite.html
```
upstream java_api {
  server 127.0.0.1:8080;
}
server {
  location ~ \.jsp$ {
    proxy_pass http://java_api;
    index index.html index.htm;
  }

  location ~ \.(jpg|png|gif)$ {
    expires 1h;
    gzip on;
  }
}
```
```
sh catalina.sh stop # 关闭tomcat
ps -aux | grep java # 查看有没有java进程
kill id # 强制杀死java进程
ps -aux | grep java # 查看有没有java进程
sh catalina,sh start;tail -f ../logs/catalina.out
netstat -luntp | grep 8080 # 查看8080端口

sh catalina.sh stop # 关闭tomcat
ps -aux | grep java # 查看有没有java进程
kill id # 强制杀死java进程
关闭tomcat后查看页面，可以看到动态资源请求不到，但是静态资源可以访问
```

### Rewrite规则_rewrite规则作用
- Nginx的rewrite规则
实现url重写以及重定向
- 场景
1.URL访问跳转，支持开发设计
页面跳转、兼容性支持、展示效果等
2.SEO优化
3.维护
后台维护、流量转发等
4.安全

### Rewrite规则_rewrite配置语法和正则表达式
- 配置语法
```
rewrite regex replacement[flag];
rewrite ^(.*)$ /pages/maintain.html break; # 配置维护页面
```
- 正则表达式

|   |    |
| --------   | -----:   |
|  .  |  匹配除换行符意外的任意字符  |
|  ?  |  重复0次或1次  |
|  +  |  重复1次或更多次  |
|  *  |  最少连接数，哪个机器连接数少就分发  |
|  \d  |  匹配数字  |
|  ^  |  匹配字符串的开始  |
|  $  |  匹配字符串的介绍  |
|  {n}  |  重复n次  |
|  {n,}  |  重复n次或更多次  |
|  [c]  |  匹配单个字符c  |
|  [a-z]  |  匹配a-z小写字母的任意一个  |
|  \  |  转义字符  |
1.正则 表达式了解透彻
2.终端测试命令pcretest

### Rewrite规则_rewrite规则中的flag
|   |    |
| --------   | -----:   |
|  last  |  停止rewrite检测  |
|  break  |  停止rewrite检测  |
|  redirect  |  返回302临时重定向，地址栏会显示跳转后的地址  |
|  permanent  |  返回301永久重定向，地址栏会显示跳转后的地址  |

- last和break的区别
```
server {
  root /opt/app/code;
  location ~ ^/break {
    rewrite ^/break /test/ break; # 停留在这一级
  }

  location ~ ^/last {
    rewrite ^/last /test/ last;  # 重新建立新的连接
  }


  location /test/ {
    default_type application/json;
    return 200 '{"status":"success"}';
  }
}

分别访问118.126.111.144/test、118.126.111.144/break、118.126.111.144/last 查看
```

- redirect和permanent区别
```
server {
  root /opt/app/code;
  location ~ ^/break {
    rewrite ^/break /test/ break; # 停留在这一级
  }

  location ~ ^/last {
    # rewrite ^/last /test/ last;  # 重新建立新的连接
    rewrite ^/last /test/ redirect;
  }

  location /test/ {
    default_type application/json;
    return 200 '{"status":"success"}';
  }

  location ~ ^/baidu {
    # rewrite ^/baidu http://www.baidu.com permanent; # 永久重定向
    rewrite ^/baidu http://www.baidu.com redirect; # 临时重定向
  }
}

tail -f /var/log/nginx/log/host.access.log
curl -vL 118.126.111.144/last
curl -vL 118.126.111.144/baidu
临时重定向，关闭nginx再访问118.126.111.144/baidu会报错
永久重定向，关闭nginx再访问118.126.111.144/baidu还是会重定向到百度，客户端会永久保存
```

### Rewrite规则_rewrite规则场景
```
server {
  location / {
    rewrite ^/course-(\d+)-(\d+)-(\d+)\.html$ /course/$1/$2/course_$3.html break;

    if($http_user_agent ~* Chrome) {
      rewrite ^/nginx http://www.baidu.com/class/121.html redirect;
    }
    
    if(!-f $request_filename) {
      rewrite ^/(.*)$ http://www.baidu.com/$1 redirect;
    }
  }
}

访问118.126.111.144/course-1-1-1.html
访问118.126.111.144/nginx
```

### Rewrite规则_rewrite规则书写
- Rewrite规则优先级
执行server块的rewrite指令
执行location匹配
执行选定的location中的rewrite
- 优雅的Rewrite规则书写

### Nginx进阶高级模块_secure_link模块作用原理
- secure_link_module模块
一、制定并允许检查请求的链接的真实性以及保护资源免遭未经授权的访问
二、限制链接生效周期
- secure_link_module配置语法
```
secure_link expression;
secure_link_md5 expression;
```
![image.png](https://upload-images.jianshu.io/upload_images/1956963-f7f39db1b95c7baa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Nginx进阶高级模块_secure_link模块实现请求资源验证
```
server {
  root /opt/app/code;
  location / {
    secure_link $arg_md5,$arg_expires;
    secure_link_md5 "$secure_link_expires$uri lh";
  }
}

使用md5url.sh生成加密串，真实开发会在后台实现。
sh md5url.sh
```

### Nginx进阶高级模块_Geoip读取地域信息模块介绍
- http_geoip_module模块
基于IP地址匹配MaxMind GeoIP 二进制文件，读取IP所在地域信息。
yum install nginx-module-geoip
- http_geoip_module模块
一、区别国内外作HTTP访问规则
二、区别国内城市地域作HTTP访问规则

### Nginx进阶高级模块_Geoip读取地域信息场景展示
```
yum install nginx-module-geoip
cd /etc/nginx/modules/ # 查看安装

vi nginx.conf

load_module "modules/ngx_http_geoip_module.so";
load_module "modules/ngx_stream_geoip_module.so";

download_geoip.sh # 执行下载国家和城市IP地域文件
gunzip 解压刚才下载的地域文件
vi test_geo.conf

geoip_country /etc/nginx/geoip/GeoIP.dat;
geoip_city /etc/nginx/geoip/GeoLiteCity.dat;
server {
  location / {
    if($geoip_country_code != CN){
      return 403;
    }
    root /usr/share/nginx/html;
    index index.html index.htm;
  }

  location /myip {
    default_type text/plain;
    return 200 "$remote_addr $geoip_country_name $geoip_country_code $geoip_city";
  }
}

nginx -tc /etc/nginx/nginx.conf
nginx -s reload -c /etc/nginx/nginx.conf
访问http://118.126.111.144/myip测试

```

### 基于Nginx的HTTPS服务_HTTPS原理和作用
- 为什么需要HTTPS?
原因：HTTP不安全
1 . 传输数据被中间人盗用、信息泄露
2 . 数据内容劫持、篡改
  
- HTTPS协议的实现
对传输内容进行加密以及身份验证

- 对称加密和非对称加密

- HTTPS加密协议原理
SSL + 非对称加密 + 对称加密
- 中间人伪造客户端和服务端问题，HTTPS解决不了
客户端对数字证书进行CA校验
如果校验成功则利用公钥加密
如果校验失败则停止回话

### 基于Nginx的HTTPS服务_证书签名生成CA证书
- 生成密钥和CA证书
```
openssl version
nginx -V 查看是否含有--with-http_ssl_module

步骤一、生成key密钥
步骤二、生成证书签名请求文件（csr文件）
步骤三、生成证书签名文件（CA文件）

```

### 基于Nginx的HTTPS服务_证书签名生成和Nginx的HTTPS服务场景演示
```
步骤一、生成key密钥
openssl version
nginx -V
rpm -qa | grep open
mkdir ssl_key
cd ssl_key
openssl genrsa -idea -out jesonc.key 1024  # 生成key密钥

步骤二、生成证书签名请求文件（csr文件）
openssl req -new -key jesonc.key -out jesonc.csr

步骤三、生成证书签名文件（CA文件）
openssl x509 -req -days 3650 -in jesonc.csr -signc.key -out jesonc.crt

```
- Nginx的HTTPS语法配置
```
server {
  ssl on;
  ssl_certificate /etc/nginx/ssl_key/jesonc.crt;
  ssl_certificate_key /etc/nginx/ssl_key/jesonc.crt;
}

nginx -s stop -c /etc/nginx/nginx.conf # 停止nginx
nginx -c /etc/nginx/nginx.conf # 启动nginx
netstat -luntp | grep 443

访问https://118.126.111.144/jesonc.html测试
```

### 基于Nginx的HTTPS服务_实战场景配置苹果要求的openssl后台HTTPS服务
- 配置苹果要求的证书
a、服务器所有的连接使用TLS1.2以上版本（openssl 1.0.2）
b、HTTPS证书必须使用SHA256以上哈希算法签名
c、HTTPS证书必须使用RSA2048位或ECC256位以上公钥算法
d、使用前向加密技术
```
update_openssl.sh # 升级openssl版本
sh ./update_openssl.sh
openssl version
# 直接通过key生成crt文件
openssl req -days 36500 -x509 -sha256 -nodes -newkey rsa:2048 -keyout jesonc.key -out jesonc_apple.crt
```
```
vi nginx.conf 修改

server {
  ssl on;
  ssl_certificate /etc/nginx/ssl_key/jesonc_apple.crt;
  ssl_certificate_key /etc/nginx/ssl_key/jesonc_key;
}

nginx -tc /etc/nginx/nginx.conf
nginx -s reload -c /etc/nginx/nginx.conf
netstat -luntp | grep 443

访问https://118.126.111.144/jesonc.html测试
```

### 基于Nginx的HTTPS服务_HTTPS服务优化
方法一、激活keepalive长连接
方法二、设置ssl session缓存
```
server {
  keepalive_timeout 100;
  ssl on;
  ssl_session_cache shared:SSL:10m;
  ssl_session_timeout 10m;
  
  ssl_certificate /etc/nginx/ssl_key/jesonc_apple.crt;
  ssl_certificate_key /etc/nginx/ssl_key/jesonc_key;
  
}

nginx -tc /etc/nginx/nginx.conf
nginx -s reload -c /etc/nginx/nginx.conf
```

### Nginx与Lua的开发_Nginx与Lua特性与优势
- 什么是Lua
是一个简洁、轻量、可扩展的脚本语言。
- Nginx+Lua优势
充分的结合Nginx的并发处理epoll优势和Lua的轻量实现简单的功能和高并发的场景。

### Nginx与Lua的开发_Lua基础开发语法
```
yum install lua
lua 

运行
print("Hello,world")
vi test.lua
chmod a+rx ./test.lua
./test.lua

注释
-- 行注释
--[[  块注释  ]]-- 

布尔类型只有nil和false，别的都是true
lua中的变量如果没有特殊说明，全是全局变量

whild循环
sum = 0
num = 1
while num <= 100 do
  sum = sum + num
  num = num + 1
end
print("sum = ",sum)
lua没有++或是+=这样的操作

for循环语句
sum = 0
for i = 1,100 do
  sum = sum + i
end

if-else语句
if age == 40 and sex == "Male" then
  print("大于40男人")
elseif age > 60 and sex ~= "Female" then
  print(“非女人而且大于60”)
else 
  local age = io.read()
  print("Your age is " ..age)
end
~= 是不等于
字符串的拼接操作符 ..
io库的分别从stdin和stdout读写的read和write函数

```

### Nginx与Lua的开发_Nginx与Lua的开发环境
- Nginx+Lua环境
LuaJIT
ngx_devel_kit和lua-nginx-module
重新编译Nginx
可以参考 http://www.imooc.com/article/19597

### Nginx与Lua的开发_Nginx调用Lua的指令及Nginx的Luaapi接口
- Nginx调用Lua模块指令
Nginx的可插拔模块化加载执行，共11个处理阶段

|   |    |
| --------   | -----:   |
|  set_by_lua set_by_lua_file  |  设置nginx变量，可以实现复杂的赋值逻辑  |
|  access_by_lua access_by_lua_file  |  请求访问阶段处理，用于访问控制  |
|  content_by_lua content_by_lua_file  |  内容处理器，接收请求处理并输出响应 |

- Nginx Lua API

|   |    |
| --------   | -----:   |
|  ngx.var  |  nginx变量  |
|  nginx.req.get_headers  |  获取请求头  |
|  nginx.req.get_uri_args  |  获取url请求参数  |
|  nginx.redirect  |  重定向  |
|  nginx.print  |  输出响应内容体  |
|  nginx.say  |  通ngx.print，但是会最后输出一个换行符  |
|  nginx.header  |  输出响应头  |
|  ...  |  ...  |

### Nginx与Lua的开发_实战场景灰度发布
- 什么是灰度发布？
按照一定的关系区别，分部分的代码进行上线，使代码的发布能平滑过渡上线

- 灰度发布
用户的信息cookie等信息区别
根据用户的ip地址

### Nginx与Lua的开发_实战场景灰度发布场景演示
![image.png](https://upload-images.jianshu.io/upload_images/1956963-db818294e6cbb290.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
yum install memcached
启动8080和9090两个tomcat
sh catalina.sh start; tail -f ../logs/catalina.out
netstat -luntp

启动memcache
memcached -p11211 -u nobody -d
ps -aux | grep mem
netstat -luntp | grep 11211

修改nginx的default.conf -> dep.conf
server {
  location /hello {
    default_type 'text/plain';
    content_by_lua 'ngx.say("hello,lua")';
  }

  location /myip {
    default_type 'text/plain';
    content_by_lua '
    clientIP = ngx.req.get_headers()["x_forwarded_for"]
    ngx.say("IP:",clientIP)
    ';
  }
}

nginx -tc /etc/nginx/nginx.conf
nginx -s reload -c /etc/nginx/nginx.conf
访问118.126.111.144/hello
访问118.126.111.144/myip

灰度发布 dep.conf
server {
  location / {
    default_type 'text/html';
    content_by_lua_file /opt/app/lua/dep.lua;
    #add_after_body "$http_x_forwarded_for";
  }

  location @server {
    proxy_pass http://127.0.0.1:9090;
  }

  location @server_test {
    proxy_pass http://127.0.0.1:8080;
  }
}

dep.lua略

telnet 127.0.0.1 11211 手动修改memcache值
set 118.126.111.144 0 0 1
get 118.126.111.144

nginx -s reload -c /etc/nginx/nginx.conf

```

# 五、Nginx架构篇
### Nginx常见问题__多个server_name中虚拟主机读取的优先级
- 相同server_name多个虚拟主机优先级访问
```
server {
  listen 80;
  server_name testserver1 www.baidu.com
  location {
    ...
  }
}

server {
  listen 80;
  server_name testserver2 www.baidu.com
  location {
    ...
  }
}

nginx -tc /etc/nginx/nginx.conf # 出现警告
nginx -s reload -c /etc/nginx/nginx.conf # 出现警告
ps -aux | grep nginx # nginx启动成功
# 会优先读取最先读取的配置
 
```

### Nginx常见问题_多个location匹配的优先级
```
location匹配优先级
=   进行普通字符精确匹配，也就是完全匹配，优先级高
^~ 表示普通字符匹配，使用前缀匹配，优先级高
~\~*  表示执行一个正则匹配()，优先级低，~区分大小写，~*不区分大小写

server {
  
  location = /code1/ {
    rewrite ^(.*)$ /code1/index.html break;
  }

  location ~ /code.* {
    rewrite ^(.*)$ /code3/index.html break;
  }

  location ^~ /code {
    rewrite ^(.*)$ /code2/index.html break;
  }

}
```

### Nginx常见问题_try_files使用
- Nginx的try_files的使用
按顺序检查文件是否存在
```
location / {
  try_files $uri $uri/ /index.php
}
```
```
server {
  location / {
    root /opt/app/code/cache;
    try_files $uri @java_page; # 如果@uri有，直接返回，否则去找@java_page
  }

  location @java_page {
    proxy_pass http://127.0.0.1:9090;
  }
}
```

### Nginx常见问题_alias和root的使用区别
- Nginx的alias和root区别
```
location /request_path/image/ {
  root /local_path/image/;
}

网页访问 http://118.126.111.144/request_path/image/cat.png
实际上访问的是 /local_path/image/request_path/image/cat.png

location /request_path/image/ {
  alias /local_path/image/;
}

网页访问 http://118.126.111.144/request_path/image/cat.png
实际上访问的是 /local_path/image/cat.png

```

### Nginx常见问题_如何获取用户真实的ip信息
- 用什么样的方法传递用户的真实IP地址
remote_addr取不到
x_forward_for容易被篡改
最好的方式是：第一级代理约定设置set x_real_ip=$remote_addr
后端服务$x_real_ip=IP1可以取到

### Nginx常见问题_Nginx中常见错误码
- Nginx：413 Request Entity Too Large
修改用户上传文件限制client_max_body_size
- 502 bad gateway
后端服务无响应
- 504 Gateway Time-out
后端服务执行超时

### Nginx的性能优化_内容介绍及性能优化考虑
- 性能优化考虑点
 1 . 当前系统结构瓶颈：观察指标、压力测试
2 . 了解业务模式：接口业务类型、系统层次化结构
3 . 性能与安全

### Nginx的性能优化_ab压测工具
- ab接口压力测试工具
1 . 安装
yum install httpd-tools
2 . 使用
ab -n 2000 -c 2 http://127.0.0.1/jesonc.html
ab -n 2000 -c 10 http://127.0.0.1/jesonc.html
-n 总的请求数
-c 并发数
-k 是否开启长连接
注意观察qps：每秒有多少个请求：Requests per second

```
sleepjava.jsp
ab -n 2000 -c 2 http://127.0.0.1/sleepjava.jsp
curl -I http://127.0.0.1/sleepjava.jsp
sh catalina.sh start;tail -f ../logs/catalina.out
netstat -luntp | grep java # 查看是否服务都启动了
curl -I http://127.0.0.1/sleepjava.jsp
ab -n 20 -c 2 http://127.0.0.1/sleepjava.jsp



ab -n 2000 -c 2 http://127.0.0.1/jesonc.html
ab -n 2000 -c 10 http://127.0.0.1/jesonc.html
分别测试tomcat和nginx，查看qps，证明nginx处理静态资源很快

```

### Nginx的性能优化_系统与Nginx性能优化
- 网络
- **系统**
- **服务**
- 程序
- 数据库、底层服务

### Nginx的性能优化_文件句柄设置
- 文件句柄
linux\Unix一切皆文件，文件句柄就是一个索引
- 设置方式
系统全局修改、用户局部性修改、进程局部性修改

系统全局修改、用户局部性修改：vi /etc/security/limits.conf
```
添加
root soft nofile 65535
root hard nofile 65535
*    soft nofile 25535
*    hard nofile 25535
```
进程局部性修改：vi /etc/nginx/nginx.conf
```
添加
worker_rlimit_nofile 65535;
```

### Nginx的性能优化_CPU亲和配置
- CPU的亲和
把进程通常不会在处理器之间频繁迁移进程迁移的频率小，减少性能损耗
```
cat /proc/cpuinfo | grep "physical id"|sort|uniq|wc -l   #查看cpu个数
cat /proc/cpuinfo | grep "cpu cores"|uniq   #查看cpu核心数
top 按1展示出cpu核心数量已经进程使用率

如何绑定：vi /etc/nginx/nginx.conf

server {
  worker_processes 16;   # 表示当前启动多少个worker进程，需要和cpu核心一致
  # worker_cpu_affinity 好多0不抄了   # cpu亲和配置方式
  worker_cpu_affinity 1010101010101010 0101010101010101; # 配置2个work
  worker_cpu_affinity auto;   # nginx自动处理，开发可以使用这种
}

ps -eo pid,args,psr | grep [n]ginx   # 输出nginx进程的pid,args,psr
```

### Nginx的性能优化_Nginx通用配置优化
提供了一套配置，我觉得可以去github找找看通用配置

### Nginx安全_恶意行为控制手段
- 常见的恶意行为
爬虫行为和恶意抓取、资源盗用
基础防盗链功能-目的不让恶意用户能轻易的爬取网站对外数据
secure_link_module-对数据安全性提高加密验证和失效性，适合如核心重要数据
access_module-对后台、部分用户服务的数据提供IP防护

### Nginx安全_攻击手段之暴力破解
- 常见的攻击手段
后台密码撞库-通过猜测密码字典不断对后台系统登录性尝试，获取后台登录密码
方法一、后台登录密码复杂度
方法二、access_module-对后台提供IP防控
方法三、预警机制

### Nginx安全_文件上传漏洞
- 常见的攻击手段
文件上传漏洞-利用这些可以上传的接口将恶意代码移植到服务器中，再通过url去访问以执行代码
```
http://118.126.111.144/upload/1.jpg/1.php
Nginx将1.jpg作为php代码执行

location ^~ /upload{
  root /opt/app/images;
  if($request_filename ~* (.*)\.php){
    return 403;
  }
}
```
### Nginx安全_SQL注入
- 常见的攻击手段
SQL注入-利用未过滤 /未审核用户输入的攻击方法，让应用运行本不应该运行的SQL代码
- SQL注入场景
```
select * from users where username = '' or 1=1 #' and password =‘123455’
' or 1=1 #
```

### Nginx安全_Nginx+LUA防火墙功能
![image.png](https://upload-images.jianshu.io/upload_images/1956963-983e4b640192de32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Nginx安全_Nginx+LUA防火墙防sql注入场景演示 
https://github.com/loveshell/ngx_lua_waf

### Nginx安全_复杂的访问攻击中CC攻击方式
```
curl -I http://118.126.111.144/1.html
ab -n 2000 -c 200 http://118.126.111.144/1.html
curl -I http://118.126.111.144/1.html
```

### Nginx安全_Nginx版本更新和本身漏洞
- 查看版本更新描述

### Nginx架构总结_静态资源服务的功能设计
- 基于Nginx的中间件架构
1 . 了解需求
定义Nginx在服务体系中的角色：静态资源服务、代理服务、动静分离
静态资源服务的功能设计：类型分类、浏览器缓存、防盗链、流量限制、防盗链盗用、压缩
代理服务：协议类型、正向代理、反向代理、负载均衡、代理缓存、头信息处理、LNMP、分片请求、Poxypass
2 . 设计评估
硬件：CPU、内存、硬盘
系统：用户权限、日志目录存放
关联服务：LVS、keepalive、syslog、Fastcgi
3 . 配置注意事项
合理配置
了解原理
关注日志

# 附录
### [Nginx中文文档](https://www.baidu.com/link?url=wpkRGe-tPbxc-1voN7eWfbHPyb97svID9QvYu8EQA-lUhZKdzs0vlCGJ3UHLDds8&wd=&eqid=c9edb3d100023485000000055b6ab1de)
