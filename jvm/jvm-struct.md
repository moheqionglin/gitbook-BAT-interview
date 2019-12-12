![][1]

- MetaSpace
	- 类信息，常量，静态变量
- 堆
	- 类运行时信息
- 栈

### GC root 
栈变量 + 本地栈变量 + metaSpace的静态变量，常量 

### 优化思路
jvm优化目的：
	- 减少GC次数
	- 减少一次GC时间
	
- 对程序中 永远需要的内存和瞬时内存变量的比值，决定JVM的新生代和老年代比例，以及新生代的年龄。
	- 对于程序中的缓存Map，线程池，对象池等都是要伴随应用的整个生命周期，因此我们希望他最好能够一下到老年代，减少yong GC次数。

![][2]
![][3]
![][4]
![][5]
### 各个区的关系
  运行时，先把class加载到MetaSpace，然后执行new Object以后，把常量信息

### GC日志

```
-XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -Xloggc:./gc.log

```

[1]: ../images/jmv-optimize/20191129105439.jpg
[2]: ../images/jmv-optimize/jvm-memory-mod.jpg
[3]: ../images/jmv-optimize/jvm-object-memory-mod.jpg
[4]: ../images/jmv-optimize/jvm-heap-memory.jpg
[5]: ../images/jmv-optimize/jvm-heap-memory2.jpg