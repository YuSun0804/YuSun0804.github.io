---
title: Build OpenJDK
tags:
---

# Download Souce Code
```
hg clone http://hg.openjdk.java.net/jdk8/jdk8 YourOpenJDK 
cd YourOpenJDK 
bash ./get_source.sh
```
# Building & Debuging

# Reading Source Code
![vm_directory]({{ "/_image/vm_directory.png" | absolute_url }})
- jdk
jdk source code.
>src/
>  share/       平台无关的实现
>    classes/     平台无关的Java代码实现
>    native/      平台无关的native代码实现（主要是C）

- hotspot
vm source code.
- langtools



thanks
https://www.jianshu.com/p/ee7e9176632c
https://blog.csdn.net/manageer/article/details/72812149