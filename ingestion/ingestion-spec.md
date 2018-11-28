# 摄取规格

德鲁伊摄取规范由3个组成部分组成：

```json
{
  "dataSchema" : {...},
  "ioConfig" : {...},
  "tuningConfig" : {...}
}
```

| 领域         | 类型     | 描述                                                         | 需要 |
| ------------ | -------- | ------------------------------------------------------------ | ---- |
| dataSchema   | JSON对象 | 指定传入数据的架构。所有提取规范都可以共享相同的dataSchema。 | 是   |
| IOconfig进行 | JSON对象 | 指定数据的来源以及数据的去向。此对象将随摄取方法而变化。     | 是   |
| tuningConfig | JSON对象 | 指定如何调整各种摄取参数。此对象将随摄取方法而变化。         | 没有 |

# DataSchema

示例dataSchema如下所示：

```json
"dataSchema" : {
  "dataSource" : "wikipedia",
  "parser" : {
    "type" : "string",
    "parseSpec" : {
      "format" : "json",
      "timestampSpec" : {
        "column" : "timestamp",
        "format" : "auto"
      },
      "dimensionsSpec" : {
        "dimensions": [
          "page",
          "language",
          "user",
          "unpatrolled",
          "newPage",
          "robot",
          "anonymous",
          "namespace",
          "continent",
          "country",
          "region",
          "city",
          {
            "type": "long",
            "name": "countryNum"
          },
          {
            "type": "float",
            "name": "userLatitude"
          },
          {
            "type": "float",
            "name": "userLongitude"
          }
        ],
        "dimensionExclusions" : [],
        "spatialDimensions" : []
      }
    }
  },
  "metricsSpec" : [{
    "type" : "count",
    "name" : "count"
  }, {
    "type" : "doubleSum",
    "name" : "added",
    "fieldName" : "added"
  }, {
    "type" : "doubleSum",
    "name" : "deleted",
    "fieldName" : "deleted"
  }, {
    "type" : "doubleSum",
    "name" : "delta",
    "fieldName" : "delta"
  }],
  "granularitySpec" : {
    "segmentGranularity" : "DAY",
    "queryGranularity" : "NONE",
    "intervals" : [ "2013-08-31/2013-09-01" ]
  },
  "transformSpec" : null
}
```

| 领域            | 类型         | 描述                                                         | 需要 |
| --------------- | ------------ | ------------------------------------------------------------ | ---- |
| 数据源          | 串           | 摄取数据源的名称。数据源可以被视为表。                       | 是   |
| 解析器          | JSON对象     | 指定如何解析所摄取的数据。                                   | 是   |
| metricsSpec     | JSON对象数组 | [聚合器](http://druid.io/docs/0.12.3/querying/aggregations.html)列表。 | 是   |
| granularitySpec | JSON对象     | 指定如何创建段和汇总数据。                                   | 是   |
| transformSpec   | JSON对象     | 指定如何过滤和转换输入数据。请参阅[转换规范](http://druid.io/docs/0.12.3/ingestion/transform-spec.html)。 | 没有 |

## 分析器

如果`type`未包含，则解析器默认为`string`。有关其他数据格式，请参阅我们的[扩展列表](http://druid.io/docs/0.12.3/development/extensions.html)。

### String Parser

| 领域      | 类型     | 描述                                                         | 需要 |
| --------- | -------- | ------------------------------------------------------------ | ---- |
| 类型      | 串       | 这应该说`string`是一般的，或者`hadoopyString`在Hadoop索引作业中使用时。 | 没有 |
| parseSpec | JSON对象 | 指定数据的格式，时间戳和维度。                               | 是   |

### ParseSpec

ParseSpecs有两个目的：

- String Parser使用它们来确定传入行的格式（即JSON，CSV，TSV）。
- 所有解析器都使用它们来确定传入行的时间戳和维度。

如果`format`未包含，则parseSpec默认为`tsv`。

#### JSON ParseSpec

与String Parser一起使用它来加载JSON。

| 领域           | 类型     | 描述                                                         | 需要 |
| -------------- | -------- | ------------------------------------------------------------ | ---- |
| 格式           | 串       | 这应该说`json`。                                             | 没有 |
| timestampSpec  | JSON对象 | 指定时间戳的列和格式。                                       | 是   |
| dimensionsSpec | JSON对象 | 指定数据的维度。                                             | 是   |
| flattenSpec    | JSON对象 | 指定嵌套JSON数据的展平配置。有关详细信息，请参阅[Flattening JSON](http://druid.io/docs/0.12.3/ingestion/flatten-json.html)。 | 没有 |

#### JSON小写ParseSpec

_jsonLowercase_解析器已弃用，可能会在将来的Druid版本中删除。

这是JSON ParseSpec的一种特殊变体，它可以降低传入JSON数据中所有列名称的大小。如果您从Druid 0.6.x更新到Druid 0.7.x，直接使用混合大小写的列名称来获取JSON，没有任何ETL来小写这些列名称，并且想要进行查询，则需要此parseSpec包括使用0.6.x和0.7.x创建的数据。

| 领域           | 类型     | 描述                      | 需要 |
| -------------- | -------- | ------------------------- | ---- |
| 格式           | 串       | 这应该说`jsonLowercase`。 | 是   |
| timestampSpec  | JSON对象 | 指定时间戳的列和格式。    | 是   |
| dimensionsSpec | JSON对象 | 指定数据的维度。          | 是   |

#### CSV ParseSpec

与String Parser一起使用它来加载CSV。使用com.opencsv库解析字符串。

| 领域           | 类型     | 描述                     | 需要                  |
| -------------- | -------- | ------------------------ | --------------------- |
| 格式           | 串       | 这应该说`csv`。          | 是                    |
| timestampSpec  | JSON对象 | 指定时间戳的列和格式。   | 是                    |
| dimensionsSpec | JSON对象 | 指定数据的维度。         | 是                    |
| listDelimiter  | 串       | 多值维度的自定义分隔符。 | 不（默认== ctrl + A） |
| 列             | JSON数组 | 指定数据的列。           | 是                    |

#### TSV / Delimited ParseSpec

与String Parser一起使用它可以加载任何不需要特殊转义的分隔文本。默认情况下，分隔符是一个选项卡，因此这将加载TSV。

| 领域           | 类型           | 描述                     | 需要                  |
| -------------- | -------------- | ------------------------ | --------------------- |
| 格式           | 串             | 这应该说`tsv`。          | 是                    |
| timestampSpec  | JSON对象       | 指定时间戳的列和格式。   | 是                    |
| dimensionsSpec | JSON对象       | 指定数据的维度。         | 是                    |
| 分隔符         | 串             | 数据值的自定义分隔符。   | 不（默认== \ t）      |
| listDelimiter  | 串             | 多值维度的自定义分隔符。 | 不（默认== ctrl + A） |
| 列             | JSON字符串数组 | 指定数据的列。           | 是                    |

#### TimeAndDims ParseSpec

与非String Parsers一起使用它可以为它们提供时间戳和维度信息。Non-String Parsers自己处理所有格式化决策，而不使用ParseSpec。

| 领域           | 类型     | 描述                    | 需要 |
| -------------- | -------- | ----------------------- | ---- |
| 格式           | 串       | 这应该说`timeAndDims`。 | 是   |
| timestampSpec  | JSON对象 | 指定时间戳的列和格式。  | 是   |
| dimensionsSpec | JSON对象 | 指定数据的维度。        | 是   |

### TimestampSpec

| 领域 | 类型 | 描述                                                         | 需要             |
| ---- | ---- | ------------------------------------------------------------ | ---------------- |
| 柱   | 串   | 时间戳的列。                                                 | 是               |
| 格式 | 串   | iso，millis，posix，auto或任何[Joda时间](http://joda-time.sourceforge.net/apidocs/org/joda/time/format/DateTimeFormat.html)格式。 | 不（默认=='自动' |

### DimensionsSpec

| 领域                | 类型           | 描述                                                         | 需要          |
| ------------------- | -------------- | ------------------------------------------------------------ | ------------- |
| 尺寸                | JSON数组       | [维度架构](http://druid.io/docs/0.12.3/ingestion/ingestion-spec.html#dimension-schema)对象或维度名称的列表。提供名称相当于提供具有给定名称的String类型维度模式。如果这是一个空数组，Druid会将非时间戳或公制列的所有列视为字符串类型的维列。 | 是            |
| dimensionExclusions | JSON字符串数组 | 要从摄取中排除的维度的名称。                                 | 不（默认== [] |
| spatialDimensions   | JSON对象数组   | 一系列[空间维度](http://druid.io/docs/0.12.3/development/geo.html) | 不（默认== [] |

#### 维度架构

维度模式指定要摄取的维度的类型和名称。

对于字符串列，维度模式还可用于通过设置`createBitmapIndex`布尔值来启用或禁用位图索引 。默认情况下，为所有字符串列启用位图索引。只有字符串列才能有位图索引; 数字列不支持它们。

例如，以下`dimensionsSpec`部分将`dataSchema`一列作为Long（`countryNum`），两列作为Float（`userLatitude`，`userLongitude`），其他列作为字符串，并为`comment`列禁用位图索引。

```json
"dimensionsSpec" : {
  "dimensions": [
    "page",
    "language",
    "user",
    "unpatrolled",
    "newPage",
    "robot",
    "anonymous",
    "namespace",
    "continent",
    "country",
    "region",
    "city",
    {
      "type": "string",
      "name": "comment",
      "createBitmapIndex": false
    },
    {
      "type": "long",
      "name": "countryNum"
    },
    {
      "type": "float",
      "name": "userLatitude"
    },
    {
      "type": "float",
      "name": "userLongitude"
    }
  ],
  "dimensionExclusions" : [],
  "spatialDimensions" : []
}
```

## metricsSpec

这`metricsSpec`是一个[聚合器](http://druid.io/docs/0.12.3/querying/aggregations.html)列表。如果`rollup`粒度规范中的false为false，则metrics规范应该是一个空列表，并且应该在相应的位置定义所有列`dimensionsSpec`（没有汇总，在摄取时间内维度和度量之间没有真正的区别）。然而，这是可选的。

## GranularitySpec

默认粒度规范是`uniform`，可以通过设置`type`字段来更改。目前，`uniform`和`arbitrary`类型的支持。

### 均匀粒度规格

此规范用于生成具有均匀间隔的段。

| 领域               | 类型 | 描述                                                         | 需要                                                         |
| ------------------ | ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| segmentGranularity | 串   | 创建细分的粒度。                                             | 不（默认=='DAY'）                                            |
| queryGranularity   | 串   | 能够查询结果的最小粒度以及段内数据的粒度。例如，“分钟”的值意味着数据以微小的粒度聚合。也就是说，如果元组中存在冲突（分钟（时间戳），维度），那么它将使用聚合器将值聚合在一起，而不是存储单个行。粒度为“无”意味着毫秒粒度。 | 不（默认=='无'）                                             |
| 卷起               | 布尔 | 汇总与否                                                     | 不（默认== true）                                            |
| 间隔               | 串   | 正在摄取的原始数据的间隔列表。忽略实时摄取。                 | 没有。如果指定，批量摄取任务可能会跳过确定分区阶段，从而导致更快的摄取。 |

### 任意粒度规格

此规范用于生成具有任意间隔的段（它尝试创建大小均匀的段）。实时处理不支持此规范。

| 领域             | 类型 | 描述                                                         | 需要                                                         |
| ---------------- | ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| queryGranularity | 串   | 能够查询结果的最小粒度以及段内数据的粒度。例如，“分钟”的值意味着数据以微小的粒度聚合。也就是说，如果元组中存在冲突（分钟（时间戳），维度），那么它将使用聚合器将值聚合在一起，而不是存储单个行。粒度为“无”意味着毫秒粒度。 | 不（默认=='无'）                                             |
| 卷起             | 布尔 | 汇总与否                                                     | 不（默认== true）                                            |
| 间隔             | 串   | 正在摄取的原始数据的间隔列表。忽略实时摄取。                 | 没有。如果指定，批量摄取任务可能会跳过确定分区阶段，从而导致更快的摄取。 |

# 变换规格

变换规范允许Druid在摄取期间转换和过滤输入数据。请参见[转换规格](http://druid.io/docs/0.12.3/ingestion/transform-spec.html)

# IO配置

IOConfig规范根据摄取任务类型而有所不同。

- 本机批量摄取：请参阅本[机批量IOConfig](http://druid.io/docs/0.12.3/ingestion/native-batch.html#ioconfig)
- [Hadoop Batch ingestion](http://druid.io/docs/0.12.3/ingestion/hadoop.html#ioconfig)：请参阅[Hadoop Batch IOConfig](http://druid.io/docs/0.12.3/ingestion/hadoop.html#ioconfig)
- Kafka Indexing Service：见[Kafka Supervisor IOConfig](http://druid.io/docs/0.12.3/development/extensions-core/kafka-ingestion.html#KafkaSupervisorIOConfig)
- 流推入摄取：使用Tranquility进行流推入不需要IO配置。
- Stream Pull Ingestion（不推荐使用）：请参阅[Stream pull ingestion](http://druid.io/docs/0.12.3/ingestion/stream-pull.html#ioconfig)。

# 调整配置

TuningConfig规范根据摄取任务类型而有所不同。

- 本机批量摄取：请参阅[Native Batch TuningConfig](http://druid.io/docs/0.12.3/ingestion/native-batch.html#tuningconfig)
- [Hadoop Batch ingestion](http://druid.io/docs/0.12.3/ingestion/hadoop.html#tuningconfig)：请参阅[Hadoop Batch TuningConfig](http://druid.io/docs/0.12.3/ingestion/hadoop.html#tuningconfig)
- Kafka Indexing Service：请参阅[Kafka Supervisor TuningConfig](http://druid.io/docs/0.12.3/development/extensions-core/kafka-ingestion.html#KafkaSupervisorTuningConfig)
- Stream Push [Ingestion](http://static.druid.io/tranquility/api/latest/#com.metamx.tranquility.druid.DruidTuning)（Tranquility）：参见[Tranquility TuningConfig](http://static.druid.io/tranquility/api/latest/#com.metamx.tranquility.druid.DruidTuning)。
- Stream Pull Ingestion（不推荐使用）：请参阅[Stream pull ingestion](http://druid.io/docs/0.12.3/ingestion/stream-pull.html#tuningconfig)。

# 评估时间戳，维度和指标

德鲁伊将按以下顺序解释维度，维度排除和指标：

- 维度列表中列出的任何列都被视为维度。
- 维度排除列表中列出的任何列都将作为维度排除。
- 默认情况下会排除度量标准所需的时间戳列和列/字段名称。
- 如果度量标准也列为维度，则度量标准必须具有与维度名称不同的名称。