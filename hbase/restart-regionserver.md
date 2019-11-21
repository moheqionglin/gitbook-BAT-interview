1. 重启一个regionserver

```
bin/graceful_stop.sh --restart --reload --debug regionserver_nodename

# 然后重新开启balance
hbase shell
balance_switch true
```

这个操作是平滑的重启regionserver进程，对服务不会有影响，他会先将需要重启的regionserver上面的所有region迁移到其它的服务器，然后重启，最后又会将之前的region迁移回来，但我们修改一个配置时，可以用这种方式重启每一台机子，这个命令会关闭balancer，所以最后我们要在hbase shell里面执行一下balance_switch true，对于hbase regionserver重启，不要直接kill进程，这样会造成在zookeeper.session.timeout这个时间长的中断，也不要通过bin/hbase-daemon.sh stop regionserver去重启，如果运气不太好，-ROOT-(目前root表已经删除)或者hbase:meta表在上面的话，所有的请求会全部失败。

2. 关闭下线一台regionserver

```
bin/graceful_stop.sh --stop  regionserver_nodename


# 然后重新开启balance
hbase shell
balance_switch true
```

和上面一样，系统会在关闭之前迁移所有region，然后stop进程，同样最后我们要手工balance_switch true，开启master的region均衡。
