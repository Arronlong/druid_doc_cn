# 多值维度

德鲁伊支持“多值”字符串维度。当输入字段包含值数组而不是单个值（ee JSON数组或包含一个或多个`listDelimiter`字符的TSV字段）时，会生成这些值。

本文档描述了groupBy（topN具有类似行为）查询对多值维度的行为，当它们被用作按组分类的维度时。有关内部表示的详细信息，请参阅段中多值列的 [部分](http://druid.io/docs/0.12.3/design/segments.html#multi-value-columns)。

## 查询多值维度

假设您有一个dataSource，其中包含以下行的段，并且调用了多值维`tags`。

```text
{"timestamp": "2011-01-12T00:00:00.000Z", "tags": ["t1","t2","t3"]}  #row1
{"timestamp": "2011-01-13T00:00:00.000Z", "tags": ["t3","t4","t5"]}  #row2
{"timestamp": "2011-01-14T00:00:00.000Z", "tags": ["t5","t6","t7"]}  #row3
{"timestamp": "2011-01-14T00:00:00.000Z", "tags": []}                #row4
```

### 过滤

所有查询类型以及[过滤的聚合](http://druid.io/docs/0.12.3/querying/aggregations.html#filtered-aggregator)器都可以过滤多值维度。过滤器遵循多值维度的这些规则：

- 如果多值维度的任何值与过滤器匹配，则值过滤器（如“selector”，“bound”和“in”）将匹配一行。
- 如果尺寸有任何重叠，则“列比较”过滤器将匹配一行。
- 匹配`null`或`""`（空字符串）的值过滤器将匹配多值维度中的空单元格。
- 逻辑表达式过滤器的行为方式与单值维度相同：如果所有基础过滤器与该行匹配，则“和”匹配行; 如果任何基础过滤器与该行匹配，则“或”匹配一行; 如果基础过滤器与行不匹配，则“not”匹配一行。

例如，此“或”过滤器将匹配上面数据集的row1和row2，但不匹配row3：

```text
{
  "type": "or",
  "fields": [
    {
      "type": "selector",
      "dimension": "tags",
      "value": "t1"
    },
    {
      "type": "selector",
      "dimension": "tags",
      "value": "t3"
    }
  ]
}
```

这个“和”过滤器只匹配上面数据集的row1：

```text
{
  "type": "and",
  "fields": [
    {
      "type": "selector",
      "dimension": "tags",
      "value": "t1"
    },
    {
      "type": "selector",
      "dimension": "tags",
      "value": "t3"
    }
  ]
}
```

此“选择器”过滤器将匹配上面数据集的第4行：

```text
{
  "type": "selector",
  "dimension": "tags",
  "value": null
}
```

### 分组

topN和groupBy查询可以对多值维度进行分组。在对多值维度进行分组时，匹配行中的*所有*值将用于为每个值生成一个组。查询可以返回比行更多的组。例如，在尺寸a TOPN `tags`与过滤器`"t1" AND "t3"`将匹配仅ROW1，并与三个组生成结果：`t1`，`t2`，和`t3`。如果您只需要包含与过滤器匹配的值，则可以使用[过滤的dimensionSpec](http://druid.io/docs/0.12.3/querying/dimensionspecs.html#filtered-dimensionspecs)。这也可以提高性能。

### 示例：没有过滤的GroupBy查询

有关详细信息，请参阅[GroupBy查询](http://druid.io/docs/0.12.3/querying/groupbyquery.html)。

```json
{
  "queryType": "groupBy",
  "dataSource": "test",
  "intervals": [
    "1970-01-01T00:00:00.000Z/3000-01-01T00:00:00.000Z"
  ],
  "granularity": {
    "type": "all"
  },
  "dimensions": [
    {
      "type": "default",
      "dimension": "tags",
      "outputName": "tags"
    }
  ],
  "aggregations": [
    {
      "type": "count",
      "name": "count"
    }
  ]
}
```

返回以下结果。

```json
[
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 1,
      "tags": "t1"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 1,
      "tags": "t2"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 2,
      "tags": "t3"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 1,
      "tags": "t4"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 2,
      "tags": "t5"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 1,
      "tags": "t6"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 1,
      "tags": "t7"
    }
  }
]
```

注意原始行如何“爆炸”成多行并合并。

### 示例：使用选择器查询过滤器的GroupBy查询

有关选择器查询过滤[器](http://druid.io/docs/0.12.3/querying/filters.html)的详细信

```json
{
  "queryType": "groupBy",
  "dataSource": "test",
  "intervals": [
    "1970-01-01T00:00:00.000Z/3000-01-01T00:00:00.000Z"
  ],
  "filter": {
    "type": "selector",
    "dimension": "tags",
    "value": "t3"
  },
  "granularity": {
    "type": "all"
  },
  "dimensions": [
    {
      "type": "default",
      "dimension": "tags",
      "outputName": "tags"
    }
  ],
  "aggregations": [
    {
      "type": "count",
      "name": "count"
    }
  ]
}
```

返回以下结果。

```json
[
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 1,
      "tags": "t1"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 1,
      "tags": "t2"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 2,
      "tags": "t3"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 1,
      "tags": "t4"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 1,
      "tags": "t5"
    }
  }
]
```

您可能会惊讶地看到结果中包含“t1”，“t2”，“t4”和“t5”。之所以会发生这种情况，是因为在爆炸之前会在行上应 对于多值维度，“t3”的选择器过滤器将匹配row1和row2，然后进行爆炸。对于多值维度，如果多个值中的任何单个值与查询过滤器匹配，则查询过滤器会匹配一行。

### 示例：使用选择器查询过滤器和“维度”属性中的其他过滤器进行GroupBy查询

要解决上述问题并仅返回“t3”返回的行，您必须使用“过滤维度规范”，如下面的查询中所示。

查看在过滤dimensionSpecs部分[dimensionSpecs](http://druid.io/docs/0.12.3/querying/dimensionspecs.html#filtered-dimensionspecs)了解详情。

```json
{
  "queryType": "groupBy",
  "dataSource": "test",
  "intervals": [
    "1970-01-01T00:00:00.000Z/3000-01-01T00:00:00.000Z"
  ],
  "filter": {
    "type": "selector",
    "dimension": "tags",
    "value": "t3"
  },
  "granularity": {
    "type": "all"
  },
  "dimensions": [
    {
      "type": "listFiltered",
      "delegate": {
        "type": "default",
        "dimension": "tags",
        "outputName": "tags"
      },
      "values": ["t3"]
    }
  ],
  "aggregations": [
    {
      "type": "count",
      "name": "count"
    }
  ]
}
```

返回以下结果。

```json
[
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 2,
      "tags": "t3"
    }
  }
]
```

请注意，对于groupBy查询，您可以获得与[具有规范的](http://druid.io/docs/0.12.3/querying/having.html)类似结果，但使用过滤的dimensionSpec更有效，因为它应用于查询处理管道中的最低级别。具有规范应用于groupBy查询处理的最外层。