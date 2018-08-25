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
![vm_directory]({ "/assets/images/post/vm_directory.png" | absolute_url})
## jdk
jdk source code.
![vm_directory @2x]({{ "/assets/images/post/jdk_source.png" | absolute_url }})
>src/   
  share/       平台无关的实现   
    classes/     平台无关的Java代码实现   
    native/      平台无关的native代码实现（主要是C） 

two types of JNI call
```c
static JNINativeMethod methods[] = {
     {"hashCode",     "()I",     (void *)&JVM_IHashCode},
     {"wait",         "(J)V",    (void *)&JVM_MonitorWait},     
     {"notify",       "()V",     (void *)&JVM_MonitorNotify},     
     {"notifyAll",    "()V",     (void *)&JVM_MonitorNotifyAll},            
     {"clone",        "()Ljava/lang/Object;", (void *)&JVM_Clone},};
```

```c
return (*env)->GetObjectClass(env, this);
```

## hotspot
vm source code.
![vm_directory]({{ "/assets/images/post/vm_source.png" | absolute_url }})
- first type call in jvm.cpp(share/vm/prims/jvm.cpp)
- second type call in jni.cpp(src/share/vm/prims/jni.cpp)

# langtools
mainly for tools like java， javah，javadoc，javap
![vm_directory]({{ "/assets/images/post/langtools_source.png" | absolute_url }})


thanks
https://www.jianshu.com/p/ee7e9176632c
https://blog.csdn.net/manageer/article/details/72812149

more soucce code https://blog.csdn.net/sinat_38259539/article/details/78120498

how java run https://blog.csdn.net/xiangzhihong8/article/details/65657914 main gate in \jdk\src\share\bin\java.c