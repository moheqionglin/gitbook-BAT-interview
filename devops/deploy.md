生产级别的打包部署方式有很多种，但是万变不离其宗。大致思路如下：
<br/>
-   使用 jenkins 打包（为了安全，jenkins只负责打包，不负责推包）
    -   可以使用spring boot打包方式，把所有东西打成一个很大的jar包
    -   也可以使用assembly方式处理非springboot项目，然后打成tar.gz
-   调用部署脚本推包
    -   这一步牛逼一点可以写一个界面，然后点击按钮触发推包，但是其实底层还是执行脚本。
-   运行jar包
    - 这一步 可以使用启动脚本（牛逼点可以写个界面）。
    - 可以蓝绿发布，新版本完全启动新的机器（买ECS，安装镜像系统，推包，启动脚本启动）
    - 可以再已有的ECS上发布系统，启动脚本
    - 配置守护进行比如 supervisord

下面我们完整的走一遍

##   assembly 打包

### pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>devops</artifactId>
        <groupId>com.moheqionglin</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>devops-deploy</artifactId>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <tar.finalName>moheqionglin-${project.version}</tar.finalName>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.moheqionglin</groupId>
            <artifactId>devops-service</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
    <build>
        <finalName>${tar.finalName}</finalName>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.4</version>
                <executions>
                    <execution>
                    </execution>
                    <execution>
                        <id>make-assemble</id>
                        <goals>
                            <goal>single</goal>
                        </goals>
                        <phase>package</phase>
                        <configuration>
                            <appendAssemblyId>false</appendAssemblyId>
                            <attach>false</attach>
                            <descriptors>
                                <descriptor>assembly-package.xml</descriptor>
                            </descriptors>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>

```  

```
<?xml version="1.0" encoding="UTF-8"?>
<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2 http://maven.apache.org/xsd/assembly-1.1.2.xsd">

<!--    id 标识符，添加到生成文件名称的后缀符。如果指定 id 的话，目标文件则是 ${artifactId}-${id}.tar.gz-->
    <id>bin-tgz</id>

<!-- 支持的打包格式有zip、tar、tar.gz (or tgz)、tar.bz2 (or tbz2)、jar、dir、war，可以同时指定多个打包格式-->
    <formats>
        <format>tar.gz</format>
    </formats>

    <!--   tar -zxvf 以后的目录名字-->
    <baseDirectory>devops</baseDirectory>
<!--    <includeBaseDirectory>true</includeBaseDirectory>-->
<!--
        用来定制工程依赖 jar 包的打包方式，核心元素如下：
        outputDirectory String 指定包依赖目录，该目录是相对于根目录
        includes/include*   List<String>	包含依赖
        excludes/exclude*   List<String>	排除依赖
 -->
    <dependencySets>
        <dependencySet>
            <useProjectArtifact>false</useProjectArtifact>
            <outputDirectory>lib</outputDirectory>
            <scope>runtime</scope>
            <excludes>
                <exclude>junit*</exclude>
            </excludes>
        </dependencySet>
    </dependencySets>

<!--
    管理一组文件的存放位置，核心元素如下
     outputDirectory String 指定包依赖目录，该目录是相对于根目录
     includes/include*   List<String>	包含依赖
     excludes/exclude*   List<String>	排除依赖

-->
    <fileSets>
<!--        相当于 mv src/deploy bin, chmod bin 0755-->
		<fileSet>
			<directory>src/deploy</directory>
            <includes>
                <include>*.conf</include>
                <include>*.sh</include>
            </includes>
			<outputDirectory>deploy</outputDirectory>
			<fileMode>0755</fileMode>
			<directoryMode>0755</directoryMode>
		</fileSet>
<!--        相当于 mv src/config config, chmod bin 0755-->
		<fileSet>
			<directory>src/config</directory>
            <includes>
                <include>*.properties</include>
            </includes>
			<outputDirectory>config</outputDirectory>
		</fileSet>
        <fileSet>
            <directory>src/run</directory>
            <outputDirectory>run</outputDirectory>
        </fileSet>
    </fileSets>
</assembly>


```

## jvm 参数

[jvm参数和启动脚本见本链接][1]

```


```

[1]: https://github.com/moheqionglin/spring-demo/tree/master/devops/devops-deploy/src/deploy
