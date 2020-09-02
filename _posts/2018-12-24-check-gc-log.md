---
title: GC Log Analysis
categories:
- Java
tags:
---

# Introduction
This blog is going to show how to read the gc log both in java7 and java8. Firstly, if we want to learn gc log better, we need to sse below parameters to print the timestamp and detail info:
- -XX:+PrintGCDetails
- -XX:+PrintGCTimeStamps

In java7, we usually use ParNew(Copying) for young area, and CMS(Mark-Sweep) for old area; while in java8, G1 is widely used for young and old area.

# Young GC
```
39.910: [GC 39.910: [ParNew: 261760K->0K(261952K), 0.2314667 secs] 262017K->26386K(1048384K), 0.2318679 secs]
```

Size of young area is 261952Kb, after gc the used memory decrease from 261970Kb to 0Kb, with 0.2318679 second consumed.

# Old GC
```
40.146: [GC [1 CMS-initial-mark: 26386K(786432K)] 26404K(1048384K), 0.0074495 secs]
```
Starting the old gc of initial-mark, with size of young area is 786432Kb, used memory before gc is 26386Kb. This phrase is stop-the-world and process by one thread. there are two things to do:
1. marking the object link by the root directly
2. marking the object link by the yound area object

```
40.154: [CMS-concurrent-mark-start]
```
Starting the concurrent-mark, gc thread and application thread execute concurrently.

```
40.683: [CMS-concurrent-mark: 0.521/0.529 secs]
```
The concurrent-mark is end, cost 0.521 second.
因为该阶段是并发执行的,运行期间可能会在老年代产生新的垃圾，或者老年代的对象被新生代对象重新引用,这些对象都是要重新标记的，有以下几种场景:
1. 新生代对象晋升到老年代
2. 新生代新创建对象引用了原先老年代未被标记的对象
3. 直接在老年代分配的对象
4. 老年代对象的引用关系发生变化
为了提高重新标记的效率,该阶段会把上述对象所存在的card标记为dirty，后续只需要扫描dirty card对应的对象。

```
40.683: [CMS-concurrent-preclean-start]
```
Staring the preclean, it can be closed by -XX:-CMSPrecleaningEnabled. This is for clean the young area.

```
40.701: [CMS-concurrent-preclean: 0.017/0.018 secs]
```
The preclean is end, cost 0.018 second

```
40.704: [GC40.704: [Rescan (parallel) , 0.1790103 secs]40.883: [weak refs processing, 0.0100966 secs] [1 CMS-remark: 26386K(786432K)] 52644K(1048384K), 0.1897792 secs]
```
This is the remark with Stop-the-world, there are some need to do:
1. 遍历新生代对象，重新标记
2. 根据GCRoots,重新标记
3. 遍历老年代的Dirty card重新标记，这部分对象大部分在预清理阶段已经处理过。

```
40.894: [CMS-concurrent-sweep-start]
41.020: [CMS-concurrent-sweep: 0.126/0.126 secs]
```
清除死亡的(没有标记的)对象，这个阶段jvm应用线程并发执行。清除阶段会0.126秒.

```
41.020: [CMS-concurrent-reset-start]
41.147: [CMS-concurrent-reset: 0.127/0.127 secs]
```
Reset阶段,这个阶段,CMS的数据结构重新初始化,为下一次垃圾回收做准备。


# Case Analysis
```
197.976: [GC 197.976: [ParNew: 260872K->260872K(261952K), 0.0000688 secs]
197.976: [CMS197.981: [CMS-concurrent-sweep: 0.516/0.531 secs]
(concurrent mode failure): 402978K->248977K(786432K), 2.3728734 secs] 663850K->248977K(1048384K), 2.3733725 secs]
```
可以看出开始了新生代的收集,但是没有执行因为这时候又发生了CMS GC，CMS老年代又没有足够的空间来容纳新生代的对象，这样就出现了concurrent mode failure这样的字样,这个时候会放弃CMS GC周期转而执行一次Full GC。concurrent mode failure可以通过增加老年代的空间或者减少老年代GC的阈值来解决,可以把GC参数CMSInitiatingOccupancyFraction设置小一点,默认是92,这个参数只在第一次收集时才生效,后续jvm会自动调整,你要同时设置UseCMSInitiatingOccupancyOnly=true这个参数才一直生效,CMSInitiatingOccupancyFraction这个参数要选的比较合适才行，太小了会导致CMS GC太频繁,太大了就可能出现并发模式失败,有时候你会发现老年代明明有足够的空间还是出现了promotion failures，这是因为CMS GC产生了很多内存碎片,没有连续的内存区域来存储新生代的对象。CMS GC还有一种原因是晋升担保,cms收集器会计算新生代的对象是否能够成功晋升到老年代,如果晋升担保失败也是会触发一次CMS GC周期，最新版本的CMS晋升担保会根据新生代晋升到老年代对象的历史数据来决定这个担保的值。

```
7688.150: [CMS-concurrent-preclean-start]
7688.186: [CMS-concurrent-preclean: 0.034/0.035 secs]
7688.186: [CMS-concurrent-abortable-preclean-start]
7688.465: [GC 7688.465: [ParNew: 1040940K->1464K(1044544K), 0.0165840 secs] 1343593K->304365K(2093120K), 0.0167509 secs]
7690.093: [CMS-concurrent-abortable-preclean: 1.012/1.907 secs]
7690.095: [GC[YG occupancy: 522484 K (1044544 K)]7690.095: [Rescan (parallel) , 0.3665541 secs]7690.462: [weak refs processing, 0.0003850 secs] [1 CMS-remark: 302901K(1048576K)] 825385K(2093120K), 0.3670690 secs]
```
上面的日志信息，在并发预清理之后开启了可中断的预清理阶段,当新生代的使用率到达522484K(50%阈值)时,预清理被中断随后执行重新标记阶段。从JDK1.5开始,CMS多了一个阶段,叫做concurrent-abortable-preclean，可中断的预清理就当在并发预清理和重新标记中间,这个阶段也是用来减少重新标记花费的时长,这个阶段通过jvm参数CMSScheduleRemarkEdenSizeThreshold设置阈值,默认为2M，即新生代使用率超过2M才开启concurrent-abortable-preclean，这个阶段主要循环做2件事情:
1. 处理from和to区的对象,标记可达的老年代对象
2. 和并发预清理一样，处理被标记为dirty card区的对象

这个逻辑不会一直循环下去，打断这个循环的条件有三个：
1. 通过参数CMSMaxAbortablePrecleanLoops设置最多循环的次数，默认是0,没有循环次数限制;
2. 执行这个逻辑的时间达到了CMSMaxAbortablePrecleanTime参数设置的阈值，默认是5s。
3. 新生代Eden区的内存使用率达到了CMSScheduleRemarkEdenPenetration参数设置的阈值，默认50%，会退出循环。
如果在循环退出之前，发生了一次YGC，对于后面的Remark阶段来说，大大减轻了扫描年轻代的负担。
