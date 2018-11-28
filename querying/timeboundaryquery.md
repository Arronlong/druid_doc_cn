# 时间边界查询

时间边界查询返回数据集的最早和最新数据点。语法是：

```json
{
    "queryType" : "timeBoundary",
    "dataSource": "sample_datasource",
    "bound"     : < "maxTime" | "minTime" > # optional, defaults to returning both timestamps if not set 
    "filter"    : { "type": "and", "fields": [<filter>, <filter>, ...] } # optional
}
```

时间边界查询有3个主要部分：

| 属性     | 描述                                                         | 需要？ |
| -------- | ------------------------------------------------------------ | ------ |
| 查询类型 | 这个字符串应该始终是“timeBoundary”;这是德鲁伊首先想弄清楚如何解释查询 | 是     |
| 数据源   | 定义要查询的数据源的String或Object，非常类似于关系数据库中的表。有关更多信息，请参阅[DataSource](http://druid.io/docs/0.12.3/querying/datasource.html)。 | 是     |
| 界       | 可选，设置为`maxTime`或`minTime`仅返回最新或最早的时间戳。如果未设置，则默认返回两者 | 没有   |
| 过滤     | 见[滤波器](http://druid.io/docs/0.12.3/querying/filters.html) | 没有   |
| 上下文   | 见[上下文](http://druid.io/docs/0.12.3/querying/query-context.html) | 没有   |

结果的格式是：

```json
[ {
  "timestamp" : "2013-05-09T18:24:00.000Z",
  "result" : {
    "minTime" : "2013-05-09T18:24:00.000Z",
    "maxTime" : "2013-05-09T18:37:00.000Z"
  }
} ]
```