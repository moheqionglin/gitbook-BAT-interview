###使用编程事务实现手动事务

#### 手动事务1
```
@Component
public class TransactionUtils {

    @Autowired
    private DataSourceTransactionManager dataSourceTransactionManager;

    // 开启事务
    public TransactionStatus begin() {
        TransactionStatus transaction = dataSourceTransactionManager.getTransaction(new DefaultTransactionAttribute());
        return transaction;
    }

    // 提交事务
    public void commit(TransactionStatus transactionStatus) {
        dataSourceTransactionManager.commit(transactionStatus);
    }

    // 回滚事务
    public void rollback(TransactionStatus transactionStatus) {
        dataSourceTransactionManager.rollback(transactionStatus);
    }
}

@Service
public class UserService {
    @Autowired
    private UserDao userDao;
    @Autowired
    private TransactionUtils transactionUtils;

    public void add() {
        TransactionStatus transactionStatus = null;
        try {
            transactionStatus = transactionUtils.begin();
            userDao.add("wangmazi", 27);
            int i = 1 / 0;
            System.out.println("我是add方法");
            userDao.add("zhangsan", 16);
            transactionUtils.commit(transactionStatus);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (transactionStatus != null) {
                transactionStatus.rollbackToSavepoint(transactionStatus);
            }
        }

    }

}

```

#### 手动事务2

```
@Component
@Aspect
public class AopTransaction {
    @Autowired
    private TransactionUtils transactionUtils;

    // // 异常通知
    @AfterThrowing("execution(* com.service.UserService.add(..))")
    public void afterThrowing() {
        System.out.println("程序已经回滚");
        // 获取程序当前事务 进行回滚
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
    }

    // 环绕通知
    @Around("execution(* com.service.UserService.add(..))")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("开启事务");
        TransactionStatus begin = transactionUtils.begin();
        proceedingJoinPoint.proceed();
        transactionUtils.commit(begin);
        System.out.println("提交事务");
    }

}

``` 

### 声明式事务

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
         http://www.springframework.org/schema/context/spring-context.xsd
         http://www.springframework.org/schema/aop
         http://www.springframework.org/schema/aop/spring-aop.xsd
         http://www.springframework.org/schema/tx
          http://www.springframework.org/schema/tx/spring-tx.xsd">


    <!-- 开启注解 -->
    <context:component-scan base-package="com.itmayiedu"></context:component-scan>
    <!-- 1. 数据源对象: C3P0连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/test"></property>
        <property name="user" value="root"></property>
        <property name="password" value="root"></property>
    </bean>

    <!-- 2. JdbcTemplate工具类实例 -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <!-- 配置事物 -->
    <bean id="dataSourceTransactionManager"
        class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <!-- 开启注解事物 -->
    <tx:annotation-driven transaction-manager="dataSourceTransactionManager" />
</beans>

@Transactional
public void add() {
    userDao.add("wangmazi", 27);
    int i = 1 / 0;
    System.out.println("我是add方法");
    userDao.add("zhangsan", 16);
}

```