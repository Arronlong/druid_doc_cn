# 数据源元数据查询

数据源元数据查询返回dataSource的元数据信息。这些查询返回以下信息：

- dataSource的最新摄取事件的时间戳。这是摄取事件而不考虑汇总。

这些查询的语法是：

```json
{
    "queryType" : "dataSourceMetadata",
    "dataSource": "sample_datasource"
}
```

数据源元数据查询有两个主要部分：

| 属性     | 描述                                                         | 需要？ |
| -------- | ------------------------------------------------------------ | ------ |
| 查询类型 | 该String应始终为“dataSourceMetadata”; 这是德鲁伊首先想弄清楚如何解释查询 | 是     |
| 数据源   | 定义要查询的数据源的String或Object，非常类似于关系数据库中的表。有关更多信息，请参阅[DataSource](http://druid.io/docs/0.12.3/querying/datasource.html)。 | 是     |
| 上下文   | 见[上下文](http://druid.io/docs/0.12.3/querying/query-context.html) | 没有   |

结果的格式是：

```json
[ {
  "timestamp" : "2013-05-09T18:24:00.000Z",
  "result" : {
    "maxIngestedEventTime" : "2013-05-09T18:24:09.007Z",
  }
} ]
```