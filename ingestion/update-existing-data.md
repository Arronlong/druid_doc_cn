# 更新现有数据

在一段时间内在dataSource中摄取一些数据并创建德鲁伊段后，您可能希望对摄取的数据进行更改。有几种方法可以做到这一点。

##### 更新维度值

如果您的维度需要经常更新值，请首先尝试使用[查找](http://druid.io/docs/0.12.3/querying/lookups.html)。查找的经典用例是当您将一个ID维度存储在Druid段中，并希望将ID维度映射到可能需要定期更新的人类可读字符串值。

##### 重建细分（重建索引）

如果查找不充分，您可以在特定的时间间隔内完全重建德鲁伊片段。重建段称为重建索引数据。例如，如果要在现有细分中添加或删除列，或者要更改细分的汇总粒度，则必须重新编制数据索引。

我们建议您保留原始数据的副本，以备您需要重新编制数据索引时使用。

##### 处理延迟事件（Delta摄取）

如果您有批处理提取管道并且有延迟事件进入并希望将这些事件附加到现有段并避免使用重建索引重建新段的开销，则可以使用增量提取。

### 使用Hadoop批量摄取重新索引和Delta摄取

本节假定读者了解如何使用Hadoop进行批量提取。有关更多信息，请参阅 [Hadoop批量提取](http://druid.io/docs/0.12.3/ingestion/hadoop.html)。Hadoop批量摄取可用于重建索引和增量摄取。

德鲁伊使用一个`inputSpec`在`ioConfig`知道在哪里可以摄取数据的位置以及如何读取它。对于简单的Hadoop批量提取，`static`或者`granularity`规范类型允许您读取存储在深存储中的数据。

还有其他类型`inputSpec`可以启用重建索引和增量摄取。

#### `dataSource`

这是一种`inputSpec`读取已经存储在Druid中的数据的类型。

| 领域          | 类型       | 描述                                                         | 需要 |
| ------------- | ---------- | ------------------------------------------------------------ | ---- |
| 类型          | 串。       | 这应该始终是'dataSource'。                                   | 是   |
| ingestionSpec | JSON对象。 | 要装载的德鲁伊段的规格。见下文。                             | 是   |
| maxSplitSize  | 数         | 允许根据段的大小将多个段组合成单个Hadoop InputSplit。使用-1时，德鲁伊根据用户指定的地图任务数量（mapred.map.tasks或mapreduce.job.maps）计算最大分割大小。默认情况下，对一个段进行一次拆分。 | 没有 |

内容如下`ingestionSpec`：

| 领域                 | 类型       | 描述                                                         | 需要 |
| -------------------- | ---------- | ------------------------------------------------------------ | ---- |
| 数据源               | 串         | 从中加载数据的德鲁伊数据源名称。                             | 是   |
| 间隔                 | 名单       | 表示ISO-8601间隔的字符串列表。                               | 是   |
| 段                   | 名单       | 从中读取数据的段列表，默认情况下自动获取。您可以通过在url / druid / coordinator / v1 / metadata / datasources / segments上向协调员发出POST查询来获取要放入此处的段列表？完整的请求paylod中指定的时间间隔列表，例如[“2012-01-01T00： 00：00.000 / 2012-01-03T00：00：00.000“，”2012-01-05T00：00：00.000 / 2012-01-07T00：00：00.000“]。您可能希望手动提供此列表，以确保读取的段与任务提交时的段完全相同，如果用户提供的列表与任务实际运行时的数据库状态不匹配，任务将失败。 | 没有 |
| 过滤                 | JSON       | 见[滤波器](http://druid.io/docs/0.12.3/querying/filters.html) | 没有 |
| 尺寸                 | 字符串数组 | 要加载的维列的名称。默认情况下，列表将从parseSpec构造。如果parseSpec没有明确的维度列表，那么将读取存储数据中存在的所有维度列。 | 没有 |
| 指标                 | 字符串数组 | 要加载的度量标准列的名称。默认情况下，列表将从所有已配置聚合器的“名称”构造。 | 没有 |
| ignoreWhenNoSegments | 布尔       | 如果没有找到段，是否忽略此摄取规则。默认行为是在未找到任何段时抛出错误。 | 没有 |

例如

```json
"ioConfig" : {
  "type" : "hadoop",
  "inputSpec" : {
    "type" : "dataSource",
    "ingestionSpec" : {
      "dataSource": "wikipedia",
      "intervals": ["2014-10-20T00:00:00Z/P2W"]
    }
  },
  ...
}
```

#### `multi`

这是一个组合inputSpec以组合其他inputSpecs。此inputSpec用于增量摄取。请注意，您只能有一个`dataSource`作为`multi`inputSpec的子项。

| 领域 | 类型           | 描述                               | 需要 |
| ---- | -------------- | ---------------------------------- | ---- |
| 孩子 | JSON对象的数组 | 包含其他inputSpecs的JSON对象列表。 | 是   |

例如：

```json
"ioConfig" : {
  "type" : "hadoop",
  "inputSpec" : {
    "type" : "multi",
    "children": [
      {
        "type" : "dataSource",
        "ingestionSpec" : {
          "dataSource": "wikipedia",
          "intervals": ["2012-01-01T00:00:00.000/2012-01-03T00:00:00.000", "2012-01-05T00:00:00.000/2012-01-07T00:00:00.000"],
          "segments": [
            {
              "dataSource": "test1",
              "interval": "2012-01-01T00:00:00.000/2012-01-03T00:00:00.000",
              "version": "v2",
              "loadSpec": {
                "type": "local",
                "path": "/tmp/index1.zip"
              },
              "dimensions": "host",
              "metrics": "visited_sum,unique_hosts",
              "shardSpec": {
                "type": "none"
              },
              "binaryVersion": 9,
              "size": 2,
              "identifier": "test1_2000-01-01T00:00:00.000Z_3000-01-01T00:00:00.000Z_v2"
            }
          ]
        }
      },
      {
        "type" : "static",
        "paths": "/path/to/more/wikipedia/data/"
      }
    ]  
  },
  ...
}
```

强烈建议`dataSource`明确提供inputSpec 中的段列表，以便您的delta摄取任务是幂等的。您可以通过以下呼叫协调员来获取该段列表。POST `/druid/coordinator/v1/metadata/datasources/{dataSourceName}/segments?full` 请求正文：[interval1，interval2，...]例如[“2012-01-01T00：00：00.000 / 2012-01-03T00：00：00.000”，“2012-01-05T00：00：00.000 / 2012 -01-07T00：00：00.000" ]

### 使用本机批量摄取重新索引

本节假设读者了解如何在没有Hadoop的情况下使用[本地批量索引](http://druid.io/docs/0.12.3/ingestion/native-batch.html)进行批量提取，后者使用“firehose”来了解读取输入数据的位置和方式。[IngestSegmentFirehose](http://druid.io/docs/0.12.3/ingestion/firehose.html#ingestsegmentfirehose) 可用于从德鲁伊内部的片段中读取数据。请注意，IndexTask仅用于原型设计，因为它必须在单个进程内进行所有处理，并且无法扩展。对于处理超过1GB数据的生产方案，请使用Hadoop批量提取。