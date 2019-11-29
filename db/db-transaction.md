
### mysql 
#### 首先确认的几个东西
```
#查看数据库版本
select @@version;
#确认是否自动提交
show variables like 'autocommit';
# 为了做demo可以先关闭自动提交
set autocommit=0;

```


```


#mysql8数据库事务隔离级别命令
select @@global.transaction_isolation,@@transaction_isolation;
#老版本
select @@global.tx_isolation,@@tx_isolation;
#SET [SESSION | GLOBAL] TRANSACTION ISOLATION LEVEL {READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE}
set session transaction isolation level serializable;
set session transaction isolation level read committed;
```