实际项目开发过程中，常常需要动态修改运行时某个java对象的某个属性。动态修改某个java对象的属性有很多种方法，下面列举下：
* JMX bean的形式动态修改
* 修改配置中心，然后热更新配置生效
* 自己写一个RPC报漏出去，共外面的人调用RPC修改对应的值。

<br />
下面先介绍第三种, 
<br />

整体架构如下， [源码][1]
![架构图][2]



[1]: https://github.com/moheqionglin/spring-demo/tree/master/devops/devops-service/src/main/java/com/moheqionglin/remotecontroll
[2]: ../images/spring/remoteControllerTelnet.jpg