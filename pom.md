IDEA springboot热加载

### pom

```
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<optional>true</optional>
</dependency>



<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<fork>true</fork>
					<addResources>true</addResources>
				</configuration>
</plugin>
```

### IDEA配置

-	勾选 Preferences -> Compile -> Build project automatically

-	shift+ctrl+alt+/ 选择 Registry 勾选 compiler.automake.allow.when.app.running

### spring boot 配置
```
spring:
  devtools:
    restart:
      enabled: true
      additional-paths: src/main/java
```



