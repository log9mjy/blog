---
title: Java虚拟机调优
date: 2019-05-21 12:10:40
tags: JVM
---

## 1.jvm运行参数

### 标准参数

- -help
  -version

### -x参数

```
java  -X           ----查看-X所有参数

-Xmixed 混合模式执行 (默认)
-Xint 仅解释模式执行
-Xbootclasspath:<用 : 分隔的目录和 zip/jar 文件>
设置搜索路径以引导类和资源
-Xbootclasspath/a:<用 : 分隔的目录和 zip/jar 文件>
附加在引导类路径末尾
-Xbootclasspath/p:<用 : 分隔的目录和 zip/jar 文件>
置于引导类路径之前
-Xdiag 显示附加诊断消息
-Xnoclassgc 禁用类垃圾收集
-Xincgc 启用增量垃圾收集
-Xloggc:<file> 将 GC 状态记录在文件中 (带时间戳)
-Xbatch 禁用后台编译
-Xms<size> 设置初始 Java 堆大小
-Xmx<size> 设置最大 Java 堆大小
-Xss<size> 设置 Java 线程堆栈大小
-Xprof 输出 cpu 配置文件数据
-Xfuture 启用最严格的检查, 预期将来的默认值
-Xrs 减少 Java/VM 对操作系统信号的使用 (请参阅-Xcheck:jni 对 JNI 函数执行其他检查
-Xshare:off 不尝试使用共享类数据
-Xshare:auto 在可能的情况下使用共享类数据 (默认)
-Xshare:on 要求使用共享类数据, 否则将失败。
-XshowSettings 显示所有设置并继续
-XshowSettings:all
显示所有设置并继续
-XshowSettings:vm 显示所有与 vm 相关的设置并继续
-XshowSettings:properties
显示所有属性设置并继续
-XshowSettings:locale
显示所有与区域设置相关的设置并继续
```

常用：

-Xms<size> 设置初始 Java 堆大小
-Xmx<size> 设置最大 Java 堆大小
-Xss<size> 设置 Java 线程堆栈大小

### -xx参数

-XX参数也是非标准参数，主要用于jvm的调优和debug操作。

```
-Xmx2048m：等价于-XX:MaxHeapSize，设置JVM最大堆内存为2048M。 

-Xms512m：等价于-XX:InitialHeapSize，设置JVM初始堆内存为512M。 

-XX:+PrintFlagsFinal ： 打印参数

```

## 2.查看参数

1.查看所有的参数，用法：jinfo -flags <进程id> 

2.通过jps 或者 jps -l 查看java进程

3.查看class加载统计

```
jstat -class 6219

Loaded Bytes Unloaded Bytes Time 

3273   7122.3  0       0.0   3.98 
```

```
Loaded：加载class的数量 

Bytes：所占用空间大小 

Unloaded：未加载数量 

Bytes：未加载占用空间 

Time：时间
```

4.**垃圾回收统计**

```
[root@node01 ~]# jstat -gc 6219 

S0C S1C S0U S1U EC EU OC OU MC MU CCSC 

CCSU YGC YGCT FGC FGCT GCT 

9216.0 8704.0 0.0 6127.3 62976.0 3560.4 33792.0 20434.9 23808.0 23196.1 

2560.0 2361.6 7 1.078 1 0.244 1.323 
```

```
说明：
S0C：第一个Survivor区的大小（KB）
S1C：第二个Survivor区的大小（KB）
S0U：第一个Survivor区的使用大小（KB）
S1U：第二个Survivor区的使用大小（KB）
EC：Eden区的大小（KB）
EU：Eden区的使用大小（KB）
OC：Old区大小（KB）
OU：Old使用大小（KB）
MC：方法区大小（KB）
MU：方法区使用大小（KB）
CCSC：压缩类空间大小（KB）
CCSU：压缩类空间使用大小（KB）
YGC：年轻代垃圾回收次数
YGCT：年轻代垃圾回收消耗时间
FGC：老年代垃圾回收次数
FGCT：老年代垃圾回收消耗时间
GCT：垃圾回收消耗总时间
```

## 3.jmap的使用

### 3.1 查看内存使用情况

```
[root@node01 ~]# jmap -heap 6219
Attaching to process ID 6219, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.141-b15
using thread-local object allocation.
Parallel GC with 2 thread(s)
Heap Configuration: #堆内存配置信息
MinHeapFreeRatio = 0
MaxHeapFreeRatio = 100
MaxHeapSize = 488636416 (466.0MB)
NewSize = 10485760 (10.0MB)
MaxNewSize = 162529280 (155.0MB)
OldSize = 20971520 (20.0MB)
NewRatio = 2
SurvivorRatio = 8
MetaspaceSize = 21807104 (20.796875MB)
CompressedClassSpaceSize = 1073741824 (1024.0MB)
MaxMetaspaceSize = 17592186044415 MB
G1HeapRegionSize = 0 (0.0MB)
Heap Usage: # 堆内存的使用情况
PS Young Generation #年轻代
Eden Space:
capacity = 123731968 (118.0MB)
used = 1384736 (1.320587158203125MB)
free = 122347232 (116.67941284179688MB)
1.1191416594941737% used
From Space:
capacity = 9437184 (9.0MB)
used = 0 (0.0MB)
free = 9437184 (9.0MB)
0.0% used
To Space:
capacity = 9437184 (9.0MB)
used = 0 (0.0MB)
free = 9437184 (9.0MB)
0.0% used
PS Old Generation #年老代
capacity = 28311552 (27.0MB)
used = 13698672 (13.064071655273438MB)
free = 14612880 (13.935928344726562MB)
48.38545057508681% used
13648 interned Strings occupying 1866368 bytes.
```

### 3.2查看内存中对象数量及大小

```
jmap -histo:live 6219 | more
```

### 3.3将内存使用情况dump到文件中

```
jmap -dump:format=b,file=dumpFileName <pid>
```

会生成dump文件

### 3.4通过jhat对dump文件进行分析

```
jhat -port 9999 /tmp/dump.dat
```

打开浏览器进行访问：http://192.168.40.133:9999/

使用Java的jvmvisual打开dump文件，来分析

