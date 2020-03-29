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


## 关于热加载的几个疑问

### 1. 热加载如是实现？ 
原理其实就是利用了 JVM中双亲委派+统一类加载器里面的类智能被加载一次的原理。 具体过程：
- 自定义ClassLoader
- 写一个Dameon进程，检测lib目录中的class文件是是否有更新。
- 如果有更新，那么卸载老的类，重新加载新的class。
- 加载完成新的class以后，反射调用 newInstance() 来新建对象。

### 2. 疑问，热加载以后，之前class实例化的对象的 对象级别的变量是否会初始化？
答案： 当然是的。 因为 热加载以后，对象会重新被创建，因此对象级别的 状态都会清空。
<br /> 举个例子

```
# 热加载以后 对象级别变量 i 会重新初始化。
@RestController
@RequestMapping("/api/v1/")
public class AddressController {
    private int i = 1;

    @GetMapping(path = "/address/tk1")
    public String getTk1(){
        System.out.println("ccc" + i);
        return ++i + "---";
    }
}

```

