使用逃逸分析，编译器可以对代码做如下优化
注意， 只有 -server模式才会有逃逸分析。

- 同步省略（锁消除）。如果一个对象被发现只能从一个线程被访问到，那么对于这个对象的操作可以不考虑同步
- 锁粗化
- 将堆分配转化为栈分配。如果一个对象在子程序中被分配，要使指向该对象的指针永远不会逃逸，对象可能是栈分配的候选，而不是堆分配
- 分离对象或标量替换。有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么对象的部分（或全部）可以不存储在内存，而是存储在CPU寄存器中

Java虚拟机对内部锁的实现进行了一些优化。这些优化主要包括锁消除（Lock Elision）、锁粗化（Lock Coarsening）、偏向锁（Biased Locking）以及适应性自旋锁（Adaptive Locking）。这些优化仅在Java虚拟机server模式下起作用（即运行Java程序时我们可能需要在命令行中指定Java虚拟机参数“-server”以开启这些优化）

### 同步省略（锁消除）

```

public void Concurrent() {
    Object lock = new Object();
    synchronized(lock) {
        System.out.println(lock);
    }
}

```

如果JIT经过逃逸分析之后发现并无线程安全问题的话, 会进行锁消除。

```
public void Concurrent() {
   System.out.println(lock);
}

```

### 锁粗化
就像你去银行办业务，你为了减少每次办理业务的时间，你把要办的五个业务分成五次去办理，这反而适得其反了。因为这平白的增加了很多你重新取号、排队、被唤醒的时间。

如果在一段代码中连续的对同一个对象反复加锁解锁，其实是相对耗费资源的，这种情况可以适当放宽加锁的范围，减少性能消耗。

当JIT发现一系列连续的操作都对同一个对象反复加锁和解锁，甚至加锁操作出现在循环体中的时候，会将加锁同步的范围扩散（粗化）到整个操作序列的外部。

```
for(int i=0;i<100000;i++){  
   synchronized(this){  
       do();  
}

```
会被粗化成

```
synchronized(this){  
   for(int i=0;i<100000;i++){  
       do();  
}

```

### 标量替换
标量（Scalar）是指一个无法再分解成更小的数据的数据。Java中的原始数据类型就是标量。相对的，那些还可以分解的数据叫做聚合量（Aggregate），Java中的对象就是聚合量，因为他可以分解成其他聚合量和标量。

在JIT阶段，如果经过逃逸分析，发现一个对象不会被外界访问的话，那么经过JIT优化，就会把这个对象拆解成若干个其中包含的若干个成员变量来代替。这个过程就是标量替换。


```
public static void main(String[] args) {
   alloc();
}
 
private static void esapeAnalysis() {
   Point point = new Point（1,2）;
   System.out.println("point.x="+point.x+"; point.y="+point.y);
}

class Point{
    private int x;
    private int y;
}
```

以上代码中，point对象并没有逃逸出alloc方法，并且point对象是可以拆解成标量的。那么，JIT就会不会直接创建Point对象，而是直接使用两个标量int x ，int y来替代Point对象。

以上代码，经过标量替换后，就会变成：

```

private static void esapeAnalysis() {
   int x = 1;
   int y = 2;
   System.out.println("point.x="+x+"; point.y="+y);
}
```
可以看到，Point这个聚合量经过逃逸分析后，发现他并没有逃逸，就被替换成两个聚合量了。那么标量替换有什么好处呢？就是可以大大减少堆内存的占用。因为一旦不需要创建对象了，那么就不再需要分配堆内存了。


### 将堆分配转化为栈分配
标量替换为栈上分配提供了很好的基础。 对分配转换为栈分配的理论基础就是标量替换。

```
public class EscapeAnalysis{
    public void alloc() {
        byte[] b = new byte[2];
        b[0] = 1;
    }
    public void batchCreate(){
        long b = System.currentTimeMillis();
        for (int i = 0; i < 100000000; i++) {
            alloc();
        }
        long e = System.currentTimeMillis();
        System.out.println(e - b);
    }
    /**
	* -server -XX:+DoEscapeAnalysis -XX:+PrintGCDetails -Xmx10m -Xms10m
    * 8
    * -server -XX:-DoEscapeAnalysis -XX:+PrintGCDetails -Xmx10m -Xms10m
    * 1058
    */
    public static void main(String[] args) {
        new EscapeAnalysis().batchCreate();
    }
}

```

### 附录， 逃逸分析以后，是不是不逃逸的就都分配到栈去了？

```
public class EscapeAnalysis{

	public static void main(String[] args) {
	   long a1 = System.currentTimeMillis();
	   for (int i = 0; i < 1000000; i++) {
	       alloc();
	   }
	   // 查看执行时间
	   long a2 = System.currentTimeMillis();
	   System.out.println("cost " + (a2 - a1) + " ms");
	   // 为了方便查看堆内存中对象个数，线程sleep
	   try {
	       Thread.sleep(100000);
	   } catch (InterruptedException e1) {
	       e1.printStackTrace();
	   }
	}
	 
	private static void alloc() {
	   Point user = new Point();
	}
	 
	static class Point {}
}	
```

```
# 关闭逃逸分析 -XX:-DoEscapeAnalysis 

~ jps
9086 StackAllocTest

~ jmap -histo 9086
 
num     #instances         #bytes  class name
----------------------------------------------
  1:           524       87282184  [I
  2:       1000000       16000000  EscapeAnalysis$Point
  3:          6806        2093136  [B
  4:          8006        1320872  [C
  5:          4188         100512  java.lang.String

# 打开逃逸分析 -XX:+DoEscapeAnalysis 
~ jps
2094 StackAllocTest

 ~ jmap -histo 2094
 
num     #instances         #bytes  class name
----------------------------------------------
  1:           524      101944280  [I
  2:          6806        2093136  [B
  3:         83619        1337904  EscapeAnalysis$Point
  4:          8006        1320872  [C
  5:          4188         100512  java.lang.String


```
**综上 开启了逃逸分析之后（-XX:+DoEscapeAnalysis），在堆内存中只有8万多个EscapeAnalysis$Point对象。也就是说在经过JIT优化之后，堆内存中分配的对象数量，从100万降到了8万**



关于逃逸分析的论文在1999年就已经发表了，但直到JDK 1.6才有实现，而且这项技术到如今也并不是十分成熟的。

其根本原因就是无法保证逃逸分析的性能消耗一定能高于他的消耗。虽然经过逃逸分析可以做标量替换、栈上分配、和锁消除。但是逃逸分析自身也是需要进行一系列复杂的分析的，这其实也是一个相对耗时的过程。

一个极端的例子，就是经过逃逸分析之后，发现没有一个对象是不逃逸的。那这个逃逸分析的过程就白白浪费掉了。

虽然这项技术并不十分成熟，但是他也是即时编译器优化技术中一个十分重要的手段。
