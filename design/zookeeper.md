# 动物园管理员

Druid使用[ZooKeeper](http://zookeeper.apache.org/)（ZK）来管理当前的集群状态。在ZK上发生的操作是

1. [协调员](http://druid.io/docs/0.12.3/design/coordinator.html)领导人选举
2. 从[历史](http://druid.io/docs/0.12.3/design/historical.html)和[实时](http://druid.io/docs/0.12.3/design/realtime.html)分段“发布”协议
3. [协调器](http://druid.io/docs/0.12.3/design/coordinator.html)和[历史](http://druid.io/docs/0.12.3/design/historical.html)之间的段加载/丢弃协议
4. [霸王](http://druid.io/docs/0.12.3/design/indexing-service.html)领袖选举
5. [索引服务](http://druid.io/docs/0.12.3/design/indexing-service.html)任务管理

### 协调员领导人选举

我们使用Curator LeadershipLatch配方在路径上进行领导者选举

```text
${druid.zk.paths.coordinatorPath}/_COORDINATOR
```

### 从历史和实时分段“发布”协议

在`announcementsPath`与`servedSegmentsPath`被用于此目的。

所有[历史](http://druid.io/docs/0.12.3/design/historical.html)和[实时](http://druid.io/docs/0.12.3/design/realtime.html)节点都在其上发布`announcementsPath`，具体而言，它们将创建一个短暂的znode

```text
${druid.zk.paths.announcementsPath}/${druid.host}
```

这表明它们存在。他们随后还将创建一个永久的znode

```text
${druid.zk.paths.servedSegmentsPath}/${druid.host}
```

当它们加载段时，它们会附加看起来像的短暂znode

```text
${druid.zk.paths.servedSegmentsPath}/${druid.host}/_segment_identifier_
```

然后，[协调器](http://druid.io/docs/0.12.3/design/coordinator.html)和[代理](http://druid.io/docs/0.12.3/design/broker.html)等节点可以观察这些路径，以查看哪些节点当前正在为哪些段服务。

### 协调器和历史之间的段加载/丢弃协议

将`loadQueuePath`用于此。

当[协调器](http://druid.io/docs/0.12.3/design/coordinator.html)决定[历史](http://druid.io/docs/0.12.3/design/historical.html)节点应该加载或删除一个段时，它会将一个短暂的znode写入

```text
${druid.zk.paths.loadQueuePath}/_host_of_historical_node/_segment_identifier
```

此节点将包含一个有效负载，该负载向历史节点指示它应对给定的段执行的操作。当历史节点完成工作时，它将删除znode，以便向协调器表明它已完成。