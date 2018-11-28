# 教程：汇总

德鲁伊可以使用我们称之为“累积”的过程在摄取时间汇总原始数据。汇总是对选定列集的第一级聚合操作，可减少存储段的大小。

本教程将演示汇总对示例数据集的影响。

对于本教程，我们假设您已经按照[单机快速入门](http://druid.io/docs/0.12.3/tutorials/index.html)中的描述下载了Druid，并让它在本地计算机上运行。

完成[教程：加载文件](http://druid.io/docs/0.12.3/tutorials/tutorial-batch.html)和[教程：查询数据](http://druid.io/docs/0.12.3/tutorials/tutorial-query.html)也很有帮助。

## 示例数据

在本教程中，我们将使用一小部分网络流事件数据，表示从源到特定秒内发生的目标IP地址的流量的数据包和字节计数。

```json
{"timestamp":"2018-01-01T01:01:35Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2","packets":20,"bytes":9024}
{"timestamp":"2018-01-01T01:01:51Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2","packets":255,"bytes":21133}
{"timestamp":"2018-01-01T01:01:59Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2","packets":11,"bytes":5780}
{"timestamp":"2018-01-01T01:02:14Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2","packets":38,"bytes":6289}
{"timestamp":"2018-01-01T01:02:29Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2","packets":377,"bytes":359971}
{"timestamp":"2018-01-01T01:03:29Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2","packets":49,"bytes":10204}
{"timestamp":"2018-01-02T21:33:14Z","srcIP":"7.7.7.7", "dstIP":"8.8.8.8","packets":38,"bytes":6289}
{"timestamp":"2018-01-02T21:33:45Z","srcIP":"7.7.7.7", "dstIP":"8.8.8.8","packets":123,"bytes":93999}
{"timestamp":"2018-01-02T21:35:45Z","srcIP":"7.7.7.7", "dstIP":"8.8.8.8","packets":12,"bytes":2818}
```

包含此样本输入数据的文件位于`examples/rollup-data.json`。

我们将使用以下提取任务规范来获取此数据，该规范位于`examples/rollup-index.json`。

```json
{
  "type" : "index",
  "spec" : {
    "dataSchema" : {
      "dataSource" : "rollup-tutorial",
      "parser" : {
        "type" : "string",
        "parseSpec" : {
          "format" : "json",
          "dimensionsSpec" : {
            "dimensions" : [
              "srcIP",
              "dstIP"
            ]
          },
          "timestampSpec": {
            "column": "timestamp",
            "format": "iso"
          }
        }
      },
      "metricsSpec" : [
        { "type" : "count", "name" : "count" },
        { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
        { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" }
      ],
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "week",
        "queryGranularity" : "minute",
        "intervals" : ["2018-01-01/2018-01-03"],
        "rollup" : true
      }
    },
    "ioConfig" : {
      "type" : "index",
      "firehose" : {
        "type" : "local",
        "baseDir" : "examples",
        "filter" : "rollup-data.json"
      },
      "appendToExisting" : false
    },
    "tuningConfig" : {
      "type" : "index",
      "targetPartitionSize" : 5000000,
      "maxRowsInMemory" : 25000,
      "forceExtendableShardSpecs" : true
    }
  }
}
```

上卷已经通过设置启用`"rollup" : true`的`granularitySpec`。

请注意，我们`srcIP`并`dstIP`定义为维度，一个longSum度量的定义`packets`和`bytes`列，而`queryGranularity`被定义为`minute`。

我们将在加载此数据后看到如何使用这些定义。

## 加载示例数据

从druid-0.12.3软件包根目录运行以下命令：

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/rollup-index.json http://localhost:8090/druid/indexer/v1/task
```

加载数据后，我们将查询数据。

## 查询示例数据

让我们发出一个`select * from "rollup-tutorial";`查询来查看所摄取的数据。

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/rollup-select-sql.json http://localhost:8082/druid/v2/sql
```

将返回以下结果：

```json
[
  {
    "__time": "2018-01-01T01:01:00.000Z",
    "bytes": 35937,
    "count": 3,
    "dstIP": "2.2.2.2",
    "packets": 286,
    "srcIP": "1.1.1.1"
  },
  {
    "__time": "2018-01-01T01:02:00.000Z",
    "bytes": 366260,
    "count": 2,
    "dstIP": "2.2.2.2",
    "packets": 415,
    "srcIP": "1.1.1.1"
  },
  {
    "__time": "2018-01-01T01:03:00.000Z",
    "bytes": 10204,
    "count": 1,
    "dstIP": "2.2.2.2",
    "packets": 49,
    "srcIP": "1.1.1.1"
  },
  {
    "__time": "2018-01-02T21:33:00.000Z",
    "bytes": 100288,
    "count": 2,
    "dstIP": "8.8.8.8",
    "packets": 161,
    "srcIP": "7.7.7.7"
  },
  {
    "__time": "2018-01-02T21:35:00.000Z",
    "bytes": 2818,
    "count": 1,
    "dstIP": "8.8.8.8",
    "packets": 12,
    "srcIP": "7.7.7.7"
  }
]
```

让我们看一下在以下期间发生的原始输入数据中的三个事件`2018-01-01T01:01`：

```json
{"timestamp":"2018-01-01T01:01:35Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2","packets":20,"bytes":9024}
{"timestamp":"2018-01-01T01:01:51Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2","packets":255,"bytes":21133}
{"timestamp":"2018-01-01T01:01:59Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2","packets":11,"bytes":5780}
```

这三行已“卷起”到下一行：

```json
  {
    "__time": "2018-01-01T01:01:00.000Z",
    "bytes": 35937,
    "count": 3,
    "dstIP": "2.2.2.2",
    "packets": 286,
    "srcIP": "1.1.1.1"
  },
```

输入行已按时间戳和维度列进行分组，并`{timestamp, srcIP, dstIP}`在度量列`packets`和列上使用总和聚合`bytes`。

在分组发生之前，由于`"queryGranularity":"minute"`摄取规范中的设置，原始输入数据的时间戳会按分钟进行分区/分层。

同样，这两个发生在事件中的事件`2018-01-01T01:02`已经汇总：

```json
{"timestamp":"2018-01-01T01:02:14Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2","packets":38,"bytes":6289}
{"timestamp":"2018-01-01T01:02:29Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2","packets":377,"bytes":359971}
  {
    "__time": "2018-01-01T01:02:00.000Z",
    "bytes": 366260,
    "count": 2,
    "dstIP": "2.2.2.2",
    "packets": 415,
    "srcIP": "1.1.1.1"
  },
```

对于记录1.1.1.1和2.2.2.2之间流量的最后一个事件，没有发生汇总，因为这是在以下期间发生的唯一事件`2018-01-01T01:03`：

```json
{"timestamp":"2018-01-01T01:03:29Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2","packets":49,"bytes":10204}
  {
    "__time": "2018-01-01T01:03:00.000Z",
    "bytes": 10204,
    "count": 1,
    "dstIP": "2.2.2.2",
    "packets": 49,
    "srcIP": "1.1.1.1"
  },
```

请注意，`count`度量标准显示原始输入数据中有多少行对最终“累计”行有贡献。