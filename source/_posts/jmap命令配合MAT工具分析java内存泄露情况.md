---
title: jmap命令配合MAT工具分析java内存泄露情况
date: 2018-12-07 11:14:05
categories:
- Linux Learning
tags:
- Linux
- tools
---

今天测试同学压测的时候又反馈内存有0.4%个百分点的上浮，其实之前用top命令观察了一段时间没什么问题。如果存在内存泄露的问题内存使用率应该会持续增加，而不是压了一个晚上也可以保持稳定。 :sweat: 不过两台服务器上一个项目的进程内存占用的确有一点点差别，保险起见dump堆转储文件分析看看，也学习一下。

<!--more-->

## 一. 内存监控检视

目前系统在测试环境压测时内存检测如下图，这种比较稳定的一般不存在内存泄露的问题：

![uws](http://martintop-image.oss-cn-shenzhen.aliyuncs.com/18-12-7/93522148.jpg)

典型的可能存在内存泄露的监控图如下：

![内存泄漏](http://martintop-image.oss-cn-shenzhen.aliyuncs.com/18-12-7/223989.jpg)

## 二、jmap命令

jmap命令是jdk自带的小工具，可以打印分析java进程的共享对象内存映射、堆内存的细节等。

jmap命令可以获得运行中的jvm的堆的快照，从而可以：

* 离线分析堆，以检查内存泄漏，检查一些严重影响性能的大对象的创建；
* 检查系统中什么对象最多，各种对象所占内存的大小等等；
* 可以使用jmap生成Heap Dump文件后抓取到本地进一步分析。

>**什么是堆Dump**
>
>堆Dump是反应Java堆使用情况的内存镜像，其中主要包括系统信息、虚拟机属性、完整的线程Dump、所有类和对象的状态等。 一般，在内存不足、GC异常等情况下，我们就会怀疑有内存泄露。这个时候我们就可以制作堆Dump来查看具体情况。

### 1.命令概述

在shell中输入jmap有很完整的命令介绍：

```shell
[app@VM_101_178_centos irps-uws]$ jmap
Usage:
    jmap [option] <pid>
        (to connect to running process)
    jmap [option] <executable <core>
        (to connect to a core file)
    jmap [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    <none>               to print same info as Solaris pmap
    #打印jvm的heap情况
    -heap                to print java heap summary
    #打印jvm的heap直方图：类型、对象数量、占用大小等，:live-可选是打印存活对象
    -histo[:live]        to print histogram of java object heap; if the "live"
                         suboption is specified, only count live objects
    -clstats             to print class loader statistics
    #显示在 F-Queue 队列等待 Finalizer 线程执行 finalize 方法的对象 
    -finalizerinfo       to print information on objects awaiting finalization
    #输出jvm堆内存信息到文件
    -dump:<dump-options> to dump java heap in hprof binary format
                         dump-options:
                           live         dump only live objects; if not specified,
                                        all objects in the heap are dumped.
                           format=b     binary format	#dump命令可选参数，以二进制输出
                           file=<file>  dump heap to <file> #命名输出文件
                         Example: jmap -dump:live,format=b,file=heap.bin <pid>
    -F                   force. Use with -dump:<dump-options> <pid> or -histo
                         to force a heap dump or histogram when <pid> does not
                         respond. The "live" suboption is not supported
                         in this mode.
    -h | -help           to print this help message
    -J<flag>             to pass <flag> directly to the runtime system
```

### 2.详细说明

#### 2.1 jmap -heap pid

打印pid的整体堆栈信息

```shell
#一般先使用top命令查看内存占用情况
#然后按大写字母M，以mem占用排序pid，再使用jmp进一步分析
[app@VM_101_178_centos irps-uws]$ jmap -heap 65067
Attaching to process ID 65067, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.152-b16

using parallel threads in the new generation.#新生代采用的是并行线程处理方式
using thread-local object allocation.
Concurrent Mark-Sweep GC#同步并行垃圾回收方式

Heap Configuration:#堆配置情况
   MinHeapFreeRatio         = 40 #最小堆使用比例
   MaxHeapFreeRatio         = 70 #最大堆使用比例
   MaxHeapSize              = 2147483648 (2048.0MB) #最大堆空间大小
   NewSize                  = 805306368 (768.0MB)	#新生代分配大小
   MaxNewSize               = 805306368 (768.0MB)	#最大可新生代分配大小
   OldSize                  = 1342177280 (1280.0MB)	#老生代大小
   NewRatio                 = 2	#新生代比例
   SurvivorRatio            = 8	#新生代与suvivor的比例
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 134217728 (128.0MB)
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:#堆使用情况
New Generation (Eden + 1 Survivor Space):#新生代（伊甸区 + survior空间）
   capacity = 724828160 (691.25MB)
   used     = 283548752 (270.4131622314453MB)
   free     = 441279408 (420.8368377685547MB)
   39.11944480744236% used
Eden Space:#伊甸区
   capacity = 644349952 (614.5MB)
   used     = 283111232 (269.99591064453125MB)
   free     = 361238720 (344.50408935546875MB)
   43.937495629704024% used
From Space:#survior1区
   capacity = 80478208 (76.75MB)
   used     = 437520 (0.4172515869140625MB)
   free     = 80040688 (76.33274841308594MB)
   0.5436502761095277% used
To Space:#survior2 区
   capacity = 80478208 (76.75MB)
   used     = 0 (0.0MB)
   free     = 80478208 (76.75MB)
   0.0% used
concurrent mark-sweep generation:#老生代使用情况
   capacity = 1342177280 (1280.0MB)
   used     = 97188168 (92.68585968017578MB)
   free     = 1244989112 (1187.3141403198242MB)
   7.241082787513733% used

33675 interned Strings occupying 4006736 bytes.
```

#### 2.2 jmap -finalizerinfo pid

打印等候回收的对象的信息。

```shell
[app@VM_101_178_centos irps-uws]$ jmap -finalizerinfo 65067
Attaching to process ID 65067, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.152-b16
Number of objects pending for finalization: 0	#0个对象在等候被回收
```

#### 2.3 jmap -histo pid

打印每个class的实例数目,内存占用,类全名信息. VM的内部类名字开头会加上前缀`*`. 如果live子参数加上后,只统计活的对象数量. 

```shell
 #-histo命令会打印大量实例，最好和less配合使用
 [app@VM_101_178_centos irps-uws]$ jmap -histo 65067|less
 num     #instances         #bytes  class name
----------------------------------------------
   1:       1915748      208902184  [C
   2:        729935       86852936  [B
   3:        491930       51925552  [Ljava.lang.Object;
   4:        670703       32193744  java.util.HashMap
   5:        349710       30774480  java.lang.reflect.Method
   6:        139541       20728016  [I
   7:        242339       20377456  [Ljava.util.HashMap$Node;
   8:        800805       19219320  java.lang.String
   9:        551565       17650080  java.util.HashMap$Node
  10:        931898       16330024  [Ljava.lang.Class;
  11:        348377       13935080  java.util.HashMap$KeyIterator
  12:        175761       12654792  java.lang.reflect.Field
......
#instance 是对象的实例个数 
#bytes 是总占用的字节数 
#class name 对应的就是 Class 文件里的 class 的标识 
B 代表 byte
C 代表 char
D 代表 double
F 代表 float
I 代表 int
J 代表 long
Z 代表 boolean
前边有 [ 代表数组， [I 就相当于 int[]
对象用 [L + 类名表示

#补充常用指令配合使用
#输出到a.log文件中，可以隔段时间输出并对比观察gc回收的情况
jmap -histo:live pid>a.log 

#只统计存活对象情况，ps：这个命令执行，JVM会先触发gc，然后再统计信息。
jmap -histo:live pid 

#查看对象数最多的对象，并按降序排序输出：
jmap -histo <pid>|grep alibaba|sort -k 2 -g -r|less #alibaba为需要搜索的对象

#查看占用内存最多的最象，并按降序排序输出：
jmap -histo <pid>|grep alibaba|sort -k 3 -g -r|less
```

#### 2.4 jmap -dump:live,format=b,file=filename pid

jmap -dump命令用于生成堆转储文件，其中的live、format指令均可选。一般生成MAT分析的文件以`.hprof`的后缀结尾。

```shell
#输出堆转储文件到dumpfile.hprof
[app@VM_101_178_centos irps-uws]$ jmap -dump:live,format=b,file=dumpfile.hprof 65067
Dumping heap to /data/appsystems/irps-uws/dumpfile.hprof ...
```

> 这个命令执行，JVM会将整个heap的信息dump写入到一个文件，heap如果比较大的话，就会导致这个过程比较耗时，并且执行的过程中为了保证dump的信息是可靠的，所以会暂停应用。

## 三、利用MAT检查内存泄露

dump了文件到服务器上，使用scp或sz命令从Linux环境下载到本地环境。

### 1.MAT下载安装

* 将MAT当做eclipse的插件进行安装：启动Eclipse --> Help --> Eclipse Marketplace，然后搜索Memory Analyzer，安装，重启eclipse即可。

* 将MAT作为一个独立的软件进行安装：去官网<http://www.eclipse.org/mat/downloads.php>，根据操作系统版本下载最新的MAT。下载后解压就可以运行了。

推荐第二种方式哈 :cocktail:

### 2.MAT配置

MAT 软件版本解压后目录内有个MemoryAnalyzer.ini文件，该文件里面有个Xmx参数，该参数表示最大内存占用量，默认为1024m，根据堆转储文件大小修改该参数即可。
* MemoryAnalyzer.ini中的参数一般默认为-vmargs– Xmx1024m，这就够用了。假如你机器的内存不大，改大该参数的值，会导致MemoryAnalyzer启动时，报错:Failed to create the Java Virtual Machine。
* 当你导出的dump文件的大小大于你配置的1024m（说明1中，提到的配置：-vmargs– Xmx1024m），MAT输出分析报告的时候，会报错：An internal error occurred during: "Parsing heap dump from XXX”。适当调大说明1中的参数即可。

### 3.使用MAT分析堆转储文件

打开mat后，选择File -> Open Heap Dump找到下载到本地的dump文件打开。

* 加载后首页如下，主要关注Leak Suspects功能-泄露分析。

![MAT首页](http://martintop-image.oss-cn-shenzhen.aliyuncs.com/18-12-7/53428060.jpg)

* 点击Leak Suspects，.hprof文件所在文件夹 :file_folder: 会生成一个压缩包。如果需要和同事一起分析这个内存问题，只需要把这个小小的 zip 包发给他就可以了，不需要把整个堆文件发给他。并且整个报告是一个 HTML 格式的文件，用浏览器就可以打开。

![MAT ZIP](http://martintop-image.oss-cn-shenzhen.aliyuncs.com/18-12-7/58536573.jpg)

* 进入suspects页面如下，MAT工具会列出来可能存在内存泄露的位置。现在看唯一的可能泄露是jar包中的并不是程序代码导致，所以程序不存在内存泄露的问题。

![Leak Suspects](http://martintop-image.oss-cn-shenzhen.aliyuncs.com/18-12-7/37832446.jpg)

* 假如存在内存泄露我们可以点击Details进一步分析，参考网上的blog如下。这个截图其实就说明是系统代码的问题了。

![suspect](http://martintop-image.oss-cn-shenzhen.aliyuncs.com/18-12-7/40088292.jpg)

* 在详情页面**Shortest Paths To the Accumulation Point**表示GC root到内存消耗聚集点的最短路径，如果某个内存消耗聚集点有路径到达GC root，则该内存消耗聚集点不会被当做垃圾被回收。（所以重点关注哦）

![内存泄露~](http://martintop-image.oss-cn-shenzhen.aliyuncs.com/18-12-7/76005603.jpg)

* 在All Accumulated Objects by Class列举了该对象所存储的所有内容。其实从这个地方已经可以大概分析出问题了，EventInfo这个对象数量不太正常。

  ![EventInfo](http://martintop-image.oss-cn-shenzhen.aliyuncs.com/18-12-7/81909733.jpg)

* 为了更精确的定位，可以抓取间隔一段时间的两个dump文件进行对比分析，更加容易定位内存泄露的位置。

> MAT同时打开两个堆转储文件，分别打开Histogram，如下图。在下图中方框1按钮用于对比两个Histogram，对比后在方框2处选择Group By package，然后对比各对象的变化。不难发现heap3.hprof比heap6.hprof少了64个eventInfo对象

![compare dump file](http://martintop-image.oss-cn-shenzhen.aliyuncs.com/18-12-7/29004956.jpg)

* 最终分析结果如下：

![result](http://martintop-image.oss-cn-shenzhen.aliyuncs.com/18-12-7/97387390.jpg)

> 参考文章：
>
> [使用 Eclipse Memory Analyzer 进行堆转储文件分析](https://blog.csdn.net/wanghuiqi2008/article/details/50724676)
>
> [利用内存分析工具（Memory Analyzer Tool，MAT）分析java项目内存泄露](https://www.ibm.com/developerworks/cn/opensource/os-cn-ecl-ma/index.html)