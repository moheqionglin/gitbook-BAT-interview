自从有了微服务，貌似日志收集，全链路追踪，服务探活，业务监控等名词一个个全部冒上了。 其实技术本身没有什么难度，网上也有一大堆现成的开源框架直接使用。 
<br>
难就难在使用过程中的配置，采坑，性能等东西。本文列举一种生产级别轻量级日志采集的工具-FileBeat。
<br>

![fileBeat][1]


Filebeat主要由两个组件构成：prospector（探测器）和harvester（收集器），这两类组件一起协作完成Filebeat的工作。
 * prospector（探测器）" 当开启Filebeat程序的时候，它会启动一个或多个探测器去检测指定的日志目录或文件，对于探测器找出的每一个日志文件，Filebeat会启动收集进程，每一个收集进程读取一个日志文件的内容，然后将这些日志数据发送到后台处理程序，后台处理程序会集合这些事件
 * input : 输入
 * harvester : A harvester is responsible for reading the content of a single file. The harvester reads each file, line by line, and sends the content to the output. One harvester is started for each file. 如果想控制harvester关闭可以配置close_*
 * output: 输出 
 
## 安装

[file beat 简介][2]  
[file beant下载地址][3]

```
tar -zxvf filebeat-7.6.1-linux-x86_64.tar
mv filebeat-7.6.1-linux-x86_64 filebeat
cd filebeat

```

## 启动、停止

```

# 线上环境配合error级别使用
nohup ./filebeat -e -c filebeat.yml &
# 停止
ps -ef | grep filebeat
Kill -9 线程号

```

## 配置
[file beat 配置][4]
 
 * input :  支持 Multiline messages（java堆栈信息）， Log（普通的日志文件）， Stdin（标准输入），Container（docker日志），Docker（配置docker的 containerID），Kafka(kafka的某个topic)， Redis（redis），UDP（从网络读取），TCP（从网络读取）， Syslog，s3，NetFlow，Google Pub/Sub，Azure eventhub
 * output: Elasticsearch, Logstash, Kafka, Redis, File, Console, Elastic Cloud,  Change the output codec
 * Live reload： 热加载配置。 注意这个不能够热加载filebeat.yml 主配置。


```
vi filebeat.yml 

filebeat.inputs:
- type: log
  enabled: true
  tail_files: True
  paths:
    - /var/log/system.log
    - /var/log/wifi.log
    - /opt/data/logs/*.log
    fields:
        kafka_topic: smOrderServiceTopic
        appName: smOrderService
        ip: 192.168.0.1
        env: prod
  multiline.pattern: '^\d{4}\-\d{2}\-\d{2}'
  multiline.negate: true
  multiline.match: after
- type: log
  paths:
    - "/var/log/apache2/*"
  fields:
    apache: true
  fields_under_root: true

output.kafka:
  hosts: ["192.168.0.1:9092", "192.168.0.2:9092", "192.168.0.3:9092"]
  topic: '%{[fields.kafka_topic]}'
  partition.round_robin:
  reachable_only: false
  required_acks: 1
  compression: gzip
  max_message_bytes: 1000000
    
# 热加载 configs下面的配置
filebeat.config.inputs:
  enabled: true
  path: configs/*.yml
  reload.enabled: true
  reload.period: 10s

processors:
  - add_host_metadata: ~
  - add_cloud_metadata: ~
  - script:
    lang: javascript
    id: my_filter
    params:
      threshold: 15
    source: >
      function process(event) {
          if (event.Get("event.code") === 1102) {
              event.Put("event.action", "cleared");
          }
      }
```

[1]: ../images/devops/filebeat.jpg
[2]: https://elkguide.elasticsearch.cn/beats/file.html
[3]: https://www.elastic.co/downloads/beats
[4]: https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-configuration.html