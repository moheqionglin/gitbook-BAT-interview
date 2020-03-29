Curator是Netflix公司开源的一套zookeeper客户端框架, 解决了很多Zookeeper客户端非常底层的细节开发工作，包括连接重连、反复注册Watcher和NodeExistsException异常等等。


## zookeeper 简介

### zookeeper应用场景
zookeeper是一款非常有修改的 CP分布式高可用跨进程状态存储同步工具。 使用场景不限于下面几种：

- 把zookeeper当做集群的配置中心： 可以再zookeeper上创建 path并且存储对应的元数据信息，集群其他node可以通过 watch /path 的形式监听对应path下面配置信息的改动，然后实时把zookeeper上面的最新配置信息更新到本地。
- 把zookeeper当分布式锁： 大致思路有两种， 1） 【非公平锁】集群的所有节点，同时向zookeeper上注册 /lock 的临时节点，zk保证有且仅有一个节点可以创建成功，这时候其他节点会报Node exists异常，证明已经有lock加锁成功。
                                     2） 【公平锁】集群所有节点同时在zk创建临时有序节点 create -s -e /prefix，然后申请加锁的所有进程通过对比自己是否是最小的sequence节点，如果不是继续自旋等待，如果是进入临界区执行代码。
- 把zookeeper当做leader全局中心： 思路：cluster所有节点同时往zookeeper上create -e /leader "${ip}" ，创建成功的为leader。

### zookeeper常用命令：

不管任何存储系统， 数据库所有的命令无外乎三种： DDL，DML，ACL

```
### DDL 暂无，无schema信息

### DML
# 增
create [-s, 自增] [-e，临时节点] path data acl

# 删
rmr path //递归删除，不管path下面有没有子节点
## 支持乐观锁
delete path [version, 乐观锁，只有当 path对应的dataVersion对的时候才set成功] （有子节点的时候会Node not empty: /t）

# 改 
## 支持乐观锁
set path data [version, 乐观锁，只有当 path对应的dataVersion对的时候才set成功]

# 查
ls path [watch， 监控path改动]
get path [watch， 监控值的改动]
 ## 会显示状态信息
ls2 path [watch]
stat path [watch]


### ACL 有两套方案 quota方案和 ACL方案

setquota -n|-b val path
delquota [-n|-b] path
listquota path

addauth [digest:username:pwd|ip:192.168.1.250|auth:username|world:anyone]:cdwra
setAcl path acl
getAcl path

```                                    

### zookeeper [存储信息介绍][2]

```
[zk: 127.0.0.1:2181(CONNECTED) 17] create /t/c ""
Created /t/c
[zk: 127.0.0.1:2181(CONNECTED) 18] stat /t

cZxid = 0x44d //创建 /t的zk 事务id
ctime = Tue Mar 24 10:15:27 CST 2020 //创建 /t的时间
mZxid = 0x44d //修改 /t的zk 事务id
mtime = Tue Mar 24 10:15:27 CST 2020 //修改 /t的时间
pZxid = 0x44e //  /t的parent 事务id
cversion = 1 //子节点 版本号（每修改一次都会版本号++）
dataVersion = 0 //数据版本号（每修改一次都会版本号++）
aclVersion = 0 //acl版本号（每修改一次都会版本号++）
ephemeralOwner = 0x0 //临时节点的时候非0
dataLength = 0 //存储数据的字节长度
numChildren = 1 //子节点个数

```
 重点关注：
-  修改 path下数据的时候， mZxid++, mtime=now(), dataVersion++, dataLength = byte_length(data)
-  创建child path的时候， mZxid++, mtime=now(), numChildren++



我们可以再zookeeper上通过创建 ``` /path ``` 的形式，让其他进程

## Curator 依赖介绍
```
<!-- 对zookeeper的底层api的一些封装 -->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
</dependency>
<!-- 封装了一些高级特性，如：Cache事件监听、选举、分布式锁、分布式Barrier -->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
</dependency>
```

### 全家桶


| groupId  |artifactId  |机器健康状态  |
| ----  | ----  | ----  |
| org.apache.curator  | curator-recipes  | 所有典型应用场景。需要依赖client和framework，需设置自动获取依赖。  |
| org.apache.curator  | curator-framework  | 最基础的库 |
| org.apache.curator  | curator-test | 包含TestingServer、TestingCluster和一些测试工具。  |
| org.apache.curator  | curator-examples | 各种使用Curator特性的案例。  |
| org.apache.curator  | curator-x-discovery | 在framework上构建的服务发现实现。。  |
| org.apache.curator  | curator-x-discoveryserver | 可以喝Curator Discovery一起使用的RESTful服务器。  |
| org.apache.curator  | curator-x-rpc | Curator framework和recipes非java环境的桥接。  |
	
## 代码


[1]: https://github.com/moheqionglin/spring-demo/tree/master/other-project/zk-curator
[2]: https://www.processon.com/view/link/5e797a83e4b092510f6ff40e