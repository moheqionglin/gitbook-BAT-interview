本文基于 dubbo2.7.0版本开始讲解： [涉及到的源码][3]

涉及到如下部分：
 - Dubbo 入门使用
    - dubbo + spring xml
    - dubbo + spring注解 (注意Provider使用de  @Service要用dubbo的注解，不要用spring的@Service)
    - 纯 java使用
    - spring boot
 - RPC 核心
    - 基础
        -   Provider的坐标： <application, interface, method, group, version >
        -   Provider 限制 < payload, Thread限制>)   
        -   延迟暴露服务
 
    - [集群容错（客户端调用Provider的时候Provider异常）][1] (集群容错是在Provider在无法提供服务的时候比如超时，挂了，但是ZK上还有注册信息，Consumer失败后的选择Provider策略)：
        -   Failover
        -   Fail Fast ： 快速失败，只发起一次调用，失败立即报错非幂等写操作，比如新增记录
        -   Fail safe： 客户端忽略server端的异常，通常用于写入审计日志等不重要操作
        -   Fail back： 失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。
        -   Forking Cluster： 并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 forks="2" 来设置最大并行数。
        -   Broadcast Cluster： 调用所有Provider，任意一台报错则报错 [2]。通常用于通知所有提供者更新缓存或日志等本地资源信息。
    - [负载均衡(客户端调用Provider的时候选择策略)]：
        -   Random LoadBalance： 按权重设置随机概率
        -   RoundRobin LoadBalance： 按公约后的权重设置轮询比率（存在慢的提供者累积请求的问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上）
        -   LeastActive LoadBalance：最少活跃调用数（使慢的提供者收到更少请求）
        -   ConsistentHash LoadBalance：
    - 协议
        -   Dubbo
        -   Rest
        -   Http
                  
 - 服务治理
    -   注册中心
    -   配置中心
    -   监控指标
    -   断路器： Hystrix， Sentinel
    -   Admin界面
 -  微服务组件
    - API Gateway： dubbo proxy （http调用dubbo）
    - API Manager： Dubbo-swigger  
    
## 实际生产性能
下面压测基于 dubbo 2.5.3 ， 

|  序号   | 机器配置  |测试结果  |机器健康状态  | 场景 |
|  ----  | ----  | ----  | ----  | ----  |
|  1  | 1* 2C4G共享型  | 3.08W/S 稳定  | CPU接近100%  | 发送800B大小的数据到server  |
|  2  | 3* 4C8G 1TB  | 40W/S  | CPU接近40%,内存正常  | protobuf格式发送800B大小压缩前数据到kafka， 60个partition  |
|  3  | 3* 8C16G 20TB  | 20W/S  | CPU在40%到100%见波动，有毛刺  | geomesa格式写入attr |


   
    
    
[1]: http://dubbo.apache.org/zh-cn/docs/user/demos/fault-tolerent-strategy.html
[2]: http://dubbo.apache.org/zh-cn/docs/user/demos/loadbalance.html
[3]: https://github.com/moheqionglin/spring-demo/tree/master/other-project/dubbo-app
    