---
title: off-head memory
description: 
categories:
 - java memory
tags:
---

广义的堆外内存
-Xmx设置了堆的大小

堆以外的内存都是堆外内存

这些内存主要包括：

线程占用内存

JVM虚拟机占用内存

JNI分配的内存

Direct Buffer

狭义的堆外内存
而作为java开发者，我们常说的堆外内存溢出了，其实是狭义的堆外内存，这个主要是指java.nio.DirectByteBuffer在创建的时候分配内存

堆外内存的申请
以前申请Direct Buffer只能通过Unsafe类，并且只能使用发射的方法：

在使用Unsafe类的时候：

`Unsafe f = Unsafe.getUnsafe();`