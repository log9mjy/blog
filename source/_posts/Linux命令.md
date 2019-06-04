---
title: Linux命令
date: 2019-06-03 10:25:02
tags: [Linux,服务器]
---

```shell
 df -h
```

linux中df命令的功能是用来检查linux服务器的文件系统的磁盘空间占用情况。可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。

```shell
su  root
```

切到root账户 

```shell
du -sh *
```

查看当前每个文件夹大小

```shell
ps -ef |grep java 
```

搜索Java相关进程

```shell
lsof -i:{端口号}
```

查看端口占用

```shell 
netstat -tunlp
```

用于显示tcp，udp的端口和进程等相关情况

```shell
netstat -tunlp|grep 端口号
```

用于查看指定端口号的进程情况，如查看22端口的情况，netstat -tunlp|grep 22