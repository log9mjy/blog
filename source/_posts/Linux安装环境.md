---
title: Linux安装环境
date: 2019-07-02 19:39:38
tags: 安装环境
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

