---
title: 如何阅读JDK源码
date: 2019-04-17 15:56:05
categories:
- Java 
---

从去年年中跳槽几次面试就已经意识到了阅读JDK源码的重要性。从最基础的List，Map衍生开去，各种数据结构的相关特性，是否线程安全？如何进行扩容？不同业务场景下选用何种数据类型？以及有的面试官会问String类型的哈希值，以及计算原理。。。(略坑)。不仅仅是JDK源码，学习Spring、Netty、Dubbo的源码对于个人提升都有好处。对于我来说克服畏难情绪固然重要，好的方法也是可以坚持下去的重要因素。

<!--more-->

## How to

开始前的准备工作~

### 搜索网上资料

阅读一个框架的源码，最基础的操作肯定是扫一遍网上的各种分析文章。通过这个操作，你可以对这个框架有个大致的了解。站在巨人的肩膀上，少走许多弯路。但如果你看的项目是公司内部的框架，那么你只能找公司内部的文档了，更甚者，有些连文档都没有。那么你可以略过这一步。

### 扫一遍源码

当你拿到框架的源码的时候，你可以大致把源码的每个包，以及每个包下面的文件扫读一遍。扫读并不需要你弄清楚每一行代码的意思，只需要让你知道源码每一部分的作用。

如果一个开源框架足够标准，那么他的命名是非常语义化的。所以我们扫读的时候，通过包名、文件名就可以判断出这个包是用来干嘛的。例如 util 包是工具类，那我们可以直接跳过。vo 包是存放实体模型的，同样可以跳过。protocol 包是存放协议相关的等等。通过这么一个步骤，你会对整个项目有一个基本的印象，知道这个项目大概有哪些东西，哪些相对比较重要。

### 找到入口

阅读任何一个框架的源码，首先就是要找到框架的入口。通过上面扫读源码，你应该能够发现一些入口的迹象，例如对于 Dubbo 来说，你会发现它有一个名为 dubbo-demo 的子模块，那么我们肯定重点看它。进一步发掘需求你会发现它的入口就是 dubbo-demo 中的 Provider 类、Consumer 类。我们可以直接接运行这两个类的 main 方法，并一步步跟踪代码的执行情况。

### 通读源码

找到入口之后，下一步就是通读所有源码了，就是把源码的每个文件每一行都看一遍。在这个阶段不求完全弄懂细致的业务逻辑，但是要形成一个大概的框架，知道这个框架是如何设计的，有哪些大致的模块，这些模块是如何设计的。

> 在通读源码的过程中最容易出现倦怠的情绪，通过系统给的统计和记录我觉得是一个比较有作用的方法。
>
> 利用idea代码统计插件-Statistics，对需要阅读的代码进行整理统计，使用EXCEL进行记录。

 ![统计](https://martintop-image.oss-cn-shenzhen.aliyuncs.com/19-4-24/20190424180707.png)

 ![总计](https://martintop-image.oss-cn-shenzhen.aliyuncs.com/19-4-24/20190424180729.png)

### 梳理框架

在通读源码的过程中，你就会对框架有许多新的认识，会知道这个框架大致分为哪几个部分，每个部分的作用是什么，这个模块用了什么设计理念等等。

如果说上个阶段是通读源码，那么这个阶段就是要把你在通读源码过程中的收获整理出来。在整理的过程中，你肯定会有更多的疑问，你会不断地细化，不断地精读。

### 批判性思考

通过了上面几个阶段，你会发现你对这个框架有了整体的认识，并且对每个模块的实现细节都有了比较深刻的认识。这个时候，你可以想一想为什么它要这么做，这么做有什么好处，那能用另一种方式做得更好吗？

## What to

### JDK部分

JDK其实就是Java SE Development Kit的缩写，要玩好这东西可不简单。JDK主要包含了三部分，第一部分就是 Java运行时环境 ，这其实就是JVM。此外，第二部分就是 Java的基础类库 ，这个类库的数量还是非常可观的。最后，第三部分就是 Java的开发工具 ，它们都是辅助你更好的使用Java的利器。

那么很显然，要玩好JDK，就是要玩好JDK的这三部分。接下来，咱们就逐个的来说一下，每一个部分要学什么，学到什么程度。

#### 第一部分：Java运行时环境

这一部分其实就是常说的jre，而它的核心其实就一个东西，就是JVM。

JVM这个东西，它的重要性LZ不想再强调了，JVM那本书甚至比《Thinking in java》还重要，这已经足见LZ多么看重JVM了。

当然了，只是LZ看重，当然没什么卵用，但只要Java稍微高级一点点的职位，这部分基本上都是面试必问内容，这更加说明了JVM的重要性。

#### 第二部分：Java的基础类库

1. **精读源码**

   * java.io

   * java.lang

   * java.uti

2. **深刻理解**

   * java.lang.reflect
   * java.netjavax.net.
   * java.nio.
   * java.util.concurrent.*

3. **会用即可**

  java.lang.annotation
  javax.annotation.*
  java.lang.refjava.mathjava.rmi.*
  javax.rmi.*
  java.security.*
  javax.security.*
  java.sqljavax.sql.*
  javax.transaction.*
  java.textjavax.xml.*
  org.w3c.dom.*
  org.xml.sax.*
  javax.crypto.*
  javax.imageio.*
  javax.jws.*
  java.util.*

  剩下的诸如swing、awt稍微了解一下即可，已经很少使用了。更加详细的顺序可以参考这篇博客的内容： [JDK源码阅读顺序](https://blog.csdn.net/qq_21033663/article/details/79571506)

### 拓展

[有哪些值得一读的Java源码-知乎问答](https://www.zhihu.com/question/61539640)

[聊聊我的知识体系](https://www.cnblogs.com/chanshuyi/p/take_about_my_java_tech_system.html#%E5%B9%B6%E5%8F%91%E9%9B%86%E5%90%88%E6%BA%90%E7%A0%81)



参考文章：

> [扎实的java基础](https://mp.weixin.qq.com/s?__biz=MzA3ODg3OTk4OA==&mid=2651087534&idx=1&sn=93f5bd700af1de2f193062b2fd991994&chksm=844ccf35b33b46231892b82bf6aa86b5fb89120a319fec1602a6e74cabff9d5c8e3b9b4e2436&mpshare=1&scene=23&srcid=0430tGrNDhqZlXzixAqH8xxp#rd)
>
> [面对枯燥的源码如何阅读下去](https://www.cnblogs.com/chanshuyi/p/how_to_read_source_code.html)