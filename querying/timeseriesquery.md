

# 时间序列查询

这些类型的查询采用时间序列查询对象并返回JSON对象数组，其中每个对象表示时间序列查询所请求的值。

示例时间序列查询对象如下所示：

```json
{
  "queryType": "timeseries",
  "dataSource": "sample_datasource",
  "granularity": "day",
  "descending": "true",
  "filter": {
    "type": "and",
    "fields": [
      { "type": "selector", "dimension": "sample_dimension1", "value": "sample_value1" },
      { "type": "or",
        "fields": [
          { "type": "selector", "dimension": "sample_dimension2", "value": "sample_value2" },
          { "type": "selector", "dimension": "sample_dimension3", "value": "sample_value3" }
        ]
      }
    ]
  },
  "aggregations": [
    { "type": "longSum", "name": "sample_name1", "fieldName": "sample_fieldName1" },
    { "type": "doubleSum", "name": "sample_name2", "fieldName": "sample_fieldName2" }
  ],
  "postAggregations": [
    { "type": "arithmetic",
      "name": "sample_divide",
      "fn": "/",
      "fields": [
        { "type": "fieldAccess", "name": "postAgg__sample_name1", "fieldName": "sample_name1" },
        { "type": "fieldAccess", "name": "postAgg__sample_name2", "fieldName": "sample_name2" }
      ]
    }
  ],
  "intervals": [ "2012-01-01T00:00:00.000/2012-01-03T00:00:00.000" ]
}
```

时间序列查询有7个主要部分：

| 属性             | 描述                                                         | 需要？ |
| ---------------- | ------------------------------------------------------------ | ------ |
| 查询类型         | 该字符串应始终为“timeseries”; 这是德鲁伊首先想弄清楚如何解释查询 | 是     |
| 数据源           | 定义要查询的数据源的String或Object，非常类似于关系数据库中的表。有关更多信息，请参阅[DataSource](http://druid.io/docs/0.12.3/querying/datasource.html)。 | 是     |
| 降               | 是否降序排序结果。默认为`false`（升序）。                    | 没有   |
| 间隔             | 表示ISO-8601间隔的JSON对象。这定义了运行查询的时间范围。     | 是     |
| 粒度             | 定义桶查询结果的粒度。见[粒度](http://druid.io/docs/0.12.3/querying/granularities.html) | 是     |
| 过滤             | 见[滤波器](http://druid.io/docs/0.12.3/querying/filters.html) | 没有   |
| 聚合             | 请参阅[聚合](http://druid.io/docs/0.12.3/querying/aggregations.html) | 没有   |
| postAggregations | 请参阅[后聚合](http://druid.io/docs/0.12.3/querying/post-aggregations.html) | 没有   |
| 上下文           | 见[上下文](http://druid.io/docs/0.12.3/querying/query-context.html) | 没有   |

要将它们全部拉到一起，上面的查询将从“sample_datasource”表返回2个数据点，一个在2012-01-01和2012-01-03之间，每天一个。每个数据点将是sample_fieldName1的（长）总和，sample_fieldName2的（double）和以及sample_fieldName1的（double）结果除以过滤器集的sample_fieldName2。输出如下：

```json
[
  {
    "timestamp": "2012-01-01T00:00:00.000Z",
    "result": { "sample_name1": <some_value>, "sample_name2": <some_value>, "sample_divide": <some_value> } 
  },
  {
    "timestamp": "2012-01-02T00:00:00.000Z",
    "result": { "sample_name1": <some_value>, "sample_name2": <some_value>, "sample_divide": <some_value> }
  }
]
```

#### 零填充

时间序列查询通常用零填充空的内部时间桶。例如，如果您针对时间间隔2012-01-01 / 2012-01-04发出“日”粒度时间序列查询，并且2012-01-02没有数据，您将收到：

```json
[
  {
    "timestamp": "2012-01-01T00:00:00.000Z",
    "result": { "sample_name1": <some_value> }
  },
  {
   "timestamp": "2012-01-02T00:00:00.000Z",
   "result": { "sample_name1": 0 }
  },
  {
    "timestamp": "2012-01-03T00:00:00.000Z",
    "result": { "sample_name1": <some_value> }
  }
]
```

完全位于数据间隔之外的时间桶不是零填充的。

您可以使用上下文标志“skipEmptyBuckets”禁用所有零填充。在此模式下，结果将省略2012-01-02的数据点。

具有此上下文标志集的查询将如下所示：

```json
{
  "queryType": "timeseries",
  "dataSource": "sample_datasource",
  "granularity": "day",
  "aggregations": [
    { "type": "longSum", "name": "sample_name1", "fieldName": "sample_fieldName1" }
  ],
  "intervals": [ "2012-01-01T00:00:00.000/2012-01-04T00:00:00.000" ],
  "context" : {
    "skipEmptyBuckets": "true"
  }
}
```