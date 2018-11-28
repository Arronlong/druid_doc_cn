# 压实任务

压缩任务合并给定间隔的所有段。语法是：

```json
{
    "type": "compact",
    "id": <task_id>,
    "dataSource": <task_datasource>,
    "interval": <interval to specify segments to be merged>,
    "dimensions" <custom dimensionsSpec>,
    "tuningConfig" <index task tuningConfig>,
    "context": <task context>
}
```

| 领域           | 描述                                                         | 需要 |
| -------------- | ------------------------------------------------------------ | ---- |
| `type`         | 任务类型。应该`compact`                                      | 是   |
| `id`           | 任务ID                                                       | 没有 |
| `dataSource`   | 要压缩的dataSource名称                                       | 是   |
| `interval`     | 要压缩的段的间隔                                             | 是   |
| `dimensions`   | 自定义尺寸规格。压缩任务将使用此维度规范（如果存在）而不是生成一个。请参阅下面的更多细节。 | 没有 |
| `tuningConfig` | [索引任务tuningConfig](http://druid.io/docs/0.12.3/ingestion/native-batch.html#tuningconfig) | 没有 |
| `context`      | [任务上下文](http://druid.io/docs/0.12.3/ingestion/locking-and-priority.html#task-context) | 没有 |

压缩任务的一个例子是

```json
{
  "type" : "compact",
  "dataSource" : "wikipedia",
  "interval" : "2017-01-01/2018-01-01"
}
```

此压缩任务读取间隔的*所有段*`2017-01-01/2018-01-01`并生成新段。请注意，`2017-01-01/2018-01-01`无论segmentGranularity是什么，输入段的间隔都会合并为一个区间。要控制结果段的数量，您可以设置`targetPartitionSize`或`numShards`。有关更多详细信息，请参见[indexTuningConfig](http://druid.io/docs/0.12.3/ingestion/native-batch.html#tuningconfig)。要将每天的数据合并到不同的细分中，您可以提交多个`compact`任务，每天一个。它们将并行运行。

压缩任务在内部生成`index`任务规范，用于使用一些固定参数执行压缩工作。例如，其`firehose`始终是[ingestSegmentSpec](http://druid.io/docs/0.12.3/ingestion/firehose.html#ingestsegmentfirehose)，和`dimensionsSpec`与`metricsSpec` 包括在默认情况下输入段的所有维度和指标。

如果您指定的间隔没有加载数据段（或者您指定的间隔为空），压缩任务将以失败状态代码退出，而不执行任何操作。

除非所有输入段具有相同的元数据，否则输出段可以具有与输入段不同的元数据。

- 维度：由于德鲁伊支持模式更改，因此即使它们是同一数据源的一部分，维度也可能不同。如果输入段具有不同的维度，则输出段基本上包括输入段的所有维度。但是，即使输入段具有相同的维度集，维度顺序或维度的数据类型也可能不同。例如，可以更改某些维度的数据类型`string`对于原始类型，或者可以更改维度的顺序以获得更好的位置。在这种情况下，根据数据类型和排序，最近段的维度先于旧段的维度。这是因为更新的段更可能具有新的所需订单和数据类型。如果要使用自己的排序和类型，可以`dimensionsSpec`在压缩任务规范中指定自定义。
- 汇总：仅在`rollup`为所有输入段设置时汇总输出段。有关详细信息，请参阅[总览](http://druid.io/docs/0.12.3/ingestion/index.html#rollup)。您可以使用“ [细分数据库查询”](http://druid.io/docs/0.12.3/querying/segmentmetadataquery.html#analysistypes)来检查您的细分是否已汇总。
- 分区：压缩任务是本机批量索引任务的一种特殊形式，因此它始终在整个维度集上使用基于散列的分区。