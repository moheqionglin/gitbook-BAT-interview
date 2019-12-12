内存和CPU缓存数据不一致性。 为了解决数据不一致性， 需要串行化执行代码，所以需要互斥锁。
redis和数据库数据不一致性。 为了解决数据不一致性，需要




synchronized 底层用C++

线程安全，原因：  
   内存和CPU缓存数据不一致性。 为了解决数据不一致性， 需要串行化执行代码，所以需要互斥锁。
   
AQS： doug lee，完全用java实现
- 自旋
- CAS
- LockSupport.park, LockSupport.unpark

https://processon.com/view/link/5cb6c8a4e4b059e209fbf369#map


https://www.jianshu.com/p/4682a6b0802d
https://www.jianshu.com/p/7ccf49e50920