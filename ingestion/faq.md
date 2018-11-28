## 我的数据未加载

### 实时摄取

造成这种情况的最常见原因是因为摄入的事件是德鲁伊的带外事件`windowPeriod`。德鲁伊实时摄取仅接受当前时间的可配置windowPeriod内的事件。您可以通过查看包含“ingest / events / *”的日志行的实时进程日志来验证这是发生了什么。这些指标将指示摄取，拒绝等事件。我们建议对生产中的历史数据使用批量摄取方法。

### 批量摄取

如果您尝试批量加载历史数据但未加载任何事件，请确保提取规范的间隔实际上封装了数据的间隔。超出此间隔的事件将被删除。

## 德鲁伊支持哪些类型的数据？

德鲁伊可以开箱即用地摄取JSON，CSV，TSV和其他分隔数据。Druid支持单维值或多维值（字符串数组）。Druid支持long，float和double数字列。

## 并非我的所有活动都被摄取了

德鲁伊将拒绝窗口期之外的事件。查看事件是否被拒绝的最佳方法是检查[德鲁伊摄取指标](http://druid.io/docs/0.12.3/operations/metrics.html)。

如果摄取事件的数量似乎正确，请确保您的查询正确形成。如果`count`在摄取规范中包含聚合器，则需要使用`longSum`聚合器查询此聚合的结果。使用计数聚合器发出查询将计算德鲁伊行的数量，其中包括[汇总](http://druid.io/docs/0.12.3/design/index.html)。

## 摄取后，我的德鲁伊片段会在哪里结束？

根据`druid.storage.type`设置的内容，Druid会将段上传到某些[Deep Storage](http://druid.io/docs/0.12.3/dependencies/deep-storage.html)。本地磁盘用作默认深层存储。

## 我的流摄取不会关闭段

首先，确保摄取过程的日志中没有例外。还要确保将`druid.storage.type`其设置为深层存储，`local`如果您正在运行分布式群集，则不会。

交接失败的其他常见原因如下：

1）德鲁伊无法写入元数据存储。确保您的配置正确无误。

2）历史节点超出容量，无法再下载任何段。如果发生这种情况，您将在协调器日志中看到异常，协调器控制台将显示历史记录接近容量。

3）细分已损坏且无法下载。如果发生这种情况，您将在历史节点中看到异常。

4）深度存储配置不正确。确保您的段实际存在于深层存储中，并且协调器日志没有错误。

## 我如何让HDFS工作？

确保在类路径中包含`druid-hdfs-storage`所有hadoop配置，依赖项（可以通过`hadoop classpath`在已设置hadoop的机器上运行命令获得）。并且，按照[Deep Storage中的说明](http://druid.io/docs/0.12.3/dependencies/deep-storage.html)提供必要的HDFS设置。

## 我没有在我的历史节点上看到我的德鲁伊片段

您可以查看位于的协调器控制台`<COORDINATOR_IP>:<PORT>`。确保您的段实际上已加载到[历史节点上](http://druid.io/docs/0.12.3/design/historical.html)。如果您的段不存在，请检查协调器日志以获取有关复制错误容量的消息。不下载段的一个原因是因为历史节点的maxSize太小，使得它们无法下载更多数据。您可以使用（例如）更改它：

```text
-Ddruid.segmentCache.locations=[{"path":"/tmp/druid/storageLocation","maxSize":"500000000000"}]
-Ddruid.server.maxSize=500000000000
```

## 我的查询返回空结果

您可以对已为数据源创建的维度和指标使用[段元数据查询](http://druid.io/docs/0.12.3/querying/segmentmetadataquery.html)。确保您在查询中使用的聚合器的名称与这些指标之一匹配。还要确保指定的查询间隔与数据存在的有效时间范围匹配。

## 如何使用架构更改重新编制德鲁伊中的现有数据？

您可以将IngestSegmentFirehose与索引任务一起使用新架构来摄取现有的德鲁伊段，并更改段的名称，维度，指标，汇总等。有关[IngestSegmentFirehose](http://druid.io/docs/0.12.3/ingestion/firehose.html)的更多详细信息，请参阅Firehose。或者，如果您使用基于hadoop的摄取，那么您可以使用“dataSource”输入规范来进行重建索引。

有关详细信息，请参阅[更新现有数据](http://druid.io/docs/0.12.3/ingestion/update-existing-data.html)

## 如何更改Druid中现有数据的粒度？

在很多情况下，您可能希望降低旧数据的粒度。例如，任何超过1个月的数据都只有小时级别的粒度，但较新的数据具有分钟级别的粒度。此用例与重新索引相同。

为此，请使用IngestSegmentFirehose并运行索引器任务。IngestSegment firehose将允许您从Druid中获取现有的段并将它们聚合并将它们反馈给Druid。它还允许您在重新输入时过滤这些段中的数据。这意味着如果有要删除的行，则可以在重新摄取期间过滤掉它们。通常情况下，上述操作将作为批处理作业运行，以表示每天数据块中的日常数据并对其进行汇总。或者，如果您使用基于hadoop的摄取，那么您可以使用“dataSource”输入规范来进行重建索引。

有关详细信息，请参阅[更新现有数据](http://druid.io/docs/0.12.3/ingestion/update-existing-data.html)

## 实时摄取似乎被卡住了

有几种方法可以实现。如果中间体持续时间过长或者交接时间太长，德鲁伊会遏制摄入以防止内存不足问题。如果您的节点日志指示某些列需要很长时间才能构建（例如，如果您的段粒度为每小时，但创建单个列需要30分钟），则应重新评估配置或扩展实时摄入。

## 更多信息

对于初次使用的用户来说，将数据输入德鲁伊绝对是困难的。请不要犹豫，在我们的IRC频道或我们的[Google](https://groups.google.com/forum/#!forum/druid-user)网上论坛[页面](https://groups.google.com/forum/#!forum/druid-user)上提问。