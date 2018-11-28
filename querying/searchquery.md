# 搜索查询

搜索查询返回与搜索规范匹配的维度值。

```json
{
  "queryType": "search",
  "dataSource": "sample_datasource",
  "granularity": "day",
  "searchDimensions": [
    "dim1",
    "dim2"
  ],
  "query": {
    "type": "insensitive_contains",
    "value": "Ke"
  },
  "sort" : {
    "type": "lexicographic"
  },
  "intervals": [
    "2013-01-01T00:00:00.000/2013-01-03T00:00:00.000"
  ]
}
```

搜索查询有几个主要部分：

| 属性             | 描述                                                         | 需要？           |
| ---------------- | ------------------------------------------------------------ | ---------------- |
| 查询类型         | 这个字符串应该始终是“搜索”; 这是德鲁伊首先想弄清楚如何解释查询。 | 是               |
| 数据源           | 定义要查询的数据源的String或Object，非常类似于关系数据库中的表。有关更多信息，请参阅[DataSource](http://druid.io/docs/0.12.3/querying/datasource.html)。 | 是               |
| 粒度             | 定义查询的粒度。见[粒度](http://druid.io/docs/0.12.3/querying/granularities.html)。 | 是               |
| 过滤             | 见[滤波器](http://druid.io/docs/0.12.3/querying/filters.html)。 | 没有             |
| 限制             | 定义要返回的搜索结果的每个历史节点（解析为int）的最大数量。  | 不（默认为1000） |
| 间隔             | 表示ISO-8601间隔的JSON对象。这定义了运行查询的时间范围。     | 是               |
| searchDimensions | 运行搜索的维度。排除这意味着搜索将在所有维度上运行。         | 没有             |
| 询问             | 请参阅[SearchQuerySpec](http://druid.io/docs/0.12.3/querying/searchqueryspec.html)。 | 是               |
| 分类             | 一个对象，指定应如何对搜索结果进行排序。 可能的类型是“lexicographic”（默认排序），“alphanumeric”，“strlen”和“numeric”。 有关详细信息，请参阅[排序顺序](http://druid.io/docs/0.12.3/querying/sorting-orders.html)。 | 没有             |
| 上下文           | 见[上下文](http://druid.io/docs/0.12.3/querying/query-context.html) | 没有             |

结果的格式是：

```json
[
  {
    "timestamp": "2012-01-01T00:00:00.000Z",
    "result": [
      {
        "dimension": "dim1",
        "value": "Ke$ha",
        "count": 3
      },
      {
        "dimension": "dim2",
        "value": "Ke$haForPresident",
        "count": 1
      }
    ]
  },
  {
    "timestamp": "2012-01-02T00:00:00.000Z",
    "result": [
      {
        "dimension": "dim1",
        "value": "SomethingThatContainsKe",
        "count": 1
      },
      {
        "dimension": "dim2",
        "value": "SomethingElseThatContainsKe",
        "count": 2
      }
    ]
  }
]
```

### 实施细节

#### 策略

可以使用两种不同的策略执行搜索查询。默认策略由代理上的“druid.query.search.searchStrategy”运行时属性确定。这可以在查询上下文中使用“searchStrategy”覆盖。如果既未设置上下文字段也未设置属性，则将使用“useIndexes”策略。

- 默认情况下，“useIndexes”策略首先根据对位图索引的支持将搜索维度分为两组。然后，它将仅索引和基于游标的执行计划分别应用于支持位图和其他维度的维度组。仅索引计划仅使用索引进行搜索查询处理。对于每个维度，它读取每个维度值的位图索引，评估搜索谓词，最后检查时间间隔和过滤谓词。对于基于游标的执行计划，请参阅“cursorOnly”策略。仅索引计划显示大基数的搜索维度的低性能，这意味着搜索维度的大多数值是唯一的。
- “cursorOnly”策略生成基于游标的执行计划。此计划创建一个游标，该查询从queryableIndexSegment读取一行，然后计算搜索谓词。如果某些过滤器支持位图索引，则游标只能读取满足这些过滤器的行，从而节省了I / O开销。但是，使用低选择性的过滤器可能会很慢。
- “自动”策略使用基于成本的计划程序来选择最佳搜索策略。它估计仅索引和基于游标的执行计划的成本，并选择最佳的执行计划。目前，由于成本估算的开销，默认情况下不启用它。

#### 服务器配置

以下运行时属性适用：

| 属性                                | 描述               | 默认       |
| ----------------------------------- | ------------------ | ---------- |
| `druid.query.search.searchStrategy` | 默认搜索查询策略。 | useIndexes |

#### 查询上下文

以下查询上下文参数适用：

| 属性             | 描述                                                |
| ---------------- | --------------------------------------------------- |
| `searchStrategy` | 覆盖`druid.query.search.searchStrategy`此查询的值。 |