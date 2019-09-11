---
title: Linux安装环境
date: 2019-07-02 19:39:38
tags: 
categories: Linux
---

##  安装JDK

1.将jdk-8u11-linux-x64.tar上传到Linux中,并且解压

2.配置环境变量

```
vim /etc/profile
```

最后写入

```
export JAVA_HOME=/usr/local/java/jdk1.8.0_11
export CLASSPATH=$JAVA_HOME/lib/
export PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH
```

3.生效环境变量

```
source /etc/profile
```

## 安装数据库

```
版本:mysql-5.6.42-linux-glibc2.12-x86_64.tar.gz
```

```
解压
tar zxvf mysql-5.6.33-linux-glibc2.5-x86_64.tar.gz

复制解压后的mysql目录到系统的本地软件目录
cp mysql-5.6.33-linux-glibc2.5-x86_64 /usr/local/mysql -r

添加系统mysql组和mysql用户
groupadd mysql
useradd -r -g mysql -s /bin/false mysql

进入安装mysql软件目录，修改目录拥有者为mysql用户
cd mysql/
chown -R mysql:mysql ./

安装数据库，此处可能出现错误。

./scripts/mysql_install_db --user=mysql

如果出现错误
    FATAL ERROR: please install the following Perl modules before executing scripts/mysql_install_db:
    Data::Dumper

yum install -y perl-Data-Dumper

如果出现错误
     Installing MySQL system tables..../bin/mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory
#解决方法：     
yum install libaio* -y
修改当前目录拥有者为root用户
chown -R root:root ./

修改当前data目录拥有者为mysql用户
chown -R mysql:mysql data

============== 到此数据库安装完毕 =============

添加mysql服务开机自启动

添加开机启动，把启动脚本放到开机初始化目录。
cp support-files/mysql.server /etc/init.d/mysql
# 赋予可执行权限
chmod +x /etc/init.d/mysql
# 添加服务
chkconfig --add mysql 
# 显示服务列表
chkconfig --list 
如果看到mysql的服务，并且3,4,5都是on的话则成功，如果是off，则执行
chkconfig --level 345 mysql on

启动mysql服务
#创建缺少的文件夹
mkdir /var/log/mariadb
service mysql start
正常提示信息：Starting MySQL. SUCCESS!

把mysql客户端放到默认路径
ln -s /usr/local/mysql/bin/mysql /usr/local/bin/mysql
注意：建议使用软链过去，不要直接包文件复制，便于系统安装多个版本的mysql

=================== 这是分割线 ==================

通过使用 mysql -uroot -p 连接数据库（默认数据库的root用户没有密码，这个需要设置一个密码）。
mysql安装目录support_file下的my-default文件 在里面找到 [mysqld] 这一项，然后在该配置项下添加 skip-grant-tables
#重启 
service mysql restart
免密登录MySQL
敲入 mysql -u root -p 命令然后回车，当需要输入密码时，直接按enter键，便可以不用密码登录到数据库当中
使用 set password for 'root'@'localhost' = password('密码') 命令修改新的密码。
--------------------- 
错误信息：ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)

解决方法：打开/etc/my.cnf,看看里面配置的socket位置是什么目录。“socket=/var/lib/mysql/mysql.sock”

路径和“/tmp/mysql.sock”不一致。建立一个软连接：ln -s /var/lib/mysql/mysql.sock /tmp/mysql.sock

 到这里任务算是完成了。之后就可以创建数据库用户，然后使用数据库了。

###################### 分割线 ######################

权限控制

1、去除匿名用户

# 测试匿名用户登录
mysql -ux3

可以看到匿名用户可以登录，具有information_schema和test库的相关权限。

# 删除匿名用户，使用root用户登录数据库
delete from mysql.user where User='';
flush privileges;

再次测试匿名用户登录

 
```

  

## Linux安装Redis

https://www.cnblogs.com/wangchunniu1314/p/6339416.html

## 安装elasticsearch

1. 去官网下载ES（下载地址：<https://www.elastic.co/cn/downloads/elasticsearch）
2. 上传并且解压

3. vim config/elasticsearch.yml

```
# 节点名称
cluster.name: elasticsearch

# 数据存储位置
path.data: /usr/local/elasticserch-6.3.0/data

# 日志文件的路径
path.logs: /usr/local/elasticserch-6.3.0/logs

#绑定监听IP
 network.host: 192.168.192.128(阿里云私有IP非公网IP)    

# 设置节点间交互的tcp端口,默认是9300
 transport.tcp.port: 9300  
 
# 设置对外服务的http端口,默认为9200
 http.port: 9200 
```

4. 修改(不修改会报错)

```
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
 切换到root用户，编辑limits.conf添加如下内容
 `vi /etc/security/limits.conf`
 `* soft nofile 65536`
 `* hard nofile 65536
```

```
[3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
 修改/etc/sysctl.conf文件，增加配置vm.max_map_count=262144
 `vi /etc/sysctl.conf`
 vm.max_map_count=262144
 `sysctl -p`
 执行命令sysctl -p生效
```

5.启动

```
ES不能用root用户启动，否则会启动报错

①添加es用户   

adduser es

②给es用户赋予操作文件夹的权限

chown -R es:es /usr/local/elasticserch-6.3.0

③切换到es用户，启动es

su es

./elasticsearch

后台启动 ./elasticsearch -d
```

6.测试

访问:<http://39.106.156.244:9200/>

7.安装kibana

```
在es官网下载

vim config/kibana.yml

server.port: 5601

server.host: "192.168.192.128"

elasticsearch.url: "http://192.168.192.128:9200"

启动

./kibana
```

## 安装fastDFS

```
1.下载安装libfastcommon
git clone https://github.com/happyfish100/libfastcommon.git
cd libfastcommon/
./make.sh
./make.sh install
2.下载安装fastdfs
wget https://github.com/happyfish100/fastdfs/archive/V5.05.tar.gz
tar -zxvf V5.05.tar.gz
cd V5.05
./make.sh
./make.sh install
执行安装后，默认会安装到/usr/bin中，并在/etc/fdfs中添加三个配置文件。
3.修改配置文件
将/etc/fdfs中三个文件的名字去掉sample.
tracker.conf 中修改：
base_path=/usr/lgip_fastdfs/fastdfs-tracker-log   #用于存放日志
storage.conf 中修改：
base_path=/usr/lgip_fastdfs/fastdfs-storage-log   #用于存放日志
store_path0=/usr/lgip_fastdfs/fastdfs-file-save   #存放数据
tracker_server=192.168.20.35:22122                #指定tracker服务器地址
client.conf 中同样要修改：
base_path=/usr/lgip_fastdfs/fastdfs-client-log    #用于存放日志。
tracker_server=192.168.20.35:22122                #指定tracker服务器地址
注：以上的base_path、store_path0的路径均需要进行手动创建。
4.启动tracker和storage
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf  
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf 
5.重启tracker和storage
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf  restart
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf  restart
6.停止tracker和storage
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf  stop
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf  stop
7.启动成功，测试服务是否正常运行：
上传：/usr/bin/fdfs_upload_file /etc/fdfs/client.conf /usr/01.jpg
下载：/usr/bin/fdfs_download_file /etc/fdfs/client.conf group1/M00/00/00/eSosZVfrMy2ADEcxAADS9IecoKQ527.jpg /usr/02.jpg
删除：/usr/bin/fdfs_delete_file /etc/fdfs/client.conf group1/M00/00/00/eSosZVfrLr6AfbmDAADS9IecoKQ093.jpg
8.在所有storage节点安装fastdfs-nginx-module
tar -xzvf fastdfs-nginx-module_v1.16.tar.gz
修改 fastdfs-nginx-module 的 config 配置文件
cd fastdfs-nginx-module/src
 vi config
将
CORE_INCS="$CORE_INCS /usr/local/include/fastdfs /usr/local/include/fastcommon/" 
修改为:
CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
9.安装Nginx
tar -zxvf nginx-1.10.0.tar.gz
./configure --add-module=/usr/local/src/fastdfs-nginx-module/src &&make && make install

该出的add-module为fastdfs-nginx-module路径
10.复制fastdfs-nginx-module下的mod_fastdfs.conf到/etc/fdfs/
cp /usr/local/src/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/
修改配置文件
vi /etc/fdfs/mod_fastdfs.conf
     connect_timeout=10
     base_path=/tmp
     tracker_server=ip01:22122
     storage_server_port=23000
     group_name=group1
     url_have_group_name = true
     store_path0=/fastdfs/storage     //为上传数据的的路径,storage.conf 中的store_path0的路径
11.复制 FastDFS 的部分配置文件到/etc/fdfs 目录
cd /usr/local/src/FastDFS/conf
cp http.conf mime.types /etc/fdfs/
12.配置nginx
user root

worker_processes 1;

events {

    worker_connections 1024;

}

http {

    include mime.types;

    default_type application/octet-stream;

    sendfile on;

    keepalive_timeout 65;

    server {

        listen 8888;

        server_name localhost;

        location ~/group([0-9])/M00 {

            ngx_fastdfs_module;

        }

        error_page 500 502 503 504 /50x.html;

 

        location = /50x.html {

            root html;

        }

    }

}
注意:
 A、8888 端口值是要与/etc/fdfs/storage.conf 中的 http.server_port=8888 相对应, 因为 http.server_port 默认为 8888,如果想改成 80,则要对应修改过来。
 B、Storage 对应有多个 group 的情况下,访问路径带 group 名,如/group1/M00/00/00/xxx, 对应的 Nginx 配置为:
     location ~/group([0-9])/M00 {
         ngx_fastdfs_module;
      }
成功后:http://47.110.73.123:8888/group1/M00/00/00/rBCMX1v09E6AXJorAAB-mPrAqvM742.jpg
```

## 安装nginx

```
安装依赖
yum install gcc
yum install pcre-devel
yum install zlib zlib-devel
yum install openssl openssl-devel
//一键安装上面四个依赖
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
下载nginx的tar包
复制代码
//创建一个文件夹
cd /usr/local
mkdir nginx
cd nginx
//下载tar包
wget http://nginx.org/download/nginx-1.13.7.tar.gz
tar -xvf nginx-1.13.7.tar.gz
复制代码
安装nginx
//进入nginx目录
cd /usr/local/nginx
//执行命令
./configure
//执行make命令
make
//执行make install命令
make install
Nginx常用命令
//测试配置文件
安装路径下的/nginx/sbin/nginx -t
复制代码
//启动命令
安装路径下的/nginx/sbin/nginx
//停止命令
安装路径下的/nginx/sbin/nginx -s stop
或者 : nginx -s quit
//重启命令
安装路径下的/nginx/sbin/nginx -s reload
复制代码
//查看进程命令
ps -ef | grep nginx
//平滑重启
kill -HUP Nginx主进程号
```

