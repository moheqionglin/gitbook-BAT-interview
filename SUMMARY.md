# Summary

* [Introduction](README.md)
	* [total](total.md)
* [正文](part2/README.md)
    * [hbase 文章](part/hbase.md)
* [性能排查](performence.md)	
    
--

* [react](react/react-组件生命周期.md)
    * [组件生命周期](react/react-组件生命周期.md)
    * [布局](react/布局.md)
    
--  
* geomesa
	* [geomesa cli](geomesa/geomesa-cli.md)
	* [geohash,s2,空间曲线](geomesa/z2-s2.md)
	
--
* spring
	* [本地源码搭建](spring/spring-source.md) 
	* [spring 生命周期](spring/life-cycle.md)
	* [spring 向容器中增加bean方式](spring/spring-beans.md)
	* [spring AOP植入](spring/spring-aop.md)
	* [spring 事务](spring/spring-transaction.md)
* dubbo
	* [dubbo 扩展SPI](dubbo/dubbo-extension-spi.md)
	* [dubbo使用的一些细节](dubbo/dubbo-use.md)
	
* mybatis
	*[mybatis 官方资料](mybatis/mybatis-index.md)	
--
* 算法
	* [位运算的应用](algorithm/algorithm.md)

--
* hbase
	* [优雅重启regionserver命令](hbase/restart-regionserver.md)

-- 
* 数据库
	* [数据库连接池](db/db-connection-pool.md)
	* [数据库事务](db/db-transaction.md)

--

* 高并发
	* [Java Memory Modle内存模型]
		* [线程，进程](concurrent/java-thread.md)
		* [jmm模型，volatile，EMSI](concurrent/java-jmm-volatile.md)
		* [MESI缓存一致性协议和volatile，EMSI](concurrent/java-volatile.md)
		* [指令重排，可见性，原子性，顺序一致性]
	* 并发同步处理
		* [synchronized内置锁实现原理](concurrent/java-synchronized.md)  
		* [synchronized锁膨胀过程](concurrent/java-synchronized2.md) 
		* [AbstractQueueSynchronizer详解](concurrent/java-aqs.md)
		* [乐观锁&悲观锁，重入锁&非重入锁，公平锁&非公平锁，锁粒度]
		* [JVM逃逸分析,锁消除，锁粗化，堆栈转移](concurrent/escape-analysis.md)
		* [ReentrantLock, ReentrantReadWriteLock, ReadWriteLock源码]
		* [Condition 条件队列和Object.wait队列]
	* 并发同步工具类
		* [CountDownLatch] 
		* [CyclicBarrier]  
		* [Semaphore]
		* [Exchanger]
	* Atomic包
		* [并发Atomic，CAS，Unsafe](concurrent/java-unsafe.md)
	* 并发集合
		* [BlockQueue]
			* [ArrayBlockingQueue]
			* [ConcurrentLinkedQueue]
			* [PriorityBlockingQueue]
			* [DelayQueue]
		* 并发安全集合
			* [HashMap, ConcurrentHashMap源码]
			* [ArrayList, LinkedList, CopyOnWriteArrayList源码]			* [Set 和 CopyOnWriteArraySet源码]
	* 线程池源码
		* 线程池ThreadPoolExecutor源码
		* ScheduleExecutorService详解
		* Future模式详解
	* ForkJoin框架
		* ForkJoin框架
		* ForkJoin原理
		* ForkJoin应用	
	* 分布式高并发
		* [redis缓存和数据库一致性](concurrent/cache-consistent.md)
	
* JVM
	* [JVM 概述](jvm/jvm-struct.md)
	* [metaspace空间](jvm/jvm-metaspace.md)
	* [JVM指令集和javap查看编译后的文件](jvm/jvm-instruction.md)
	* [jvm 加载器](jvm/jvm-class-loader.md)
	* [jvm热加载原理](jvm/jvm-hot-reload.md)
* DevOps
	* [DevOps](devops/devops.md)
	* [日志收集](devops/log-collect.md)
	* [JMX 收集](devops/JMX-monitor.md) 
	* [linux运维必会命令](devops/linux运维必会命1令.md) 
    * [remote-controll](devops/remote-controll.md)
--
* kafka
	* [kafka broker倾斜问题解决方案](kafka/kafka-skew.md)


* shell
	* [shell编程](shell/shell.md)
	
* 应用类
	* [小程序申请](app/xiaochengxu.md)
	* [小程序申请支付](app/xiaochengxu-pay.md)
	* [小程序申请并配置免费证书](购买免费的ssl证书.md)
	* [字词句](app/xiaochengxu-statement.md)
	* [easy-mock](app/easy-mock.md)
	* [centos 安装 docker，docker-compose](app/centos-docker.md)
	* [centos 安装 jenkins docker 配置内存限制](app/centos-docker.md)
	* [PG 协议](app/pg-v3.md)