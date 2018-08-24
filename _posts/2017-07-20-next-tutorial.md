---
title: Off-Heap Memory
description: 
categories:
tags:
---

# Introduction
generalized off-heap memory, including:

- memory used by thread
- memory used by jvm
- memory allocated by JNI
- memory allocated by Direct Buffer

narrow off-heap memory only means the memory allocated by Direct Buffer

# Use Off-Heap Memory
1. Unsafe
```
Field f = Unsafe.class.getDeclaredField("theUnsafe");
f.setAccessible(true);
Unsafe us = (Unsafe) f.get(null);
long id = us.allocateMemory(1000);
```

2. ByteBuffer

# Advantage And Disadvantage
1. merit
- reduce gc
- reduce replication(when use the IO operation, first need to copy the data to dirrect memory)

2. weakness
- difficult to control(collection)
- difficult to store complex objects except for byte

# Off-Heap Memory Collection
in young area, hold the reference(DirectByteBuffer) to the direct memory, which is usually to big like iceberg. if the reference object is unreachable, the direct memory will be collected along with the the collection of the reference.
![off-heap1]({{ "/image/off-heap1.png" | absolute_url }})

after gc promotion, if the DirectByteBuffer object promote to the old area,only full gc can collect the object. without the full gc, it will cause the out of memory error.
![off-heap1]({{ "/image/off-heap2.png" | absolute_url }})

# Solution
use System.gc()
notice: do not use -XX:DisableExplicitGC; use the -XX:ExplicitGCInvokesConcurrent instead of -XX:DisableExplicitGC, if necessary.

