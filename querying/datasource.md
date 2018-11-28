## 数据源

数据源是数据库表的德鲁伊等价物。但是，查询也可以伪装成数据源，提供类似子查询的功能。查询数据源目前仅由[GroupBy](http://druid.io/docs/0.12.3/querying/groupbyquery.html)查询支持。

### 表数据源

表数据源是最常见的类型。它由字符串或完整结构表示：

```json
{
    "type": "table",
    "name": "<string_value>"
}
```

### 联盟数据源

此数据源将两个或多个表数据源结合在一起。

```json
{
       "type": "union",
       "dataSources": ["<string_value1>", "<string_value2>", "<string_value3>", ... ]
}
```

请注意，正在联合的数据源应具有相同的架构。联合查询应始终发送到代理/路由器节点，历史节点*不*直接支持。

### 查询数据源

这用于嵌套的groupBys，目前仅支持groupBys。

```json
{
    "type": "query",
    "query": {
        "type": "groupBy",
        ...
    }
}
```