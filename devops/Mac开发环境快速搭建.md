## Mac 几种命令替换

用过Mac都知道有些Mac的命令跟 linux命令有一定的区别。比如 Mac 不支持 ll， Mac的 sed跟linux的 sed用法差别比较大。下面解决这些问题。

```
brew install gnu-sed

vi ~/.bash_profile
alias sed=gsed
alias ll=ls -al

```

## Mac 开发Java项目默认配置全家桶

### [多版本java支持][1]

```
export M2_HOME="/usr/local/selfApp/apache-maven-3.5.3"
export PATH="$M2_HOME/bin:$PATH"
export HADOOP_HOME=/usr/local/selfApp/hadoop-2.8.5
export PATH=$PATH:$HADOOP_HOME/sbin
export PATH=$PATH:$HADOOP_HOME/bin 

export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
export HBASE_HOME=/usr/local/selfApp/hbase-1.3.3 && export PATH=$HBASE_HOME/bin:$PATH
export GEOMESA_HOME=/Users/wanli.zhou/Workspace/geomesa-hbase_2.11-2.2.1
export GEOMESA_HBASE_HOME=/Users/wanli.zhou/Workspace/geomesa-hbase_2.11-2.2.1
export PATH=$GEOMESA_HOME/bin:$PATH
export NVM_DIR="$HOME/.nvm"
export SCALA_HOME=/usr/local/selfApp/scala-2.12.8
export PATH=$PATH:$SCALA_HOME/bin
export GO_HOME=/usr/local/go
export GRADLE_HOME=/usr/local/selfApp/gradle-4.9
export PATH=$PATH:$GO_HOME/bin:$GRADLE_HOME/bin

```


## mysql, postgresql, redis, mongo, es, dubbo-admin, hbase, hadoop, geomesa, flink, sprark docker运行

工程化开发要处理太多的开源组件，还好有docker帮我们解决这个问题。
1. 做好端口映射规划。
2. 做好数据目录映射， data目录，log目录，jvm日志目录。

|  程序   | dockerFile  |  docker镜像   | 目录映射  | 端口映射  |
|  ----  | ----  |  ----  | ---- | ---- |
| zookeeper  | - | docker search zookeeper  | </conf /data /datalog> | 2181:2181 |
| dubbo-admin  | [dubbo-admin:0.2.0 Dockerfile][2] | [dubbo-admin:0.2.0 image][3]  | 暂无 | 8090:8080 |
| mysql  | [mysql Dockerfile][2] | [mysql image][3]  |   |   |
| redis  | [redis Dockerfile][2] | [redis image][3]  |   |   |
| mongo  | [mongo Dockerfile][2] | [mongo image][3]  |   |   |
| es  | [es Dockerfile][2] | [es image][3]  |   |   |
| hbase,geomesa  | [hbase ,geomesa Dockerfile][2] | [hbase,geomesa image][3]  |   |   |
| hadoop  | [hadoop Dockerfile][2] | [hadoop image][3]  |   |   |
| spark  | [spark Dockerfile][2] | [spark image][3]  |   |   |
| flink  | [flink Dockerfile][2] | [flink image][3]  |   |   |
| kafka  | [kafka Dockerfile][2] | [kafka image][3]  |   |   |
| kafk-admin  | [kafk-admin Dockerfile][2] | [kafk-admin image][3]  |   |   |







[1]: http://www.moheqionglin.com/site/blogs/69/detail.html
[2]: https://github.com/moheqionglin/spring-demo/blob/master/devops/devops-service/src/main/resources/dubbo/Dockerfile
[3]: https://hub.docker.com/repository/docker/moheqionglin/dubbo-admin