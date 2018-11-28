# 教程：编写摄取规范

本教程将指导读者完成定义摄取规范的过程，指出关键考虑因素和指南。

对于本教程，我们假设您已经按照[单机快速入门](http://druid.io/docs/0.12.3/tutorials/index.html)中的描述下载了Druid，并让它在本地计算机上运行。

完成[教程：加载文件](http://druid.io/docs/0.12.3/tutorials/tutorial-batch.html)，[教程：查询数据](http://druid.io/docs/0.12.3/tutorials/tutorial-query.html)和[教程：汇总](http://druid.io/docs/0.12.3/tutorials/tutorial-rollup.html)也很有帮助。

## 示例数据

假设我们有以下网络流量数据：

- `srcIP`：发件人的IP地址
- `srcPort`：发件人端口
- `dstIP`：接收方的IP地址
- `dstPort`：接收器端口
- `protocol`：IP协议号
- `packets`：传输的数据包数
- `bytes`：传输的字节数
- `cost`：发送流量的成本

```json
{"ts":"2018-01-01T01:01:35Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":2000, "dstPort":3000, "protocol": 6, "packets":10, "bytes":1000, "cost": 1.4}
{"ts":"2018-01-01T01:01:51Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":2000, "dstPort":3000, "protocol": 6, "packets":20, "bytes":2000, "cost": 3.1}
{"ts":"2018-01-01T01:01:59Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":2000, "dstPort":3000, "protocol": 6, "packets":30, "bytes":3000, "cost": 0.4}
{"ts":"2018-01-01T01:02:14Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":5000, "dstPort":7000, "protocol": 6, "packets":40, "bytes":4000, "cost": 7.9}
{"ts":"2018-01-01T01:02:29Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":5000, "dstPort":7000, "protocol": 6, "packets":50, "bytes":5000, "cost": 10.2}
{"ts":"2018-01-01T01:03:29Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":5000, "dstPort":7000, "protocol": 6, "packets":60, "bytes":6000, "cost": 4.3}
{"ts":"2018-01-01T02:33:14Z","srcIP":"7.7.7.7", "dstIP":"8.8.8.8", "srcPort":4000, "dstPort":5000, "protocol": 17, "packets":100, "bytes":10000, "cost": 22.4}
{"ts":"2018-01-01T02:33:45Z","srcIP":"7.7.7.7", "dstIP":"8.8.8.8", "srcPort":4000, "dstPort":5000, "protocol": 17, "packets":200, "bytes":20000, "cost": 34.5}
{"ts":"2018-01-01T02:35:45Z","srcIP":"7.7.7.7", "dstIP":"8.8.8.8", "srcPort":4000, "dstPort":5000, "protocol": 17, "packets":300, "bytes":30000, "cost": 46.3}
```

将上面的JSON内容保存到名为`ingestion-tutorial-data.json`under 的文件中`examples`。

让我们逐步完成定义可以加载此数据的摄取规范的过程。

在本教程中，我们将使用本机批处理索引任务。使用其他任务类型时，摄取规范的某些方面会有所不同，本教程将指出这些区域。

## 定义架构

德鲁伊摄取规范的核心要素是`dataSchema`。的`dataSchema`定义如何解析数据输入到一组列，将被存储在德。

让我们从一个空的开始，`dataSchema`并在我们学习本教程的过程中添加字段。

使用以下内容创建一个名为`ingestion-tutorial-index.json`under 的新文件`examples`：

```json
"dataSchema" : {}
```

在我们学习本教程的过程中，我们将对此摄取规范进行连续编辑。

### 数据源名称

数据源名称由。中的`dataSource`参数指定`dataSchema`。

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
}
```

我们来调用教程数据源`ingestion-tutorial`。

### 选择一个解析器

A `dataSchema`有一个`parser`字段，它定义了德鲁伊将用于解释输入数据的解析器。

由于我们的输入数据表示为JSON字符串，我们将使用格式为的`string`解析器`json`：

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "parser" : {
    "type" : "string",
    "parseSpec" : {
      "format" : "json"
    }
  }
}
```

### 时间栏

在`parser`需要知道如何从输入数据中提取的主要时间戳字段。使用`json`类型时`parseSpec`，时间戳在a中定义`timestampSpec`。

输入数据中的timestamp列名为“ts”，包含ISO 8601时间戳，因此我们将以下`timestampSpec`信息添加到`parseSpec`：

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "parser" : {
    "type" : "string",
    "parseSpec" : {
      "format" : "json",
      "timestampSpec" : {
        "format" : "iso",
        "column" : "ts"
      }
    }
  }
}
```

### 列类型

现在我们已经定义了时间列，让我们看一下其他列的定义。

Druid支持以下列类型：String，Long，Float，Double。我们将在以下部分中看到它们的用法。

在我们继续讨论如何定义其他非时间列之前，我们`rollup`先讨论一下。

### 卷起

在摄取数据时，我们必须考虑是否要使用汇总。

- 如果启用了汇总，我们需要将输入列分为两个类别，“维度”和“指标”。“维度”是汇总的分组列，而“metrics”是要汇总的列。
- 如果禁用汇总，则所有列都将被视为“维度”，并且不会发生预聚合。

在本教程中，让我们启用汇总。这是用a指定`granularitySpec`的`dataSchema`。

注意`granularitySpec`谎言之外的谎言`parser`。`parser`当我们定义维度和指标时，我们将很快恢复。

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "parser" : {
    "type" : "string",
    "parseSpec" : {
      "format" : "json",
      "timestampSpec" : {
        "format" : "iso",
        "column" : "ts"
      }
    }
  },
  "granularitySpec" : {
    "rollup" : true
  }
}
```

#### 选择维度和指标

对于此示例数据集，以下是“维度”和“指标”的合理分割：

- 尺寸：srcIP，srcPort，dstIP，dstPort，协议
- 度量标准：数据包，字节，成本

此处的维度是一组标识IP流量单向流的属性，而度量标准表示有关维度分组指定的IP流量的事实。

我们来看看如何在摄取规范中定义这些维度和指标。

#### 外形尺寸

尺寸在`dimensionsSpec`内部指定`parseSpec`。

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "parser" : {
    "type" : "string",
    "parseSpec" : {
      "format" : "json",
      "timestampSpec" : {
        "format" : "iso",
        "column" : "ts"
      },
      "dimensionsSpec" : {
        "dimensions": [
          "srcIP",
          { "name" : "srcPort", "type" : "long" },
          { "name" : "dstIP", "type" : "string" },
          { "name" : "dstPort", "type" : "long" },
          { "name" : "protocol", "type" : "string" }
        ]
      }
    }
  },
  "granularitySpec" : {
    "rollup" : true
  }
}
```

每个维度都有a `name`和a `type`，其中`type`可以是“long”，“float”，“double”或“string”。

注意，这`srcIP`是一个“字符串”维度; 对于字符串维度，仅指定维名称就足够了，因为“string”是默认的维度类型。

另请注意，这`protocol`是输入数据中的数值，但我们将其作为“字符串”列进行摄取; 德鲁伊会在摄入过程中强制输入长串。

##### Strings vs. Numerics

数字输入应该作为数字维度还是字符串维度？

数字维度相对于字符串维度具有以下优点：*优点：数字表示可以导致磁盘上的列大小更小，并且从列中读取值时处理开销更低*缺点：数字维度没有索引，因此对它们进行过滤通常比过滤等效的String维度（具有位图索引）慢

#### 度量

度量标准在`metricsSpec`内部指定`dataSchema`：

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "parser" : {
    "type" : "string",
    "parseSpec" : {
      "format" : "json",
      "timestampSpec" : {
        "format" : "iso",
        "column" : "ts"
      },
      "dimensionsSpec" : {
        "dimensions": [
          "srcIP",
          { "name" : "srcPort", "type" : "long" },
          { "name" : "dstIP", "type" : "string" },
          { "name" : "dstPort", "type" : "long" },
          { "name" : "protocol", "type" : "string" }
        ]
      }   
    }
  },
  "metricsSpec" : [
    { "type" : "count", "name" : "count" },
    { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
    { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
    { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
  ],
  "granularitySpec" : {
    "rollup" : true
  }
}
```

在定义度量标准时，有必要指定在汇总期间应对该列执行的聚合类型。

在这里，我们已经在两个长指标列定义长和聚合，`packets`并`bytes`，并为双总和聚集`cost`列。

请注意，它`metricsSpec`位于与`dimensionSpec`or 不同的嵌套级别`parseSpec`; 它属于与`parser`内部相同的嵌套级别`dataSchema`。

请注意，我们还定义了一个`count`聚合器。计数聚合器将跟踪原始输入数据中有多少行对最终摄取数据中的“卷起”行的贡献。

### 没有汇总

如果我们没有使用汇总，则会在中指定所有列`dimensionsSpec`，例如：

```json
      "dimensionsSpec" : {
        "dimensions": [
          "srcIP",
          { "name" : "srcPort", "type" : "long" },
          { "name" : "dstIP", "type" : "string" },
          { "name" : "dstPort", "type" : "long" },
          { "name" : "protocol", "type" : "string" },
          { "name" : "packets", "type" : "long" },
          { "name" : "bytes", "type" : "long" },
          { "name" : "srcPort", "type" : "double" }
        ]
      },
```

### 定义粒度

在这一点上，我们已经完成了定义`parser`和`metricsSpec`内部`dataSchema`，我们几乎完成了编写摄取规范。

我们需要在以下内容中设置一些其他属性`granularitySpec`：*粒度类型规范：`uniform`并且`arbitrary`是两种受支持的类型。对于本教程，我们将使用`uniform`粒度规范，其中所有段具有统一的间隔大小（例如，所有段都覆盖一小时的数据）。*段粒度：单个段包含数据的时间间隔大小是多少？例如`DAY`，`WEEK` *在时间列的时间戳的铲装粒度（简称`queryGranularity`）

#### 细分粒度

段粒度由`segmentGranularity`属性中的配置`granularitySpec`。在本教程中，我们将创建每小时细分：

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "parser" : {
    "type" : "string",
    "parseSpec" : {
      "format" : "json",
      "timestampSpec" : {
        "format" : "iso",
        "column" : "ts"
      },
      "dimensionsSpec" : {
        "dimensions": [
          "srcIP",
          { "name" : "srcPort", "type" : "long" },
          { "name" : "dstIP", "type" : "string" },
          { "name" : "dstPort", "type" : "long" },
          { "name" : "protocol", "type" : "string" }
        ]
      }      
    }
  },
  "metricsSpec" : [
    { "type" : "count", "name" : "count" },
    { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
    { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
    { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
  ],
  "granularitySpec" : {
    "type" : "uniform",
    "segmentGranularity" : "HOUR",
    "rollup" : true
  }
}
```

我们的输入数据包含两个不同小时的事件，因此此任务将生成两个段。

#### 查询粒度

查询粒度由`queryGranularity`属性中的配置`granularitySpec`。对于本教程，我们使用分钟粒度：

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "parser" : {
    "type" : "string",
    "parseSpec" : {
      "format" : "json",
      "timestampSpec" : {
        "format" : "iso",
        "column" : "ts"
      },
      "dimensionsSpec" : {
        "dimensions": [
          "srcIP",
          { "name" : "srcPort", "type" : "long" },
          { "name" : "dstIP", "type" : "string" },
          { "name" : "dstPort", "type" : "long" },
          { "name" : "protocol", "type" : "string" }
        ]
      }      
    }
  },
  "metricsSpec" : [
    { "type" : "count", "name" : "count" },
    { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
    { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
    { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
  ],
  "granularitySpec" : {
    "type" : "uniform",
    "segmentGranularity" : "HOUR",
    "queryGranularity" : "MINUTE"
    "rollup" : true
  }
}
```

要查看查询粒度的效果，让我们从原始输入数据中查看此行：

```json
{"ts":"2018-01-01T01:03:29Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":5000, "dstPort":7000, "protocol": 6, "packets":60, "bytes":6000, "cost": 4.3}
```

当这行被微小的queryGranularity摄取时，德鲁伊会将行的时间戳放到分钟桶中：

```json
{"ts":"2018-01-01T01:03:00Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":5000, "dstPort":7000, "protocol": 6, "packets":60, "bytes":6000, "cost": 4.3}
```

#### 定义间隔（仅限批处理）

对于批处理任务，必须定义时间间隔。时间间隔超出时间间隔的输入行将不会被摄取。

间隔也在以下内容中指定`granularitySpec`：

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "parser" : {
    "type" : "string",
    "parseSpec" : {
      "format" : "json",
      "timestampSpec" : {
        "format" : "iso",
        "column" : "ts"
      },
      "dimensionsSpec" : {
        "dimensions": [
          "srcIP",
          { "name" : "srcPort", "type" : "long" },
          { "name" : "dstIP", "type" : "string" },
          { "name" : "dstPort", "type" : "long" },
          { "name" : "protocol", "type" : "string" }
        ]
      }      
    }
  },
  "metricsSpec" : [
    { "type" : "count", "name" : "count" },
    { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
    { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
    { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
  ],
  "granularitySpec" : {
    "type" : "uniform",
    "segmentGranularity" : "HOUR",
    "queryGranularity" : "MINUTE",
    "intervals" : ["2018-01-01/2018-01-02"],
    "rollup" : true
  }
}
```

## 定义任务类型

我们现在已经完成了我们的定义`dataSchema`。剩下的步骤是将`dataSchema`我们创建的数据放入摄取任务规范中，并指定输入源。

它`dataSchema`在所有任务类型中共享，但每种任务类型都有自己的规范格式。在本教程中，我们将使用本机批量提取任务：

```json
{
  "type" : "index",
  "spec" : {
    "dataSchema" : {
      "dataSource" : "ingestion-tutorial",
      "parser" : {
        "type" : "string",
        "parseSpec" : {
          "format" : "json",
          "timestampSpec" : {
            "format" : "iso",
            "column" : "ts"
          },
          "dimensionsSpec" : {
            "dimensions": [
              "srcIP",
              { "name" : "srcPort", "type" : "long" },
              { "name" : "dstIP", "type" : "string" },
              { "name" : "dstPort", "type" : "long" },
              { "name" : "protocol", "type" : "string" }
            ]              
          }      
        }
      },
      "metricsSpec" : [
        { "type" : "count", "name" : "count" },
        { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
        { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
        { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
      ],
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "HOUR",
        "queryGranularity" : "MINUTE",
        "intervals" : ["2018-01-01/2018-01-02"],
        "rollup" : true
      }
    }
  }
}
```

## 定义输入源

现在让我们定义一个在`ioConfig`对象中指定的输入源。每种任务类型都有自己的类型`ioConfig`。本机批处理任务使用“firehoses”来读取输入数据，因此让我们配置一个“本地”firehose来读取我们之前保存的示例netflow数据：

```json
    "ioConfig" : {
      "type" : "index",
      "firehose" : {
        "type" : "local",
        "baseDir" : "examples/",
        "filter" : "ingestion-tutorial-data.json"
      }
    }
{
  "type" : "index",
  "spec" : {
    "dataSchema" : {
      "dataSource" : "ingestion-tutorial",
      "parser" : {
        "type" : "string",
        "parseSpec" : {
          "format" : "json",
          "timestampSpec" : {
            "format" : "iso",
            "column" : "ts"
          },
          "dimensionsSpec" : {
            "dimensions": [
              "srcIP",
              { "name" : "srcPort", "type" : "long" },
              { "name" : "dstIP", "type" : "string" },
              { "name" : "dstPort", "type" : "long" },
              { "name" : "protocol", "type" : "string" }
            ]
          }      
        }
      },
      "metricsSpec" : [
        { "type" : "count", "name" : "count" },
        { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
        { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
        { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
      ],
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "HOUR",
        "queryGranularity" : "MINUTE",
        "intervals" : ["2018-01-01/2018-01-02"],
        "rollup" : true
      }
    },
    "ioConfig" : {
      "type" : "index",
      "firehose" : {
        "type" : "local",
        "baseDir" : "examples/",
        "filter" : "ingestion-tutorial-data.json"
      }
    }
  }
}
```

## 额外的调整

每个摄取任务都有一个`tuningConfig`部分，允许用户调整各种摄取参数。

例如，让我们添加一个`tuningConfig`为本机批量提取任务设置目标段大小：

```json
    "tuningConfig" : {
      "type" : "index",
      "targetPartitionSize" : 5000000
    }
```

请注意，每个摄取任务都有自己的类型`tuningConfig`。

## 最终规范

我们已经完成了摄取规范的定义，它现在应该如下所示：

```json
{
  "type" : "index",
  "spec" : {
    "dataSchema" : {
      "dataSource" : "ingestion-tutorial",
      "parser" : {
        "type" : "string",
        "parseSpec" : {
          "format" : "json",
          "timestampSpec" : {
            "format" : "iso",
            "column" : "ts"
          },
          "dimensionsSpec" : {
            "dimensions": [
              "srcIP",
              { "name" : "srcPort", "type" : "long" },
              { "name" : "dstIP", "type" : "string" },
              { "name" : "dstPort", "type" : "long" },
              { "name" : "protocol", "type" : "string" }
            ]
          }      
        }
      },
      "metricsSpec" : [
        { "type" : "count", "name" : "count" },
        { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
        { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
        { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
      ],
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "HOUR",
        "queryGranularity" : "MINUTE",
        "intervals" : ["2018-01-01/2018-01-02"],
        "rollup" : true
      }
    },
    "ioConfig" : {
      "type" : "index",
      "firehose" : {
        "type" : "local",
        "baseDir" : "examples/",
        "filter" : "ingestion-tutorial-data.json"
      }
    },
    "tuningConfig" : {
      "type" : "index",
      "targetPartitionSize" : 5000000
    }
  }
}
```

## 提交任务并查询数据

从druid-0.12.3软件包根目录运行以下命令：

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/ingestion-tutorial-index.json http://localhost:8090/druid/indexer/v1/task
```

脚本完成后，我们将查询数据。

让我们发出一个`select * from "ingestion-tutorial";`查询来查看所摄取的数据。

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/ingestion-tutorial-select-sql.json http://localhost:8082/druid/v2/sql
[
  {
    "__time": "2018-01-01T01:01:00.000Z",
    "bytes": 6000,
    "cost": 4.9,
    "count": 3,
    "dstIP": "2.2.2.2",
    "dstPort": 3000,
    "packets": 60,
    "protocol": "6",
    "srcIP": "1.1.1.1",
    "srcPort": 2000
  },
  {
    "__time": "2018-01-01T01:02:00.000Z",
    "bytes": 9000,
    "cost": 18.1,
    "count": 2,
    "dstIP": "2.2.2.2",
    "dstPort": 7000,
    "packets": 90,
    "protocol": "6",
    "srcIP": "1.1.1.1",
    "srcPort": 5000
  },
  {
    "__time": "2018-01-01T01:03:00.000Z",
    "bytes": 6000,
    "cost": 4.3,
    "count": 1,
    "dstIP": "2.2.2.2",
    "dstPort": 7000,
    "packets": 60,
    "protocol": "6",
    "srcIP": "1.1.1.1",
    "srcPort": 5000
  },
  {
    "__time": "2018-01-01T02:33:00.000Z",
    "bytes": 30000,
    "cost": 56.9,
    "count": 2,
    "dstIP": "8.8.8.8",
    "dstPort": 5000,
    "packets": 300,
    "protocol": "17",
    "srcIP": "7.7.7.7",
    "srcPort": 4000
  },
  {
    "__time": "2018-01-01T02:35:00.000Z",
    "bytes": 30000,
    "cost": 46.3,
    "count": 1,
    "dstIP": "8.8.8.8",
    "dstPort": 5000,
    "packets": 300,
    "protocol": "17",
    "srcIP": "7.7.7.7",
    "srcPort": 4000
  }
]
```