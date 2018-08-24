---
title: Off-Heap Memory
description: 
categories:
 - jvm
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

(```)
    Field f = Unsafe.class.getDeclaredField("theUnsafe");
    f.setAccessible(true);
    Unsafe us = (Unsafe) f.get(null);
    long id = us.allocateMemory(1000);
(```)
