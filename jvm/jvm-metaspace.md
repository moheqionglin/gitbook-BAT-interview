 由于服务器在运行一段时间后出现fullgc，最终定位到是由于metaspace引起fullgc，不断的fullgc又占用大量cpu导致程序最终不可用。

查看JVM配置


```
-Xms1000m -Xmx1000m -XX:MaxNewSize=256m -XX:ThreadStackSize=256 
-XX:MetaspaceSize=38m -XX:MaxMetaspaceSize=380m
```

或者用如下命令查看

```
jinfo -flag MetaspaceSize pid
jinfo -flag MaxMetaspaceSize pid
```

  根据jdk8的metaspace的fullgc的触发条件，初始metaspacesize是38m意味着当第一次加载的class达到38m的时候进行第一次gc（根据JDK 8的特性，G1和CMS都会收集Metaspace区（一般都伴随着Full GC）。），然后jvm会动态调整（gc后会进行调整）metaspacesize的大小。下面几个参数可以对Metaspace进行控制：

-XX:MetaspaceSize=N
这个参数是初始化的Metaspace大小，该值越大触发Metaspace GC的时机就越晚。随着GC的到来，虚拟机会根据实际情况调控Metaspace的大小，可能增加上线也可能降低。
-XX:MaxMetaspaceSize=N
这个参数用于限制Metaspace增长的上限，防止因为某些情况导致Metaspace无限的使用本地内存，影响到其他程序。
-XX:MinMetaspaceFreeRatio=N
当进行过Metaspace GC之后，会计算当前Metaspace的空闲空间比，如果空闲比小于这个参数，那么虚拟机将增长Metaspace的大小。设置该参数可以控制Metaspace的增长的速度，太小的值会导致Metaspace增长的缓慢，Metaspace的使用逐渐趋于饱和，可能会影响之后类的加载。而太大的值会导致Metaspace增长的过快，浪费内存。
-XX:MaxMetasaceFreeRatio=N
当进行过Metaspace GC之后， 会计算当前Metaspace的空闲空间比，如果空闲比大于这个参数，那么虚拟机会释放Metaspace的部分空间。在本机该参数的默认值为70，也就是70%。
-XX:MaxMetaspaceExpansion=N
Metaspace增长时的最大幅度。
-XX:MinMetaspaceExpansion=N
Metaspace增长时的最小幅度。
gc日志分析：
第一次fullgc:

```
[Heap Dump (before full gc): , 0.4032181 secs]2018-01-10T16:37:44.658+0800: 21.673: 
[Full GC (Metadata GC Threshold) [PSYoungGen: 14337K->0K(235520K)]
[ParOldGen: 18787K->30930K(761856K)] 33125K->30930K(997376K), 
[Metaspace: 37827K->37827K(1083392K)], 0.1360661 secs]
[Times: user=0.65 sys=0.04, real=0.14 secs]

```
  主要是Metaspace这里：[Metaspace: 37827K->37827K(1083392K)]达到了设定的初始值38m，并且gc并没有回收掉内存。1083392K这个值是使用了CompressedClassSpaceSize = 1073741824 (1024.0MB)这个导致的。
  从gc日志可以看出来（37827K->37827K），发生full gc时metaspace并没有达到阈值，没有达到阈值为什么还触发了full gc呢？这是因为metaspace committed内存达到阈值，说明metaspace有内存碎片。

第四次fullgc:

```
[Heap Dump (before full gc): , 5.3642805 secs]2018-01-10T16:53:43.811+0800: 980.825: 
[Full GC (Metadata GC Threshold) [PSYoungGen: 21613K->0K(231424K)] 
[ParOldGen: 390439K->400478K(761856K)] 412053K->400478K(993280K), 
[Metaspace: 314108K->313262K(1458176K)], 1.2320834 secs] 
[Times: user=7.86 sys=0.06, real=1.23 secs]
```

  主要是Metaspace这里：[Metaspace: 314108K->313262K(1458176K)]达到了设定的MinMetaspaceFreeRatio，并且gc几乎没有回收掉内存。1458176K这个值是CompressedClassSpaceSize = 1073741824 (1024.0MB)和 MaxMetaspaceSize = 503316480 (480.0MB)的和。后面就是频率很快的重复fullgc。

导致Full GC的罪魁祸首
  对于metaspace内存碎片化，有一个场景倒是可以满足，那就是创建了大量的classloader。目前就一次出现full gc时间点的heap dump不太能看出来问题，我通过增加jvm参数-XX:+HeapDumpBeforeFullGC、-XX:+HeapDumpAfterFullGC分别在发生full gc前后做heap dump。通过对比分析full gc发生前后的heap dump，发现在full gc前创建了大量的sun.reflect.DelegatingClassLoader，full gc后该classloader也减少了约1000个。
  到这里，导致该问题的罪魁祸首就找到了，那就是sun.reflect.DelegatingClassLoader，但是为什么类加载器过多就会导致内存碎片化呢？在类加载器第一次加载类的时候，都会在metaspace区域为其分配一块内存，并且每个类加载器的内存区域都是独立的，当然咯，一定要走出这个误区，类加载器的内存分配跟加载类的数量是没有关系的，即使类加载器只加载一个类，也是会在metaspace为其分配一块内存的。当出现频繁的类加载器创建的时候，这个时候就可能会出现metaspace内存使用率低，但是committed的内存已经达到了full gc的阈值从而触发了full gc。

  总结下原因：classloader不断创建，classloader不断加载class,之前的classloader和class在fullgc的时候没有回收掉。

程序避免创建重复classloader，减少创建classLoader的数量。
增大XX:MinMetaspaceFreeRatio（默认40）的大小，可以看到现在是（100-67.19）。
设置更大的maxmetaspaceSize。