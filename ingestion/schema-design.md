# 架构设计

此页面旨在帮助用户设计用于在Druid中摄取数据的模式。德鲁伊入射非规范化数据和列是三种类型之一：时间戳，维度或度量（或德鲁伊中已知的度量/聚合器）。这遵循 OLAP数据的[标准命名约定](https://en.wikipedia.org/wiki/Online_analytical_processing#Overview_of_OLAP_systems)。

有关更多详细信息：

- 德鲁伊的每一行都必须有一个时间戳。数据始终按时间划分，每个查询都有一个时间过滤器。查询结果也可以按时间段（如分钟，小时，天等）进行细分。
- 维度是可以过滤或分组的字段。它们总是单个字符串，字符串数组，单个长片，单个双打或单个浮点数。
- 度量标准是可以聚合的字段。它们通常存储为数字（整数或浮点数），但也可以存储为复杂对象，如HyperLogLog草图或近似直方图草图。

典型的生产表（或德鲁伊中已知的数据源）具有少于100个维度和少于100个度量，但是，基于用户证词，已创建具有数千个维度的数据源。

下面，我们概述了一些架构设计的最佳实践：

## 数字尺寸

如果用户希望将列作为数字类型的维度（Long，Double或Float）摄取，则需要在该`dimensions`部分中指定列的类型`dimensionsSpec`。如果省略该类型，Druid将摄取列作为默认String类型。

字符串和数字列之间存在性能折衷。数字列通常比字符串列更快分组。但与字符串列不同，数字列没有索引，因此它们通常较慢进行过滤。

有关更多信息，请参阅[Dimension Schema](http://druid.io/docs/0.12.3/ingestion/index.html#dimension-schema)。

## 高基数维度（例如唯一ID）

实际上，我们发现通常不需要对唯一ID的确切计数。将唯一ID存储为列会 [导致累积](http://druid.io/docs/0.12.3/ingestion/index.html#rollup)，并影响压缩。相反，存储所看到的唯一ID数量的草图，并将该草图用作聚合的一部分，将极大地提高性能（达到数量级的性能改进），并显着减少存储。Druid的`hyperUnique`聚合器基于Hyperloglog，可用于高基数维度的唯一计数。有关更多信息，请参阅[此处](https://www.youtube.com/watch?v=Hpd3f_MLdXo)。

## 嵌套尺寸

在撰写本文时，德鲁伊不支持嵌套维度。嵌套尺寸需要展平。例如，如果您有以下格式的数据：

```text
{"foo":{"bar": 3}}
```

然后在索引它之前，你应该将它转换为：

```text
{"foo_bar": 3}
```

Druid能够展平JSON输入数据，有关详细信息，请参阅[Flatten JSON](http://druid.io/docs/0.12.3/ingestion/flatten-json.html)。

## 计算摄取事件的数量

摄取时间的计数聚合器可用于计算摄取的事件数。但是，请务必注意，在查询此度量标准时，应使用`longSum`聚合器。`count`查询时的聚合器将返回时间间隔的德鲁伊行数，可用于确定汇总比率。

为了澄清一个例子，如果您的摄取规范包含：

```text
...
"metricsSpec" : [
      {
        "type" : "count",
        "name" : "count"
      },
...
```

您应该使用以下内容查询已摄取行的数量：

```text
...
"aggregations": [
    { "type": "longSum", "name": "numIngestedEvents", "fieldName": "count" },
...
```

## 无架构维度

如果`dimensions`字段在提取规范中保留为空，则德鲁伊将处理不是时间戳列的每列，已排除的维度或度量列作为维度。应该注意的是，由于[＃658，](https://github.com/druid-io/druid/issues/658) 这些段将比按字典顺序明确指定维度列表时略大。此限制不会影响查询的正确性 - 只是存储要求。

请注意，使用无模式提取时，所有维度都将作为字符串类型的维度提取。

## 包括与维度和指标相同的列

具有唯一ID的一个工作流程是能够过滤特定ID，同时仍能够对ID列执行快速唯一计数。如果您不使用无架构维度，则可以通过将`name`度量标准设置为与维度不同的值来支持此用例。如果您使用的是无架构维度，则此处的最佳做法是将同一列包含两次，一次作为维度，并作为`hyperUnique`指标。这可能涉及ETL时间的一些工作。

例如，对于无架构维度，请重复相同的列：

```text
{"device_id_dim":123, "device_id_met":123}
```

在你的`metricsSpec`，包括：

```text
{ "type" : "hyperUnique", "name" : "devices", "fieldName" : "device_id_met" }
```

`device_id_dim` 应自动获取作为维度。