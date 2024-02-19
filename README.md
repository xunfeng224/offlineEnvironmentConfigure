# offlineEnvironmentConfigure
linux离线配置基于nacos的微服务环境：jdk1.8，redis-5.0.5，MySQL5.7.43，Nacos1.4.6，nginx 1.24.0

# 环境配置

# jdk1.8

官网下载地址：https://www.oracle.com/java/technologies/downloads/#java8

## 传到linux上指定目录

```java
## /usr/local/jdk
$ tar zxvf jdk-8u141-linux-x64.tar.gz
```

## 移动到指定位置

```java
#**移动**
$ mv jdk1.8.0_141 /usr/local/
```

## 备份系统环境变量

```java
$cp /etc/profile /home/ctl/
```

## 编辑系统环境变量

```java
##编辑
$vi /etc/profile

#加入如下内容
export JAVA_HOME=/usr/local/jdk1.8.0_401
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH

#重新加载系统变量
$source /etc/profile
```

# Redis离线安装

官网下载地址：https://download.redis.io/releases/

```java
tar zxvf redis-5.0.5.tar.gz
yum install gcc-c++
mv redis-5.0.5 /usr/local/
cd /usr/local/redis-5.0.5/
make
cd src/
make install
mkdir /build/data/redis-5.0.5
cd /usr/local/redis-5.0.5/
mv redis.conf /build/data/redis-5.0.5/
vi /build/data/redis-5.0.5/redis.conf
#根据是否开放远程修改以下几项
# 开启远程注释bind
# bind 127.0.0.1
# 开启远程改为no
# protected-mode no
# 后台启动
# daemonize yes
# 设置密码
# requirepass Hithium@Dev2024

cd /usr/local/redis-5.0.5/
src/redis-server /build/data/redis-5.0.5/redis.conf
```

# MySQL5.7.43离线安装

官网下载地址：https://downloads.mysql.com/archives/community/

```java
tar zxvf mysql-5.7.43-linux-glibc2.12-x86_64.tar.gz
mv mysql-5.7.43-linux-glibc2.12-x86_64 /usr/local/mysql
cd /usr/local/mysql/
mkdir data
chmod -R 777 /usr/local/mysql/data/

#创建用户 、组、并将用户加入组,修改配置文件
groupadd mysql
useradd -g mysql mysql
vi /etc/my.cnf
```

my.cnf

```java
[mysqld]
bind-address=0.0.0.0
port=3306
user=mysql
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
socket=/tmp/mysql.sock
log-error=/usr/local/mysql/data/mysql.err
pid-file=/usr/local/mysql/data/mysql.pid
#character config
character_set_server=utf8mb4
symbolic-links=0
explicit_defaults_for_timestamp=true
cd /usr/local/mysql/bin
./mysqld --initialize --user=mysql --datadir=/usr/local/mysql/data/ --basedir=/usr/local/mysql/
```

若报错: ./mysqld: error while loading shared libraries: libnuma.so.1: cannot open shared object file: No such file or directory 安装依赖  libaio

```java
yum -y install libaio-devel.x86_64
yum -y install numactl
```

查看mysql初始化密码,并记录

```java
cat /usr/local/mysql/data/mysql.err
#添加软连接，并重启mysql服务
ln -s /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql
service mysql start
#
mysql -uroot -p
#输入记录的密码

#重设密码
set password=password('Hithium@Dev2024');
flush privileges;
#修改访问权限
use mysql;
update user set Host='%' where User='root';
flush privileges;
```

# Nacos1.4.6

下载地址https://github.com/alibaba/nacos/releases/tag/1.4.0

```java
tar zxvf nacos-server-1.4.6.tar.gz
mv nacos /usr/local/
cd /usr/local/nacos/conf
```

以此nacos-mysql.sql文件在MySQL数据库上创建nacos数据库:

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/897f2e49-1721-4cfd-9030-4fc81c71a372/5ecd813d-1664-4e99-9d5e-4a3eaa1be951/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/897f2e49-1721-4cfd-9030-4fc81c71a372/38df4a8a-2e85-4a6c-a842-12daa18d7430/Untitled.png)

修改application.properties文件,配置nacos数据库连接参数

启动nacos

```bash
cd /usr/local/nacos/bin/
./startup.sh -m standalone
# 或sh startup.sh -m standalone
# 查看日志
tail -f /usr/local/nacos/logs/start.out
```

# nginx 1.24.0

nginx官网下载地址：https://nginx.org/en/download.html

pcre：https://sourceforge.net/projects/pcre/files/pcre/8.41/

zlib：http://www.zlib.net/

oppenssl：https://www.openssl.org/source/

```bash
#/home/[username]
#上传服务器并解压
tar -zxvf nginx-1.24.0.tar.gz
tar -zxvf openssl-3.2.0.tar.gz
tar -zxvf pcre-8.41.tar.gz
tar -zxvf zlib-1.3.1.tar.gz
#进入解压后的nginx目录
cd nginx-1.24.0
#编译nginx的安装目录 默认/usr/local/nginx
./configure  --prefix=/usr/local/nginx  --with-pcre=../pcre-8.41   --with-openssl=../openssl-3.2.0  --with-zlib=../zlib-1.3.1
#安装nginx
make && make install
#配置文件路径/usr/local/nginx/conf/nginx.conf
#启动nginx
cd /usr/local/nginx/sbin
./nginx
#停止nginx 
./nginx  -s stop
#重载配置文件 
./nginx –s reload
#查看启动情况
ps -ef | grep nginx
```

nginx.conf

```
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;
	#允许跨域请求的域，* 代表所有
	add_header 'Access-Control-Allow-Origin' *;
	#允许请求的header
	add_header 'Access-Control-Allow-Headers' *;
	#允许带上cookie请求
	add_header 'Access-Control-Allow-Credentials' 'true';
	#允许请求的方法，比如 GET,POST,PUT,DELETE
	add_header 'Access-Control-Allow-Methods' *;

        location ^~/apis/ {
            proxy_pass <http://172.26.3.19:8080/>;
        }
        
				location / {
            root   index;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
   
}
```

# firewall端口操作

- 查看想开的端口是否已开：firewall-cmd --query-port=6379/tcp 
- 添加指定需要开放的端口：firewall-cmd --add-port=123/tcp --permanent 
- 重载入添加的端口：firewall-cmd --reload 
- 查询指定端口是否开启成功：firewall-cmd --query-port=123/tcp 
- 关闭端口：firewall-cmd --remove-port=80/tcp --permanent 
- firewall-cmd --reload

```bash
firewall-cmd --add-port=3306/tcp --permanent
firewall-cmd --add-port=6379/tcp --permanent
firewall-cmd --add-port=8848/tcp --permanent
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --add-port=80/tcp --permanent
#后台启动
nohup java -jar hithium-gateway-platform.jar >gateway.log &
nohup java -jar hithium-system-auth.jar >auth.log &
nohup java -jar hithium-system-platform.jar >platform.log &
#远程debug
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005
```
