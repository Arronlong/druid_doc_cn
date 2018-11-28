## DataSketches聚合器

德鲁伊聚合器基于[datasketches](http://datasketches.github.io/)库。请注意，草图算法是近似的; 请参阅datasketches doc的“Accuracy”部分中的详细信息。在摄取时，此聚合器创建theta sketch对象，这些对象存储在Druid段中。从逻辑上讲，theta sketch对象可以被认为是Set数据结构。在查询时，草图被一起读取和聚合（设置联合）。最后，默认情况下，您会收到草图对象中唯一条目数的估计值。此外，您可以使用后聚合器在同一行中的草图列上进行并集，交集或差异。请注意，您可以使用`thetaSketch`如果没有使用相同的列进行聚集，它将返回列的估计基数。建议在摄取时使用它，以便更快地进行查询。

要使用datasketch聚合器，请确保在配置文件中[包含](http://druid.io/docs/0.12.3/operations/including-extensions.html)扩展名：

```text
druid.extensions.loadList=["druid-datasketches"]
```

### 聚合器

```json
{
  "type" : "thetaSketch",
  "name" : <output_name>,
  "fieldName" : <metric_name>,  
  "isInputThetaSketch": false,
  "size": 16384
 }
```

| 属性               | 描述                                                         | 需要？          |
| ------------------ | ------------------------------------------------------------ | --------------- |
| 类型               | 这个字符串应该始终是“thetaSketch”                            | 是              |
| 名称               | 用于计算的输出（结果）名称的String。                         | 是              |
| 字段名             | 用于摄取时使用的聚合器名称的String。                         | 是              |
| isInputThetaSketch | 如果输入数据包含theta sketch对象，则只应在索引时使用此选项。如果您使用德鲁伊之外的数据表库（例如Pig / Hive）来生成您正在摄入德鲁伊的数据，就会出现这种情况。 | 不，默认为false |
| 尺寸               | 必须是2的幂。在内部，size指草图对象将保留的最大条目数。尺寸越大意味着精度越高，但存储草图的空间越大。请注意，在使用特定大小进行索引后，德鲁伊将在段中保留草图，并且您将使用大于或等于查询时的大小。有关详细信息，请参阅[DataSketches站点](https://datasketches.github.io/docs/Theta/ThetaSize.html)。一般来说，我们建议只坚持默认大小。 | 不，默认为16384 |

### 发布聚合器

#### Sketch Estimator

```json
{
  "type"  : "thetaSketchEstimate",
  "name": <output name>,
  "field"  : <post aggregator of type fieldAccess that refers to a thetaSketch aggregator or that of type thetaSketchSetOp>
}
```

#### 草图操作

```json
{
  "type"  : "thetaSketchSetOp",
  "name": <output name>,
  "func": <UNION|INTERSECT|NOT>,
  "fields"  : <array of fieldAccess type post aggregators to access the thetaSketch aggregators or thetaSketchSetOp type post aggregators to allow arbitrary combination of set operations>,
  "size": <16384 by default, must be max of size from sketches in fields input>
}
```

### 例子

假设您有一个包含（timestamp，product，user_id）的数据集。你想回答像这样的问题

有多少独特用户访问过产品A？有多少独特用户访问过产品A和产品B？

要回答上述问题，您可以使用以下聚合器索引数据。

```json
{ "type": "thetaSketch", "name": "user_id_sketch", "fieldName": "user_id" }
```

那么，样本查询，有多少独特用户访问了产品A？

```json
{
  "queryType": "groupBy",
  "dataSource": "test_datasource",
  "granularity": "ALL",
  "dimensions": [],
  "aggregations": [
    { "type": "thetaSketch", "name": "unique_users", "fieldName": "user_id_sketch" }
  ],
  "filter": { "type": "selector", "dimension": "product", "value": "A" },
  "intervals": [ "2014-10-19T00:00:00.000Z/2014-10-22T00:00:00.000Z" ]
}
```

样本查询，有多少唯一身份用户访问了产品A和B？

```json
{
  "queryType": "groupBy",
  "dataSource": "test_datasource",
  "granularity": "ALL",
  "dimensions": [],
  "filter": {
    "type": "or",
    "fields": [
      {"type": "selector", "dimension": "product", "value": "A"},
      {"type": "selector", "dimension": "product", "value": "B"}
    ]
  },
  "aggregations": [
    {
      "type" : "filtered",
      "filter" : {
        "type" : "selector",
        "dimension" : "product",
        "value" : "A"
      },
      "aggregator" :     {
        "type": "thetaSketch", "name": "A_unique_users", "fieldName": "user_id_sketch"
      }
    },
    {
      "type" : "filtered",
      "filter" : {
        "type" : "selector",
        "dimension" : "product",
        "value" : "B"
      },
      "aggregator" :     {
        "type": "thetaSketch", "name": "B_unique_users", "fieldName": "user_id_sketch"
      }
    }
  ],
  "postAggregations": [
    {
      "type": "thetaSketchEstimate",
      "name": "final_unique_users",
      "field":
      {
        "type": "thetaSketchSetOp",
        "name": "final_unique_users_sketch",
        "func": "INTERSECT",
        "fields": [
          {
            "type": "fieldAccess",
            "fieldName": "A_unique_users"
          },
          {
            "type": "fieldAccess",
            "fieldName": "B_unique_users"
          }
        ]
      }
    }
  ],
  "intervals": [
    "2014-10-19T00:00:00.000Z/2014-10-22T00:00:00.000Z"
  ]
}
```

#### 保留分析示例

假设您想回答一个问题，例如“有多少独特用户在特定时间段内执行了特定操作，还在不同时间段内执行了另一项特定操作？”

例如，“在第1周注册了多少个独立用户，并在第2周购买了什么？”

使用`(timestamp, product, user_id)`示例数据集，数据将使用以下聚合器编制索引，如上例所示：

```json
{ "type": "thetaSketch", "name": "user_id_sketch", "fieldName": "user_id" }
```

以下查询表示：

“2014年10月1日至2014年10月10日期间访问过产品A的独特用户中，有多少用户在2014年8月10日至2014年10月14日期间再次访问了产品A？”

```json
{
  "queryType": "groupBy",
  "dataSource": "test_datasource",
  "granularity": "ALL",
  "dimensions": [],
  "filter": {
    "type": "or",
    "fields": [
      {"type": "selector", "dimension": "product", "value": "A"}
    ]
  },
  "aggregations": [
    {
      "type" : "filtered",
      "filter" : {
        "type" : "and",
        "fields" : [
          {
            "type" : "selector",
            "dimension" : "product",
            "value" : "A"
          },
          {
            "type" : "interval",
            "dimension" : "__time",
            "intervals" :  ["2014-10-01T00:00:00.000Z/2014-10-07T00:00:00.000Z"]
          }
        ]
      },
      "aggregator" :     {
        "type": "thetaSketch", "name": "A_unique_users_week_1", "fieldName": "user_id_sketch"
      }
    },
    {
      "type" : "filtered",
      "filter" : {
        "type" : "and",
        "fields" : [
          {
            "type" : "selector",
            "dimension" : "product",
            "value" : "A"
          },
          {
            "type" : "interval",
            "dimension" : "__time",
            "intervals" :  ["2014-10-08T00:00:00.000Z/2014-10-14T00:00:00.000Z"]
          }
        ]
      },
      "aggregator" : {
        "type": "thetaSketch", "name": "A_unique_users_week_2", "fieldName": "user_id_sketch"
      }
    },
  ],
  "postAggregations": [
    {
      "type": "thetaSketchEstimate",
      "name": "final_unique_users",
      "field":
      {
        "type": "thetaSketchSetOp",
        "name": "final_unique_users_sketch",
        "func": "INTERSECT",
        "fields": [
          {
            "type": "fieldAccess",
            "fieldName": "A_unique_users_week_1"
          },
          {
            "type": "fieldAccess",
            "fieldName": "A_unique_users_week_2"
          }
        ]
      }
    }
  ],
  "intervals": ["2014-10-01T00:00:00.000Z/2014-10-14T00:00:00.000Z"]
}
```