# 教程：压缩段

本教程演示了如何将现有段压缩为更少但更大的段。

由于存在一些每段内存和处理开销，因此减少段的总数有时是有益的。

对于本教程，我们假设您已经按照[单机快速入门](http://druid.io/docs/0.12.3/tutorials/index.html)中的描述下载了Druid，并让它在本地计算机上运行。

完成[教程：加载文件](http://druid.io/docs/0.12.3/tutorials/tutorial-batch.html)和[教程：查询数据](http://druid.io/docs/0.12.3/tutorials/tutorial-query.html)也很有帮助。

## 加载初始数据

在本教程中，我们将使用Wikipedia编辑样本数据，其中包含摄取任务规范，该规范将为输入数据中的每个小时创建一个单独的段。

摄取规范可以在`examples/compaction-init-index.json`。让我们提交该规范，它将创建一个名为的数据源`compaction-tutorial`：

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/compaction-init-index.json http://localhost:8090/druid/indexer/v1/task
```

摄取完成后，在浏览器中转到http：// localhost：8081 /＃/ datasources / compaction-tutorial，以在Coordinator控制台中查看有关新数据源的信息。

此数据源将有24个段，输入数据中每小时一个段：

![原始细分](http://druid.io/docs/0.12.3/tutorials/img/tutorial-retention-01.png)

对此数据源运行COUNT（*）查询显示有39,244行：

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/compaction-count-sql.json http://localhost:8082/druid/v2/sql
[{"EXPR$0":39244}]
```

## 压缩数据

现在让我们将这24个细分组合成一个细分。

我们在本教程数据源中包含了一个压缩任务规范`examples/compaction-final-index.json`：

```json
{
  "type": "compact",
  "dataSource": "compaction-tutorial",
  "interval": "2015-09-12/2015-09-13",
  "tuningConfig" : {
    "type" : "index",
    "targetPartitionSize" : 5000000,
    "maxRowsInMemory" : 25000,
    "forceExtendableShardSpecs" : true
  }
}
```

这将压缩所有段的间隔`2015-09-12/2015-09-13`在`compaction-tutorial`数据源。

`tuningConfig`控制中的参数将在压缩的一组段中存在多少段。

在本教程示例中，将只创建一个压缩段，因为输入中的39244行小于5000000 `targetPartitionSize`。

我们现在提交此任务：

```json
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/compaction-final-index.json http://localhost:8090/druid/indexer/v1/task
```

任务完成后，刷新http：// localhost：8081 /＃/ datasources / compaction-tutorial页面。

最初的24个段最终将被协调员标记为“未使用”并被删除，新的压缩段将保留。

默认情况下，在协调员进程启动至少15分钟之前，德鲁伊协调员不会将段标记为未使用，因此您可以在协调器中同时看到旧的段集和新的压缩集，例如：

![压缩段中间状态](http://druid.io/docs/0.12.3/tutorials/img/tutorial-compaction-01.png)

新的压缩段具有比原始段更新的版本，因此即使协调器显示两组段，查询也只会从新的压缩段读取。

让我们再次尝试运行COUNT（*）`compaction-tutorial`，行数仍应为39,244：

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/compaction-count-sql.json http://localhost:8082/druid/v2/sql
[{"EXPR$0":39244}]
```

协调器运行至少15分钟后，http：// localhost：8081 /＃/ datasources / compaction-tutorial页面应显示只有1个段：

![压缩段最终状态](http://druid.io/docs/0.12.3/tutorials/img/tutorial-compaction-02.png)

## 进一步阅读

[任务文档](http://druid.io/docs/0.12.3/ingestion/tasks.html)

[细分优化](http://druid.io/docs/0.12.3/operations/segment-optimization.html)