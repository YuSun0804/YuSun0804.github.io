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
2. ByteBuffer

# Advantage And Disadvantage
1. merit
- reduce gc
- reduce replication(when use the IO operation, first need to copy the data to dirrect memory)

2. weakness
- difficult to control(collection)
- difficult to store complex objects except for byte
