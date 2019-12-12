## 热加载 线程工具
### Spring loaded

```
<!-- https://mvnrepository.com/artifact/org.springframework/springloaded -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>springloaded</artifactId>
    <version>1.2.6.RELEASE</version>
</dependency>
        
```
启动的时候 加上如下
```
-javaagent:/Users/wanli.zhou/.m2/repository/org/springframework/springloaded/1.2.6.RELEASE/springloaded-1.2.6.RELEASE.jar -noverify
```


