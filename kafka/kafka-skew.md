![kafka 各种倾斜问题解决方法][1]

上图有三个问题：
1. Brokers Spread
2. Brokers Skew
3. Brokers Leader Skew

### Brokers Spread
Brokers Spread：看作broker使用率，如kafka集群9个broker，某topic有7个partition，则broker spread: 7 / 9 = 77%


### Brokers Skew
Brokers Skew：partition是否存在倾斜，如kafka集群9个broker，某topic有18个partition，正常每个broker应该2个partition。若其中有3个broker上的partition数>2，则broker skew:  3 / 9 = 33%

### Brokers Leader Skew 问题解决方法

由于kafka所有读写都在leader上进行， broker leader skew会导致不同broker的读写负载不均衡，配置参数 auto.leader.rebalance.enable=true 可以使kafka每5min自动做一次leader的rebalance，消除这个问题。

![kafka leader skew问题解决方法][2]

```
https://cwiki.apache.org/confluence/display/KAFKA/Replication+tools#Replicationtools-1.PreferredReplicaLeaderElectionTool

bin/kafka-preferred-replica-election.sh --zookeeper 172.16.224.28:2181,172.16.224.29:2181,172.16.224.30:2181/kafka --path-to-json-file ./topicPartitionList.json


{
 "partitions":
  [
    {"topic": "point-topic", "partition": 2},
    {"topic": "point-topic", "partition": 3},
    {"topic": "point-topic", "partition": 8},
    {"topic": "point-topic", "partition": 14},
    {"topic": "point-topic", "partition": 15},
    {"topic": "point-topic", "partition": 20},
    {"topic": "point-topic", "partition": 26},
    {"topic": "point-topic", "partition": 27}
  ]
}

```
### Lag

Lag代表consumer的消费能力，计算公式为Lag = Consumer Offset - LogSize，Kafka Manager先从zk获取LogSize，再从kafka __consumer_offsets topic读取Offset。两步操作存在一个时间gap，因此吞吐很大的topic上会出现Offset > LogSize的情况。导致Lag负数。


[1]: ../images/kafka/kafka-skew.jpg
[2]: ../images/kafka/broker-leader-skew.png