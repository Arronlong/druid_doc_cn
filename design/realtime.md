# 实时节点

注意：不推荐使用实时节点。请使用[Kafka Indexing Service](http://druid.io/docs/0.12.3/development/extensions-core/kafka-ingestion.html)代替流式拉取用例。

有关实时节点配置，请参阅[实时配置](http://druid.io/docs/0.12.3/configuration/realtime.html)。

对于实时摄取，请参阅[实时摄取](http://druid.io/docs/0.12.3/ingestion/stream-ingestion.html)。

实时节点提供实时索引。通过这些节点索引的数据可立即用于查询。实时节点将定期构建表示他们在某段时间内收集的数据的段，并将这些段传输到[Historical](http://druid.io/docs/0.12.3/design/historical.html)节点。他们使用ZooKeeper监控传输和元数据存储，以存储有关传输段的元数据。转移后，Realtime节点会忘记段。

### 运行

```text
io.druid.cli.Main server realtime
```

## 细分传播

实时数据摄取的段传播图如下所示：

![细分传播](http://druid.io/docs/img/segmentPropagation.png)

您可以在“体系结构”部分下阅读此图中显示的各种组件（请参阅右侧菜单）。请注意，有些名称现已过时。

### 消防水带

见[Firehose](http://druid.io/docs/0.12.3/ingestion/firehose.html)。

### 水管工人

见[水管工](http://druid.io/docs/0.12.3/design/plumber.html)

## 扩展代码

实时集成旨在以两种方式进行扩展：

1. 连接来自不同系统的数据流（[Firehose](https://github.com/druid-io/druid-api/blob/master/src/main/java/io/druid/data/input/FirehoseFactory.java)）
2. 调整发布策略以满足您的需求（[水管工](https://github.com/druid-io/druid/blob/master/server/src/main/java/io/druid/segment/realtime/plumber/PlumberSchool.java)）

期望是前者将是非常普遍的，德鲁伊的用户将会定期做的事情。大多数用户可能永远不必处理后一种自定义形式。实际上，我们希望所有潜在的用例都可以作为德鲁伊本身的一部分进行打包，而无需专有的定制。

鉴于这些期望，添加一个firehose是直截了当的，并完全封装在界面内。添加水管工更为复杂，需要了解系统如何正确运行，这并非不可能，但并不是新手德鲁伊的人能够立即做到这一点。

## HTTP端点

实时节点公开了几个HTTP端点以进行交互。

### 得到

- `/status`

返回Druid版本，加载的扩展，使用的内存，总内存和有关节点的其他有用信息。