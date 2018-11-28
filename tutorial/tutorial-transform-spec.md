# 教程：转换输入数据

本教程将演示如何使用转换规范在摄取期间过滤和转换输入数据。

对于本教程，我们假设您已经按照[单机快速入门](http://druid.io/docs/0.12.3/tutorials/index.html)中的描述下载了Druid，并让它在本地计算机上运行。

完成[教程：加载文件](http://druid.io/docs/0.12.3/tutorials/tutorial-batch.html)和[教程：查询数据](http://druid.io/docs/0.12.3/tutorials/tutorial-query.html)也很有帮助。

## 样本数据

我们已经包含了本教程的示例数据`examples/transform-data.json`，为方便起见，此处转载：

```json
{"timestamp":"2018-01-01T07:01:35Z","animal":"octopus",  "location":1, "number":100}
{"timestamp":"2018-01-01T05:01:35Z","animal":"mongoose", "location":2,"number":200}
{"timestamp":"2018-01-01T06:01:35Z","animal":"snake", "location":3, "number":300}
{"timestamp":"2018-01-01T01:01:35Z","animal":"lion", "location":4, "number":300}
```

## 使用转换规范加载数据

我们将使用以下规范来摄取样本数据，该规范演示了变换规范的使用：

```json
{
  "type" : "index",
  "spec" : {
    "dataSchema" : {
      "dataSource" : "transform-tutorial",
      "parser" : {
        "type" : "string",
        "parseSpec" : {
          "format" : "json",
          "dimensionsSpec" : {
            "dimensions" : [
              "animal",
              { "name": "location", "type": "long" }
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
        { "type" : "longSum", "name" : "number", "fieldName" : "number" },
        { "type" : "longSum", "name" : "triple-number", "fieldName" : "triple-number" }
      ],
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "week",
        "queryGranularity" : "minute",
        "intervals" : ["2018-01-01/2018-01-03"],
        "rollup" : true
      },
      "transformSpec": {
        "transforms": [
          {
            "type": "expression",
            "name": "animal",
            "expression": "concat('super-', animal)"
          },
          {
            "type": "expression",
            "name": "triple-number",
            "expression": "number * 3"
          }
        ],
        "filter": {
          "type":"or",
          "fields": [
            { "type": "selector", "dimension": "animal", "value": "super-mongoose" },
            { "type": "selector", "dimension": "triple-number", "value": "300" },
            { "type": "selector", "dimension": "location", "value": "3" }
          ]
        }
      }
    },
    "ioConfig" : {
      "type" : "index",
      "firehose" : {
        "type" : "local",
        "baseDir" : "examples/",
        "filter" : "transform-data.json"
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

在变换规范中，我们有两个表达式变换：* `super-animal`：将“super-”预先添加到`animal`列中的值。这将覆盖`animal`具有转换版本的列，因为转换的名称是`animal`。* `triple-number`：将`number`列乘以3.这将创建一个新`triple-number`列。请注意，我们正在摄取原始列和转换列。

另外，我们有一个带有三个子句的OR过滤器：* `super-animal`匹配“super-mongoose”的`triple-number`值*匹配300 * `location`值的3个值

此过滤器选择前3行，它将排除输入数据中的最后一个“lion”行。请注意，过滤器在转换后应用。

我们现在提交此任务，其中包括`examples/transform-index.json`：

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/transform-index.json http://localhost:8090/druid/indexer/v1/task
```

## 查询转换后的数据

让我们来`select * from "transform-tutorial";`查询一下摄取的内容：

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/transform-select-sql.json http://localhost:8082/druid/v2/sql
[
  {
    "__time": "2018-01-01T05:01:00.000Z",
    "animal": "super-mongoose",
    "count": 1,
    "location": 2,
    "number": 200,
    "triple-number": 600
  },
  {
    "__time": "2018-01-01T06:01:00.000Z",
    "animal": "super-snake",
    "count": 1,
    "location": 3,
    "number": 300,
    "triple-number": 900
  },
  {
    "__time": "2018-01-01T07:01:00.000Z",
    "animal": "super-octopus",
    "count": 1,
    "location": 1,
    "number": 100,
    "triple-number": 300
  }
]
```

“狮子”行已被丢弃，`animal`列已被转换，我们同时拥有原始`number`列和转换列。