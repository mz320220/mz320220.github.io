---
title: java.io包之Bits类源码
date: 2019-04-22 15:56:05
categories:
- Java
- JDK源码
---

Bits为Default类型，非public类型，只可以被java.io包下的类引用。所以在实际coding中并不会用到这个类。在Bits类中方法均为static类型，通过类名进行访问，主要作为支持方法发挥作用。

<!--more-->

## 概述

> Utility methods for packing/unpacking primitive values in/out of byte arrays using big-endian byte ordering.

在Bits类开头有这么一段描述：一种通过big-endian字节序封装、解封原始数值进、出字节数组的高效方法。

**那么什么是big-endian(大端字节序)呢？**

字节序为大于一个字节大小的数据在内存中的存储顺序（你可以理解为字节为内存最小单元的大小），一般来说一个数据的存储空间在内存中是相邻的，对于CPU读取时，需要辨别多个存储单元所存储数据的顺序，顺序错了可能会得到完全不一样的数据。

计算机硬件有两种储存数据的方式：大端字节序（big endian）和小端字节序（little endian）。

- **大端字节序**：高位字节在前，低位字节在后。（这是人类读写数值的方法）
- **小端字节序**：低位字节在前，高位字节在后

举一个例子，数值`0x2211`使用两个字节保存：高位字节`0x22`，低位字节`0x11`，则大端字节序在内存中保存形式为`0x2211`，以小端字节序保存则为`0x1122`的形式。

关于字节序的进一步理解可以参考阮一峰老师的博客：[理解字节序](http://www.ruanyifeng.com/blog/2016/11/byte-order.html)

## 源码

概览Bits类的源码可以发现所有方法均为getXXX 或 putXXX互为逆过程。以最简单的byte - short转换为例：

```java
/**
 * 由byte数组类型 --> short 
 * short为两个字节，读取两个Byte共16Bits，由于是大端字节序：
 * b[off]数据的高8位 --> 左位移8位到最终位置。
 * b[off+1]数据的低8位
 */
static short getShort(byte[] b, int off) {
	return (short) ((b[off + 1] & 0xFF) +
                    (b[off] << 8));
}

/**
* 与get相反由short --> byte数组类型
* 截取val右8bits作为低位存入b[off +1]
* val无符号右移8位后截取右8bits作为高位存入b[off]
*/
static void putShort(byte[] b, int off, short val) {
	b[off + 1] = (byte) (val      );
    b[off    ] = (byte) (val >>> 8);
}
```

这块源码看起来比较简单，但是深究起来还是有地方需要理解例如：**为什么需要& 0xFF?**

网上乱七八糟的写法一堆，还是要回到计算机组成原理最基础的开始。本文不深入介绍为什么计算机在运算过程中使用补码格式，详细可以参考 [原码反码补码详解](http://www.cnblogs.com/zhangziqiu/archive/2011/03/30/ComputerCode.html)，[大数据溢出问题](https://blog.csdn.net/u011080472/article/details/51280919)；我们需要了解：

1. 计算机使用补码形式进行计算；
2. 正数的原码、反码、补码相同；
3. 负数的反码：原码的除符号位外取反；
4. 负数的补码：反码做+1运算；

除此之外还需要注意Java在二元运算中数据类型的自动提升：(以下按优先级顺序转化)

1. 如果两个操作数其中有一个是double类型，另一个操作就会转换为double类型。 
2. 如果其中一个操作数是float类型，另一个将会转换为float类型。
3. 如果其中一个操作数是long类型，另一个会转换为long类型。 
4. 否则：两个操作数都转换为int类型。（意思是：**如果在二元操作中，不存在double，float，long的话，那么byte、short、char类型都会被转化为int类型**）

现在通过一个例子看一做&0xFF的作用：

```java
public class BitsTest {
    public static void main(String[] args) {
        byte[] a = new byte[10];
        /**
         * 使用补码存储：
         * -127 -> 原码：1111 1111，第一位是符号位；
         *      -> 反码：1000 0000，除了符号位外做取反
         *      -> 补码：1000 0001，对反码做+1运算
         */
        a[0] = -127;
        System.out.println(a[0]); //输出：-127
        /**
        * 以二进制形式输出：111111111111111111111111 1000 0001，
        * 转为int类型，以符号位：1，自动补全至32位
        */
        System.out.println(Integer.toBinaryString(a[0]));
        /**
         * 0xff十六进制一个字节8位：1111 1111
         * 发现会出现byte输出到int类型时会自动补齐至32位：
         *  1111 1111 1111 1111 1111 1111 1000 0001 &
         * (0000 0000 0000 0000 0000 0000)1111 1111
         * ->
         *  0000 0000 0000 0000 0000 0000 1000 0001 = 129
         */
        int c = a[0] & 0xff;
        System.out.println(c);

        /*************************************/
        /**高位b[0]为：
         * 127 -> 0000 0000 0000 0000 0000 0000 0111 1111 << 8
         *       =0000 0000 0000 0000 0111 1111 0000 0000
         * 低位b[1]为：
         * -127-> 1111 1111 1111 1111 1111 1111 1000 0001 & 0xff
         *       =0000 0000 0000 0000 0000 0000 1000 0001
         *
         *    0000 0000 0000 0000 0111 1111 0000 0000
         *  + 0000 0000 0000 0000 0000 0000 1000 0001
         *  = 0000 0000 0000 0000 0111 1111 1000 0001 =32641
         *
         * 如果不做 & 0xff的运算，低位b[1]在二元运算中数据类型得到提升时会补1：
         * 高位b[0]为：
         * 127 -> 0000 0000 0000 0000 0000 0000 0111 1111 << 8
         *       =0000 0000 0000 0000 0111 1111 0000 0000
         * 低位b[1]为：
         * -127-> 1111 1111 1111 1111 1111 1111 1000 0001
         *
         *      0000 0000 0000 0000 0111 1111 0000 0000
         *    + 1111 1111 1111 1111 1111 1111 1000 0001 运算中溢出一位
         *   (1)0000 0000 0000 0000 0111 1110 1000 0001 = 32385
         */
        BitsTest test = new BitsTest();
        byte[] b = new byte[2];
        b[0] = 127;
        b[1] = -127;
        System.out.println(test.getShort(b,0)); //输出：32641
        System.out.println(test.getShort2(b,0)); //输出：32385
    }

    short getShort(byte[] b, int off){
        return (short)((b[off] << 8) +
                (b[off +1] & 0xFF));
    }

    short getShort2(byte[] b, int off){
        return (short)((b[off] << 8) +
                (b[off +1] ));
    }
}
```

通过以上的例子可以发现，进行& 0xFF最大的作用是：**消除运算中类型自动升级导致补码符号位的影响。**

否则会由于符号位参与导致最终结果与期望不一致，换言之，在byte数组转化过程中我们只需要**每一个字节8bits的二进制表示**，而不需要自动补全的符号位。在JDK源码中有很多地方均有使用到& 0xFF运算。

同理，在进行put操作时源码中使用了无符号右移操作(>>>)，不过这一块有点奇怪，根据java的运算规则，char双字节转换为byte类型会进行截取操作，所以无论是有符号右移>>还是无符号都不会影响到字节数组的结果。不知道是不是为了严谨才这么写的~ :sweat:

```java
static void putShort(byte[] b, int off, short val) {
	b[off + 1] = (byte) (val      );
    b[off    ] = (byte) (val >>> 8);
}
```



参考文章：

>[java中的位运算](https://blog.csdn.net/xiaochunyong/article/details/7748713)
>
>[Java中byte做&oxff运算的原因及解析](https://blog.csdn.net/zhaominpro/article/details/79602381)：这一片文章从另一个角度讲了一下，我觉得也很值得一读。