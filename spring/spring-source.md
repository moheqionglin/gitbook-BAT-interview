### 下载spring-5.1.x 源码
http://note.youdao.com/ynoteshare1/index.html?id=c3f11aab5e0a0083709cc64984a3c41a

```
# 注意一定要选好branch到 5.1.x，否则会出现本文意外的异常。

```
[https://github.com/spring-projects/spring-framework][1] spring源码下载，如果想快点下载的话，可以直接下载zip文件。

### gradle环境配置和其他环境
* gradle 4.9,高版本的gradle报各种错误。
* jdk 1.8.0_171 (Oracle Corporation 25.171-b11)


**注意，要想顺利的编译spring5的源码，推荐使用 gradle4.x， 要不然会遇到各种问题**

- 下载 gradle包，注意下载-all.zip的文件，[https://services.gradle.org/distributions/][2]
- 解压缩
- Mac环境直接 在 ~/.bash_profile中添加 


```
export GRADLE_HOME=/usr/local/selfApp/gradle-4.10.3
export PATH=$PATH:$GRADLE_HOME/bin

```
- source ~/.bash_profile

```
#验证
# gradle -version


Welcome to Gradle 4.10.3!

```

### 配置IDEA的gradle环境
* 打开首选项 搜索gradle
* 勾选 Use local Gradle distribution
* 配置Gradle home和Gradle JVM
* 配置 Service directory path

### 阅读import-into-idea.md 

```
#1. Precompile `spring-oxm` with `./gradlew :spring-oxm:compileTestJava`
# 在spring目录中执行 ./gradlew :spring-oxm:compileTestJava
./gradlew :spring-oxm:compileTestJava

Downloading https://services.gradle.org/distributions/gradle-4.10.3-bin.zip
..........................................................................
Starting a Gradle Daemon (subsequent builds will be faster)

> Task :spring-beans:compileTestJava
Note: Some input files use or override a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
Note: Some input files use unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.

> Task :spring-context:compileTestJava
Note: Some input files use unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.

> Task :spring-oxm:genCastor
[ant:javac] : warning: 'includeantruntime' was not set, defaulting to build.sysclasspath=last; set to false for repeatable builds

> Task :spring-oxm:genJaxb
[ant:javac] : warning: 'includeantruntime' was not set, defaulting to build.sysclasspath=last; set to false for repeatable builds

BUILD SUCCESSFUL in 3m 11s
59 actionable tasks: 59 executed

```

### 引入源码到IDEA中


```

# 注意一定要选中 build.gradle这个文件。否则出现各种问题。
Import into IntelliJ (File -> New -> Project from Existing Sources -> Navigate to directory -> Select build.gradle)

```

* Use auto-import 打钩
* Use local Gradle distribution 打钩
* 确认 Gradle home 和 Gradle JVM是否是自己的
* Global Gradle settings 配置 -Xmx2048m

然后等待漫长的下载各种依赖的过程。

### 预编译`spring-core` and `spring-oxm`

```
#在IDEA左侧的gradle工具中
spring --> spring-oxm --> Tasks --> compileTestJava
spring --> spring-core --> Tasks --> compileTestJava

```
### 终极编译

注释掉Could not resolve all files for configuration ':asciidoctor'.

```
# 找到文件docs.gradle
//dokka {
//	dependsOn {
//		tasks.getByName("api")
//	}
//	doFirst {
//		classpath = subprojects.collect { project -> project.jar.outputs.files.getFiles() }.flatten()
//		classpath += files(subprojects.collect { it.sourceSets.main.compileClasspath })
//
//	}
//	moduleName = "spring-framework"
//	outputFormat = "html"
//	outputDirectory = "$buildDir/docs/kdoc"
//
//	sourceDirs = files(subprojects.collect { project ->
//		def kotlinDirs = project.sourceSets.main.kotlin.srcDirs.collect()
//		kotlinDirs -= project.sourceSets.main.java.srcDirs
//	})
//	externalDocumentationLink {
//		url = new URL("https://docs.spring.io/spring-framework/docs/$version/javadoc-api/")
//		packageListUrl = new File(buildDir, "api/package-list").toURI().toURL()
//	}
//	externalDocumentationLink {
//		url = new URL("https://projectreactor.io/docs/core/release/api/")
//	}
//	externalDocumentationLink {
//		url = new URL("https://www.reactive-streams.org/reactive-streams-1.0.1-javadoc/")
//	}
//}
//
//asciidoctor {
//	sources {
//		include '*.adoc'
//	}
//	resources {
//		from(sourceDir) {
//			include 'images/*', 'stylesheets/*', 'tocbot-3.0.2/*'
//		}
//	}
//	logDocuments = true
//	backends = ["html5"]
//	// only ouput PDF documentation for non-SNAPSHOT builds
//	if(!project.getVersion().toString().contains("BUILD-SNAPSHOT")) {
//		backends += "pdf"
//	}
//	options doctype: 'book', eruby: 'erubis'
//	attributes  'icons': 'font',
//			'idprefix': '',
//			'idseparator': '-',
//			docinfo: '',
//			revnumber: project.version,
//			sectanchors: '',
//			sectnums: '',
//			'source-highlighter': 'coderay@', // TODO switch to 'rouge' once supported by the html5 backend
//			stylesdir: 'stylesheets/',
//			stylesheet: 'main.css',
//			'spring-version': project.version
//
//}


//
//asciidoctor {
//	sources {
//		include '*.adoc'
//	}
//	resources {
//		from(sourceDir) {
//			include 'images/*', 'stylesheets/*', 'tocbot-3.0.2/*'
//		}
//	}
//	logDocuments = true
//	backends = ["html5"]
//	// only ouput PDF documentation for non-SNAPSHOT builds
//	if(!project.getVersion().toString().contains("BUILD-SNAPSHOT")) {
//		backends += "pdf"
//	}
//	options doctype: 'book', eruby: 'erubis'
//	attributes  'icons': 'font',
//			'idprefix': '',
//			'idseparator': '-',
//			docinfo: '',
//			revnumber: project.version,
//			sectanchors: '',
//			sectnums: '',
//			'source-highlighter': 'coderay@', // TODO switch to 'rouge' once supported by the html5 backend
//			stylesdir: 'stylesheets/',
//			stylesheet: 'main.css',
//			'spring-version': project.version
//
//}

```


```
#直接编译，不用管网上说 spring-aop会因为什么aspectj原因编译不通不过。
spring --> Tasks --> build
# 时间会持续20分钟
```


[1]: https://github.com/spring-projects/spring-framework
[2]: https://services.gradle.org/distributions/
[3]: ../images/spring/spring-source.jpg