# 教程：加载文件

## 入门

本教程演示了如何使用Druid的本机批处理提取来执行批处理文件加载。

对于本教程，我们假设您已经按照[单机快速入门](http://druid.io/docs/0.12.3/tutorials/index.html)中的描述下载了Druid，并让它在本地计算机上运行。您还不需要加载任何数据。

## 准备数据和摄取任务规范

通过向德鲁伊霸主提交*摄取任务*规范来启动数据加载。在本教程中，我们将加载示例Wikipedia页面编辑数据。

我们提供了一个摄取规范`examples/wikipedia-index.json`，为方便起见，这里已经配置为读取`quickstart/wikiticker-2015-09-12-sampled.json.gz`输入文件：

```json
{
  "type" : "index",
  "spec" : {
    "dataSchema" : {
      "dataSource" : "wikipedia",
      "parser" : {
        "type" : "string",
        "parseSpec" : {
          "format" : "json",
          "dimensionsSpec" : {
            "dimensions" : [
              "channel",
              "cityName",
              "comment",
              "countryIsoCode",
              "countryName",
              "isAnonymous",
              "isMinor",
              "isNew",
              "isRobot",
              "isUnpatrolled",
              "metroCode",
              "namespace",
              "page",
              "regionIsoCode",
              "regionName",
              "user",
              { "name": "added", "type": "long" },
              { "name": "deleted", "type": "long" },
              { "name": "delta", "type": "long" }
            ]
          },
          "timestampSpec": {
            "column": "time",
            "format": "iso"
          }
        }
      },
      "metricsSpec" : [],
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "day",
        "queryGranularity" : "none",
        "intervals" : ["2015-09-12/2015-09-13"],
        "rollup" : false
      }
    },
    "ioConfig" : {
      "type" : "index",
      "firehose" : {
        "type" : "local",
        "baseDir" : "quickstart/",
        "filter" : "wikiticker-2015-09-12-sampled.json.gz"
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

此规范将创建名为“wikipedia”的数据源。

## 加载批次数据

我们从2015年9月12日开始包含维基百科编辑样本，以帮助您入门。

要将此数据加载到Druid中，您可以提交指向该文件的*摄取任务*。要提交此任务，请在druid-0.12.3目录的新终端窗口中将其发布到Druid：

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/wikipedia-index.json http://localhost:8090/druid/indexer/v1/task
```

如果提交成功，将打印任务的ID：

```bash
{"task":"index_wikipedia_2018-06-09T21:30:32.802Z"}
```

要查看摄取任务的状态，请转到您的霸主控制台： [http：// localhost：8090 / console.html](http://localhost:8090/console.html)。您可以定期刷新控制台，任务成功后，您应该看到任务的“成功”状态。

摄取任务完成后，数据将由历史节点加载，并可在一两分钟内进行查询。您可以通过检查是否存在带有蓝色圆圈的数据源“wikipedia”来监控协调器控制台中加载数据的进度：[http：// localhost：8081 /＃/](http://localhost:8081/#/)。

![协调员控制台](http://druid.io/docs/0.12.3/tutorials/img/tutorial-batch-01.png)

## 查询您的数据

您的数据应在一两分钟内完全可用。您可以在协调器控制台上以[http：// localhost：8081 /＃/](http://localhost:8081/#/)监视此过程。

加载数据后，请按照[查询教程](http://druid.io/docs/0.12.3/tutorials/tutorial-query.html)对新加载的数据运行一些示例查询。

## 清理

如果您希望浏览任何其他摄取教程，则需要重置群集并按照这些[重置说明进行操作](http://druid.io/docs/0.12.3/tutorials/index.html#resetting-cluster-state)，因为其他教程将写入相同的“维基百科”数据源。

## 进一步阅读

有关加载批处理数据的更多信息，请参阅[批处理提取文档](http://druid.io/docs/0.12.3/ingestion/batch-ingestion.html)。