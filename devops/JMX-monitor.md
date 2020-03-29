本文介绍一个基于小米开源的Open-Falcon的监控工具来收集JMX监控信息。
## 相关链接

[资源下载][1]
[中文文档][2]
[官网][3]
[官方安装文档][4]
[docker 安装][5]

### 安装过程
因为官方文档已经很详细，这里只列举 提纲，详细操作可以看[官方安装文档][4]。
 * 安装并初始化mysql表
 * 安装并启动redis
 * 安装agent，并启动 nohup ./falcon-agent -c cfg.json &
 * 按需安装falcon服务端组件

### 配置
```
vi agent/cfg.json 
{
    "debug": false,
    "hostname": "sm-server-1",
    "ip": "192.168.0.1",
    "plugin": {
        "enabled": true,
        "dir": "./plugin",
        "git": "https://github.com/open-falcon/plugin.git",
        "logs": "./logs"
    },
    "heartbeat": {
        "enabled": true,
        "addr": "192.168.0.2:8130",
        "interval": 60,
        "timeout": 1000
    },
    "transfer": {
        "enabled": true,
        "addrs": [
            "192.168.0.2:8133"
        ],
        "interval": 60,
        "timeout": 1000
    },
    "http": {
        "enabled": true,
        "listen": ":3188",
        "backdoor": false
    },
    "collector": {
        "ifacePrefix": ["eth", "em"]
    },
    "ignore": {
    }
}

```
### 基础监控 监控项目

|  Counters   | Type  | Notes |
|  ----  | ----  | ----  |
| cpu.busy |	GAUGE |	当前活跃线程数 |
| cpu（iowait，user，system） |	GAUGE |	一分钟内，平均每次YoungGC的新生代内存晋升大小 |
| system load |	GAUGE |	峰值线程数 |
| memory free radio |	GAUGE |	内存空闲百分比 |
| old.gen.mem.ratio |	GAUGE |	老年代的内存使用率 |
| disk ratio |	GAUGE |	磁盘使用占比 |
| inode ratio |	GAUGE |	inode使用占比 |
| disk.io |	GAUGE |	磁盘每分钟磁盘读写字节大小 |
| TCP connection status | GAUGE | tcp链接状态，ss.estab, ss.timewait, ss.ss.slabinfo.timewait, ss.orphaned, ss.closed |
| tcp throughput |	GAUGE |	网络进，出流量 |
| open files |	GAUGE |	内核文件打开个数 |



## JMX 监控 
[下载 jmxmon][6] 

```
vi jmxmon/conf.properties
# the working dir
workDir=./

# localhost jmx ports, split by comma
jmx.ports=10320

# agent port url
agent.posturl=http://localhost:1988/v1/push
hostname=host-1

```

```

nohup java -cp ./jmxmon-0.0.2-jar-with-dependencies.jar com.stephan.tof.jmxmon.JMXMonitor conf.properties &

```


### 监控明细

详情见 [Linux运维基础采集项][7]

|  Counters   | Type  | Notes |
|  ----  | ----  | ----  |
| thread.active.count |	GAUGE |	当前活跃线程数 |
| thread.peak.count |	GAUGE |	峰值线程数 |
| gc.throughput |	GAUGE |	GC的总吞吐率（应用运行时间/进程总运行时间） |
| old.gen.mem.ratio |	GAUGE |	老年代的内存使用率 |
| concurrentmarksweep.gc.avg.time |	GAUGE |	一分钟内，每次CMSGC的平均耗时 |
| parnew.gc.count |	GAUGE |	一分钟内，YoungGC(parnew)的总次数 |
| concurrentmarksweep.gc.count |	GAUGE |	一分钟内，CMSGC的总次数 |
| parnew.gc.avg.time | GAUGE | 一分钟内，每次YoungGC(parnew)的平均耗时 |
| old.gen.mem.used |	GAUGE |	老年代的内存使用量 |
| new.gen.promotion |	GAUGE |	一分钟内，新生代的内存晋升总大小 |
| new.gen.avg.promotion |	GAUGE |	一分钟内，平均每次YoungGC的新生代内存晋升大小 |

具体数据包如下：

```
[
    {
        "metric":"parnew.gc.avg.time",
        "endpoint":"sm-service-server1",
        "timestamp":1584413025,
        "step":60,
        "value":16,
        "counterType":"GAUGE",
        "tags":"jmxport=10100"
    },
    {
        "metric":"parnew.gc.count",
        "endpoint":"sm-service-server1",
        "timestamp":1584413025,
        "step":60,
        "value":2,
        "counterType":"GAUGE",
        "tags":"jmxport=10100"
    },
    {
        "metric":"concurrentmarksweep.gc.avg.time",
        "endpoint":"sm-service-server1",
        "timestamp":1584413025,
        "step":60,
        "value":0,
        "counterType":"GAUGE",
        "tags":"jmxport=10100"
    },
    {
        "metric":"concurrentmarksweep.gc.count",
        "endpoint":"sm-service-server1",
        "timestamp":1584413025,
        "step":60,
        "value":0,
        "counterType":"GAUGE",
        "tags":"jmxport=10100"
    },
    {
        "metric":"gc.throughput",
        "endpoint":"sm-service-server1",
        "timestamp":1584413025,
        "step":60,
        "value":99.94,
        "counterType":"GAUGE",
        "tags":"jmxport=10100"
    },
    {
        "metric":"old.gen.mem.used",
        "endpoint":"sm-service-server1",
        "timestamp":1584413025,
        "step":60,
        "value":113427096,
        "counterType":"GAUGE",
        "tags":"jmxport=10100"
    },
    {
        "metric":"old.gen.mem.ratio",
        "endpoint":"sm-service-server1",
        "timestamp":1584413025,
        "step":60,
        "value":11.3,
        "counterType":"GAUGE",
        "tags":"jmxport=10100"
    },
    {
        "metric":"new.gen.promotion",
        "endpoint":"sm-service-server1",
        "timestamp":1584413025,
        "step":60,
        "value":64,
        "counterType":"GAUGE",
        "tags":"jmxport=10100"
    },
    {
        "metric":"new.gen.avg.promotion",
        "endpoint":"sm-service-server1",
        "timestamp":1584413025,
        "step":60,
        "value":32,
        "counterType":"GAUGE",
        "tags":"jmxport=10100"
    },
    {
        "metric":"thread.active.count",
        "endpoint":"sm-service-server1",
        "timestamp":1584413025,
        "step":60,
        "value":101,
        "counterType":"GAUGE",
        "tags":"jmxport=10100"
    },
    {
        "metric":"thread.peak.count",
        "endpoint":"sm-service-server1",
        "timestamp":1584413025,
        "step":60,
        "value":135,
        "counterType":"GAUGE",
        "tags":"jmxport=10100"
    }
]

```


[1]: https://github.com/open-falcon/falcon-plus/releases
[2]: https://book.open-falcon.org/zh_0_2/
[3]: http://open-falcon.org/
[4]: https://book.open-falcon.org/zh_0_2/distributed_install/
[5]: https://github.com/open-falcon/falcon-plus/blob/master/docker/README.md
[6]: https://github.com/toomanyopenfiles/jmxmon/releases
[7]: http://book.open-falcon.org/zh/faq/linux-metrics.html