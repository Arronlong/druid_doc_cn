# TopN查询

TopN查询根据某些条件返回给定维度中的值的有序结果集。从概念上讲，它们可以被认为是具有[Ordering](http://druid.io/docs/0.12.3/querying/limitspec.html)规范的单个维度上的近似[GroupByQuery](http://druid.io/docs/0.12.3/querying/groupbyquery.html)。对于此用例，TopNs比GroupBys快得多，资源效率更高。这些类型的查询采用topN查询对象并返回JSON对象数组，其中每个对象表示topN查询所请求的值。

TopNs是近似值，因为每个节点将对其前K个结果进行排名，并且仅将那些前K个结果返回给代理。默认情况下，德鲁伊的K是`max(1000, threshold)`。在实践中，这意味着如果您要求排序的前1000个项目，前900个项目的正确性将是100％，并且不保证之后的结果排序。通过提高阈值可以使TopN更准确。

topN查询对象如下所示：

```json
{
  "queryType": "topN",
  "dataSource": "sample_data",
  "dimension": "sample_dim",
  "threshold": 5,
  "metric": "count",
  "granularity": "all",
  "filter": {
    "type": "and",
    "fields": [
      {
        "type": "selector",
        "dimension": "dim1",
        "value": "some_value"
      },
      {
        "type": "selector",
        "dimension": "dim2",
        "value": "some_other_val"
      }
    ]
  },
  "aggregations": [
    {
      "type": "longSum",
      "name": "count",
      "fieldName": "count"
    },
    {
      "type": "doubleSum",
      "name": "some_metric",
      "fieldName": "some_metric"
    }
  ],
  "postAggregations": [
    {
      "type": "arithmetic",
      "name": "average",
      "fn": "/",
      "fields": [
        {
          "type": "fieldAccess",
          "name": "some_metric",
          "fieldName": "some_metric"
        },
        {
          "type": "fieldAccess",
          "name": "count",
          "fieldName": "count"
        }
      ]
    }
  ],
  "intervals": [
    "2013-08-31T00:00:00.000/2013-09-03T00:00:00.000"
  ]
}
```

topN查询有11个部分。

| 属性             | 描述                                                         | 需要？ |
| ---------------- | ------------------------------------------------------------ | ------ |
| 查询类型         | 该字符串应始终为“topN”; 这是德鲁伊首先想弄清楚如何解释查询   | 是     |
| 数据源           | 定义要查询的数据源的String或Object，非常类似于关系数据库中的表。有关更多信息，请参阅[DataSource](http://druid.io/docs/0.12.3/querying/datasource.html)。 | 是     |
| 间隔             | 表示ISO-8601间隔的JSON对象。这定义了运行查询的时间范围。     | 是     |
| 粒度             | 定义桶查询结果的粒度。见[粒度](http://druid.io/docs/0.12.3/querying/granularities.html) | 是     |
| 过滤             | 见[滤波器](http://druid.io/docs/0.12.3/querying/filters.html) | 没有   |
| 聚合             | 请参阅[聚合](http://druid.io/docs/0.12.3/querying/aggregations.html) | 没有   |
| postAggregations | 请参阅[后聚合](http://druid.io/docs/0.12.3/querying/post-aggregations.html) | 没有   |
| 尺寸             | 定义您希望顶部采用的维度的String或JSON对象。有关更多信息，请参阅[DimensionSpecs](http://druid.io/docs/0.12.3/querying/dimensionspecs.html) | 是     |
| 阈               | 在topN中定义N的整数（即在顶部列表中需要多少个结果）          | 是     |
| 公               | 一个String或JSON对象，指定要对顶部列表进行排序的度量标准。有关详细信息，请参阅[TopNMetricSpec](http://druid.io/docs/0.12.3/querying/topnmetricspec.html)。 | 是     |
| 上下文           | 见[上下文](http://druid.io/docs/0.12.3/querying/query-context.html) | 没有   |

请注意，上下文JSON对象也可用于topN查询，并且应该与时间序列情况一样谨慎使用。结果的格式如下：

```json
[
  {
    "timestamp": "2013-08-31T00:00:00.000Z",
    "result": [
      {
        "dim1": "dim1_val",
        "count": 111,
        "some_metrics": 10669,
        "average": 96.11711711711712
      },
      {
        "dim1": "another_dim1_val",
        "count": 88,
        "some_metrics": 28344,
        "average": 322.09090909090907
      },
      {
        "dim1": "dim1_val3",
        "count": 70,
        "some_metrics": 871,
        "average": 12.442857142857143
      },
      {
        "dim1": "dim1_val4",
        "count": 62,
        "some_metrics": 815,
        "average": 13.14516129032258
      },
      {
        "dim1": "dim1_val5",
        "count": 60,
        "some_metrics": 2787,
        "average": 46.45
      }
    ]
  }
]
```

### 多值维度上的行为

topN查询可以对多值维度进行分组。在对多值维度进行分组时，匹配行中的*所有*值将用于为每个值生成一个组。查询可以返回比行更多的组。例如，在尺寸a TOPN `tags`与过滤器`"t1" AND "t3"`将匹配仅ROW1，并与三个组生成结果：`t1`，`t2`，和`t3`。如果您只需要包含与过滤器匹配的值，则可以使用[过滤的dimensionSpec](http://druid.io/docs/0.12.3/querying/dimensionspecs.html#filtered-dimensionspecs)。这也可以提高性能。

有关详细信息，请参阅[多值维度](http://druid.io/docs/0.12.3/querying/multi-value-dimensions.html)。

### 别名

当前的TopN算法是近似算法。返回每个段的前1000个本地结果以进行合并以确定全局topN。因此，topN算法在秩和结果中都是近似的。*仅当有超过1000个DIM值时，*近似结果才*适用*。在维度上具有少于1000个唯一维度值的topN可以被认为在排名中是准确的并且在聚合中是准确的。

可以通过服务器参数从默认值1000修改阈值，该参数`druid.query.topN.minTopNThreshold`需要重新启动服务器才能生效或`minTopNThreshold`在查询上下文中设置，该查询上下文对每个查询生效。

如果您想要一个高基数的前100个，由一些低基数，均匀分布的维度排序的均匀分布的维度，您可能会获得缺少数据的聚合。

换句话说，topN的最佳用例是当你可以确信整体结果统一在顶部时。例如，如果特定站点ID在每天每小时的某个度量标准中位于前10位，那么它可能在多天内在topN中准确。但是，如果一个站点在任何给定时间内几乎不在前1000名，但整个查询粒度在前500名（例如：一个站点在数据集中与具有高度周期性数据的站点混合得到高度统一的流量），那么top500查询可能没有该特定网站的确切排名，并且可能对该特定网站的聚合不准确。

在继续本节之前，请考虑您是否确实需要确切的结果。获得准确的结果是一个非常耗费资源的过程。对于绝大多数“有用的”数据结果，近似topN算法提供了足够的准确性。

希望获得*精确排名且精确聚合*顶部N超过1000个唯一值的用户应发出groupBy查询并自行对结果进行排序。对于高基数维度，这在计算上非常昂贵。

如果用户能够容忍大于1000个唯一值的维度的*近似排名* topN，但需要*精确聚合，则*可以发出两个查询。一个用于获取近似的topN维度值，另一个用于维度选择过滤器，仅使用第一个的topN结果。

#### 示例第一个查询：

```json
{
    "aggregations": [
             {
                 "fieldName": "L_QUANTITY_longSum",
                 "name": "L_QUANTITY_",
                 "type": "longSum"
             }
    ],
    "dataSource": "tpch_year",
    "dimension":"l_orderkey",
    "granularity": "all",
    "intervals": [
        "1900-01-09T00:00:00.000Z/2992-01-10T00:00:00.000Z"
    ],
    
    "metric": "L_QUANTITY_",
    "queryType": "topN",
    "threshold": 2
}
```

#### 示例第二个查询：

```json
{
    "aggregations": [
             {
                 "fieldName": "L_TAX_doubleSum",
                 "name": "L_TAX_",
                 "type": "doubleSum"
             },
             {
                 "fieldName": "L_DISCOUNT_doubleSum",
                 "name": "L_DISCOUNT_",
                 "type": "doubleSum"
             },
             {
                 "fieldName": "L_EXTENDEDPRICE_doubleSum",
                 "name": "L_EXTENDEDPRICE_",
                 "type": "doubleSum"
             },
             {
                 "fieldName": "L_QUANTITY_longSum",
                 "name": "L_QUANTITY_",
                 "type": "longSum"
             },
             {
                 "name": "count",
                 "type": "count"
             }
    ],
    "dataSource": "tpch_year",
    "dimension":"l_orderkey",
    "filter": {
        "fields": [
            {
                "dimension": "l_orderkey",
                "type": "selector",
                "value": "103136"
            },
            {
                "dimension": "l_orderkey",
                "type": "selector",
                "value": "1648672"
            }
        ],
        "type": "or"
    },
    "granularity": "all",
    "intervals": [
        "1900-01-09T00:00:00.000Z/2992-01-10T00:00:00.000Z"
    ],
    "metric": "L_QUANTITY_",
    "queryType": "topN",
    "threshold": 2
}
```