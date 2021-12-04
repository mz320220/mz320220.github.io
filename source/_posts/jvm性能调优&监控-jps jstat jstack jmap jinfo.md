---
title: jvm性能调优&监控-jps jstat jstack jmap jinfo 
date: 2018-12-11 16:14:05
categories:
- Linux Learning
tags:
- Linux
---

![](http://martintop-image.oss-cn-shenzhen.aliyuncs.com/18-12-11/28660764.jpg)

上一份工作其实几乎不怎么接触这些，生产环境的权限控制严格，大部分是由运维来做的。这段时间看了一些jvm性能调优、内存异常监控排查的blog，大部分也是东抄西抄，是时候系统的小结一波，大概归纳下jdk自带的常用命令，本文只简单介绍各个命令基本语法和最常用的功能，详细内容再单独开篇学习吧。

ps：除了标题列举的命令之外其实还有jhat、jconsole等命令，只不过jhat对比mat工具实在不够直观，jconsole图形化界面在服务器上也不一定能用。

<!--more-->

## 1.jps

jps=JVM Process Status Tool，用于显示当前系统(未指定hostid时)内HotSpot虚拟机中的进程情况。常用命令：

* jps -m：输出传递给main方法的参数。

* jps -v：输出传递给JVM的参数。举例如下：

```shell
[app@VM_104_20_centos irps-chs]$ jps -m
216635 JMap -histo 125676
81530 JMap -dump:format=b,file=/tmp/irps-chs.hprof
215688 Jps -m
216765 JConsole
[app@VM_104_20_centos irps-chs]$ jps -v
215744 Jps -Dapplication.home=/nemo/jdk1.8.0_152 -Xms8m
216635 JMap -Dapplication.home=/nemo/jdk1.8.0_152 -Xms8m -Dsun.jvm.hotspot.debugger.useProcDebugger -Dsun.jvm.hotspot.debugger.useWindbgDebugger
81530 JMap -Dapplication.home=/nemo/jdk1.8.0_152 -Xms8m -Dsun.jvm.hotspot.debugger.useProcDebugger -Dsun.jvm.hotspot.debugger.useWindbgDebugger
```

Oracle官方文档：[jps说明](https://docs.oracle.com/javase/7/docs/technotes/tools/share/jps.html)

## 2.jstat

jstat=JVM statistics Monitoring，用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。注意使用时需要加上进程id、所选参数。

语法结构为：

```shell
jstat [option] LVMID [interval] [count]
#option:可选操作参数
#LVMID：本地虚拟机进程ID
#interval：输出间隔，默认单位ms
#count：输出次数
```

常用命令：

* jstat -class LVMID：监视类装载数量、消耗空间，时间等。
* jstat -compiler LVMID：输入JIT编译的方法数量、失败数量、耗时、编译失败类型及方法等。
* jstat -gcutil LVMID interval count：统计程序gc情况(已用空间占总空间百分比)

```shell
[app@VM_104_20_centos irps-chs]$ jstat -class 125676
Loaded  Bytes  Unloaded  Bytes     Time   
 11825 22520.2        0     0.0    2509.31
 
[app@VM_104_20_centos irps-chs]$ jstat -compiler 125676
Compiled Failed Invalid   Time   FailedType FailedMethod
   14442      4       0   103.53          1 com/alibaba/druid/filter/FilterChainImpl connection_prepareStatement
   
[app@VM_104_20_centos irps-chs]$ jstat -gcutil 125676 3000 2 #每3s输出，一共输出两次
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
  0.37   0.00   7.33  45.04  98.17  96.25  11884  110.109     4    0.382  110.492
  0.37   0.00   7.33  45.04  98.17  96.25  11884  110.109     4    0.382  110.492
```

Oracle官方文档：[jstat说明](https://docs.oracle.com/javase/7/docs/technotes/tools/share/jstat.html)


## 3.jstack

jstack用于生成java虚拟机当前时刻的线程快照，即当前java虚拟机内**所有线程**正在执行的方法堆栈的集合。

所以jstack可以用于观察jvm中当前所有线程的运行情况和线程当前状态，定位线程出现长时间停顿的原因，如**线程间死锁、死循环、请求外部资源导致的长时间等待**等。 

> 系统崩溃了？如果java程序崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack的信息，从而可以轻松地知道java程序是如何崩溃和在程序何处发生问题。
>
> 系统hung住了？jstack工具还可以附属到正在运行的java程序中，看到当时运行的java程序的java stack和native stack的信息, 如果现在运行的java程序呈现hung的状态。

语法格式及说明：

```shell
jstack [option] LVMID
#option可选参数有：
-F : 当正常输出请求不被响应时，强制输出线程堆栈
-l : 除堆栈外，显示关于锁的附加信息
-m : 如果调用到本地方法的话，可以显示C/C++的堆栈
```

Oracle官方文档：[jstack说明](https://docs.oracle.com/javase/7/docs/technotes/tools/share/jstack.html)

## 4.jmap

jmap=JVM Memory Map，用于生成dump文件、查询finalize执行队列、Java堆和永久代的详细信息，如当前使用率、当前使用的是哪种收集器等。

语法格式如下：

```shell
jmap [option] LVMID
#option参数：
-dump : 生成堆转储快照
-finalizerinfo : 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
-heap : 显示Java堆详细信息
-histo : 显示堆中对象的统计信息
-permstat : to print permanent generation statistics
-F : 当-dump没有响应时，强制生成dump快照
```

Oracle官方文档：[jmap说明](https://docs.oracle.com/javase/7/docs/technotes/tools/share/jmap.html)

## 5.jinfo

jinfo=Java Configuration Info，相比于其他命令jinfo用法比较简单，主要用于实时查看或调整当前JVM配置的系统属性。常用命令如下：

* jinfo pid：打印指定process id的系统属性、jvm配置
* jinfo -flag [MaxHeapSize] pid：查看pid具体的参数值。举例如下：

```shell
[app@VM_104_20_centos irps-chs]$ jinfo 62316
Attaching to process ID 62316, please wait...
Debugger attached successfully.
#先展示系统相关属性
Server compiler detected.
JVM version is 25.152-b16
Java System Properties:

java.vendor = Oracle Corporation
sun.java.launcher = SUN_STANDARD
catalina.base = /irps-chs
......
#然后是jvm相关参数
VM Flags:
Non-default VM flags: -XX:+AlwaysPreTouch -XX:AutoBoxCacheMax=20000 -XX:CICompilerCount=3 -XX:CMSInitiatingOccupancyFraction=75 -XX:ErrorFile=null -XX:+ExplicitGCInvokesConcurrent
......
#指定查看MaxHeapSize大小
[app@VM_104_20_centos irps-chs]$ jinfo -flag MaxHeapSize  62316
-XX:MaxHeapSize=2147483648
```

> jinfo命令也可以实时配置一些jvm参数，使用的不太多，具体可参考：
>
> [jinfo调整jvm参数](https://www.jianshu.com/p/c321d0808a1b)