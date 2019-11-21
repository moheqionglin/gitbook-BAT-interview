## testOnborrow 


先说一下自己程序中遇到的问题，前一段时间新写了一个项目，主要架构改进，为前端提供接口（spring +springmvc+mybatis） 在新项目中使用的是阿里的druid连接池，配置简单，除了数据库地址，驱动类，用户名和密码其他一起都是默认，开始的时候由于项目更新上线频率比较多，没有出现太多的问题，后来换库了 。导致之前的链接失效了，请求的时候时好时坏，跟了一下代码以及其他项目的配置，其中有一个属性 testOnBorrow设置为false(默认设置为false) testOnBorrow=false由于不检测池里连接的可用性，
于是假如连接池中的连接被数据库关闭了，应用通过连接池getConnection时，都可能获取到这些不可

用的连接，且这些连接如果不被其他线程回收的话，它们不会被连接池被废除，也不会重新被创建，

占用了连接池的名额，项目本身作为服务端，数据库链接被关闭，客户端调用服务端就会出现大量的timeout，客户端设置了超时时间，然而主动断开，服务端必然出现close_wait ，由于tomcat 默认最大线程数是200，很快就挂掉，虽说多数源，没有问题的数据源，链接并发过来也会死掉，所以说加大tomcat 默认线程（server.tomcat.max-threads=3000）只是短时间内其他数据源链接不会死掉。


默认的配置不适用所有场景，所以使用的时候需要配合场景使用。

 但是这也不能全怪druid，毕竟testOnborrow =true 很大的消耗性能，为了保证服务器的稳定，可以配合其他配置来避免这一点，配合testWhileIdle=true（但是默认为false） 和timeBetweenEvictionRunsMillis来避免这种问题，所以testOnborrow =false是可以提高效率的



由于数据库连接池会在启动时就建立所需的若干连接，并一直保持连接状态，

但是当数据库服务停止后，这些连接就被外部因素给中断了

网上优化了的配置信息：

```

<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">  

<property name="driverClassName" value="${db.driverClassName}"/>  

<property name="url" value="${db.url}"/>  

<property name="username" value="${db.username}"/>  

<property name="password" value="${db.password}"/>  

<!--initialSize: 初始化连接-->  

<property name="initialSize" value="5"/>  

<!--maxIdle: 最大空闲连接-->  

<property name="maxIdle" value="10"/>  

<!--minIdle: 最小空闲连接-->  

<property name="minIdle" value="5"/>  

<!--maxActive: 最大连接数量-->  

<property name="maxActive" value="15"/>  

<!--removeAbandoned: 是否自动回收超时连接-->  

<property name="removeAbandoned" value="true"/>  

<!--removeAbandonedTimeout: 超时时间(以秒数为单位)-->  

<property name="removeAbandonedTimeout" value="180"/>  

<!--maxWait: 超时等待时间以毫秒为单位 6000毫秒/1000等于60秒-->  

<property name="maxWait" value="3000"/>  

<property name="validationQuery">  

<value>SELECT 1</value>  

</property>  

<property name="testOnBorrow">  

<value>true</value>  

</property>  

</bean>  


```
参数  描述

username  传递给JDBC驱动的用于建立连接的用户名

password  传递给JDBC驱动的用于建立连接的密码

url  传递给JDBC驱动的用于建立连接的URL

driverClassName  使用的JDBC驱动的完整有效的Java 类名

connectionProperties  当建立新连接时被发送给JDBC驱动的连接参数，

格式必须是 [propertyName=property;]*

注意 ：参数user/password将被明确传递，所以不需要包括在这里。

 

参数  默认值  描述

defaultAutoCommit  true  连接池创建的连接的默认的auto-commit状态

defaultReadOnly  driver default  连接池创建的连接的默认的read-only状态. 

如果没有设置则setReadOnly方法将不会被调用. (某些驱动不支持只读模式,比如:Informix)

defaultTransactionIsolation  driver default  连接池创建的连接的默认的TransactionIsolation状态. 

下面列表当中的某一个: (参考javadoc)

 

    * NONE

    * READ_COMMITTED

    * READ_UNCOMMITTED

    * REPEATABLE_READ

    * SERIALIZABLE

 

defaultCatalog   连接池创建的连接的默认的catalog

 

参数  默认值  描述

initialSize  0  初始化连接:连接池启动时创建的初始化连接数量,1.2版本后支持

maxActive  8  最大活动连接:连接池在同一时间能够分配的最大活动连接的数量, 

如果设置为非正数则表示不限制

maxIdle  8  最大空闲连接:连接池中容许保持空闲状态的最大连接数量,超过的空闲连接将被释放,

如果设置为负数表示不限制

minIdle  0  最小空闲连接:连接池中容许保持空闲状态的最小连接数量,低于这个数量将创建新的连接,

如果设置为0则不创建

maxWait  无限  最大等待时间:当没有可用连接时,连接池等待连接被归还的最大时间(以毫秒计数),

超过时间则抛出异常,如果设置为-1表示无限等待

 

参数  默认值  描述

validationQuery   SQL查询,用来验证从连接池取出的连接,在将连接返回给调用者之前.如果指定,

则查询必须是一个SQL SELECT并且必须返回至少一行记录

testOnBorrow  true  指明是否在从池中取出连接前进行检验,如果检验失败,

则从池中去除连接并尝试取出另一个.

注意: 设置为true后如果要生效,validationQuery参数必须设置为非空字符串

testOnReturn  false  指明是否在归还到池中前进行检验

注意: 设置为true后如果要生效,validationQuery参数必须设置为非空字符串

testWhileIdle  false  指明连接是否被空闲连接回收器(如果有)进行检验.如果检测失败,

则连接将被从池中去除.

注意: 设置为true后如果要生效,validationQuery参数必须设置为非空字符串

timeBetweenEvictionRunsMillis  -1  在空闲连接回收器线程运行期间休眠的时间值,以毫秒为单位.

 如果设置为非正数,则不运行空闲连接回收器线程

numTestsPerEvictionRun  3  在每次空闲连接回收器线程(如果有)运行时检查的连接数量

minEvictableIdleTimeMillis  1000 * 60 * 30  连接在池中保持空闲而不被空闲连接回收器线程

(如果有)回收的最小时间值，单位毫秒

 

参数  默认值  描述

poolPreparedStatements  false  开启池的prepared statement 池功能

maxOpenPreparedStatements  不限制  statement池能够同时分配的打开的statements的最大数量, 

如果设置为0表示不限制

 

 

这里可以开启PreparedStatements池. 当开启时, 将为每个连接创建一个statement池,

并且被下面方法创建的PreparedStatements将被缓存起来:

    * public PreparedStatement prepareStatement(String sql)

    * public PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency)

注意: 确认连接还有剩余资源可以留给其他statement

参数  默认值  描述

accessToUnderlyingConnectionAllowed  false  控制PoolGuard是否容许获取底层连接

 

 

如果容许则可以使用下面的方式来获取底层连接:

    Connection conn = ds.getConnection();

    Connection dconn = ((DelegatingConnection) conn).getInnermostDelegate();

    ...

    conn.close();

 

默认false不开启, 这是一个有潜在危险的功能, 不适当的编码会造成伤害.

(关闭底层连接或者在守护连接已经关闭的情况下继续使用它).请谨慎使用,

并且仅当需要直接访问驱动的特定功能时使用.

注意: 不要关闭底层连接, 只能关闭前面的那个.

参数  默认值  描述

removeAbandoned  false  标记是否删除泄露的连接,如果他们超过了removeAbandonedTimout的限制.

如果设置为true, 连接被认为是被泄露并且可以被删除,如果空闲时间超过removeAbandonedTimeout. 

设置为true可以为写法糟糕的没有关闭连接的程序修复数据库连接.

removeAbandonedTimeout  300  泄露的连接可以被删除的超时值, 单位秒

logAbandoned  false  标记当Statement或连接被泄露时是否打印程序的stack traces日志。

被泄露的Statements和连接的日志添加在每个连接打开或者生成新的Statement,

因为需要生成stack trace。

 

 

如果开启"removeAbandoned",那么连接在被认为泄露时可能被池回收. 这个机制在(getNumIdle() < 2)

 and (getNumActive() > getMaxActive() - 3)时被触发.

举例当maxActive=20, 活动连接为18,空闲连接为1时可以触发"removeAbandoned".

但是活动连接只有在没有被使用的时间超过"removeAbandonedTimeout"时才被删除,默认300秒.

在resultset中游历不被计算为被使用.

