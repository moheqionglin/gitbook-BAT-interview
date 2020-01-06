### @Bean



### org.springframework.beans.factory.config.SingletonBeanRegistry#registerSingleton

```
org.springframework.context.support.AbstractApplicationContext#refresh
	org.springframework.context.support.AbstractApplicationContext#prepareBeanFactory
		
```

```
AnnotationConfigApplicationContext context1 = new AnnotationConfigApplicationContext();
		context1.register(Config.class);
		context1.getBeanFactory().registerSingleton("xxx", new Object());
		context1.refresh();
```


```
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

 <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-application.xml</param-value>
</context-param>

```