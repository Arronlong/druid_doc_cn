# 分段元数据查询

段元数据查询返回有关以下内容的每段信息：

- 段中所有列的基数
- 段中字符串类型列的最小/最大值
- 段列的估计字节大小（如果它们以平面格式存储）
- 存储在段内的行数
- 细分涵盖的时间间隔
- 段中所有列的列类型
- 如果以平面格式存储，则估计总段字节大小
- 该部分是否卷起
- 细分ID

```json
{
  "queryType":"segmentMetadata",
  "dataSource":"sample_datasource",
  "intervals":["2013-01-01/2014-01-01"]
}
```

段元数据查询有几个主要部分：

| 属性                   | 描述                                                         | 需要？ |
| ---------------------- | ------------------------------------------------------------ | ------ |
| 查询类型               | 此字符串应始终为“segmentMetadata”;这是德鲁伊首先想弄清楚如何解释查询 | 是     |
| 数据源                 | 定义要查询的数据源的String或Object，非常类似于关系数据库中的表。有关更多信息，请参阅[DataSource](http://druid.io/docs/0.12.3/querying/datasource.html)。 | 是     |
| 间隔                   | 表示ISO-8601间隔的JSON对象。这定义了运行查询的时间范围。     | 没有   |
| 包括                   | 一个JSON对象，表示结果中应包含哪些列。默认为“全部”。         | 没有   |
| 合并                   | 将所有单个段元数据结果合并为单个结果                         | 没有   |
| 上下文                 | 见[上下文](http://druid.io/docs/0.12.3/querying/query-context.html) | 没有   |
| analysisTypes          | 字符串列表，指定应在结果中计算和返回哪些列属性（例如基数，大小）。默认为[“基数”，“间隔”，“minmax”]，但可以使用[段元数据查询配置](http://druid.io/docs/0.12.3/configuration/index.html#segment-metadata-query-config)覆盖。有关更多详细信息，请参阅[analysisTypes](http://druid.io/docs/0.12.3/querying/segmentmetadataquery.html#analysistypes)部分。 | 没有   |
| lenientAggregatorMerge | 如果为true，并且启用了“聚合器”analyzeType，则将合法地合并聚合器。请参阅下文了解详情。 | 没有   |

结果的格式是：

```json
[ {
  "id" : "some_id",
  "intervals" : [ "2013-05-13T00:00:00.000Z/2013-05-14T00:00:00.000Z" ],
  "columns" : {
    "__time" : { "type" : "LONG", "hasMultipleValues" : false, "size" : 407240380, "cardinality" : null, "errorMessage" : null },
    "dim1" : { "type" : "STRING", "hasMultipleValues" : false, "size" : 100000, "cardinality" : 1944, "errorMessage" : null },
    "dim2" : { "type" : "STRING", "hasMultipleValues" : true, "size" : 100000, "cardinality" : 1504, "errorMessage" : null },
    "metric1" : { "type" : "FLOAT", "hasMultipleValues" : false, "size" : 100000, "cardinality" : null, "errorMessage" : null }
  },
  "aggregators" : {
    "metric1" : { "type" : "longSum", "name" : "metric1", "fieldName" : "metric1" }
  },
  "queryGranularity" : {
    "type": "none"
  },
  "size" : 300000,
  "numRows" : 5000000
} ]
```

维度列将具有类型`STRING`。指标列将具有类型`FLOAT`或`LONG`底层复杂类型或名称，例如`hyperUnique`在复杂的度量的情况下。时间戳列将具有类型`LONG`。

如果该`errorMessage`字段为非null，则不应信任响应中的其他字段。他们的内容未定义。

只有尺寸（即具有类型`STRING`）的列才具有任何基数。其余列（时间戳和公制列）将显示基数`null`。

### 间隔

如果未指定间隔，则查询将使用跨越最近段结束时间之前的可配置时段的默认时间间隔。

此默认时间段的长度在代理配置中通过：druid.query.segmentMetadata.defaultHistory设置

### 包括

toInclude对象有3种类型。

#### 所有

语法如下：

```json
"toInclude": { "type": "all"}
```

#### 没有

语法如下：

```json
"toInclude": { "type": "none"}
```

#### 名单

语法如下：

```json
"toInclude": { "type": "list", "columns": [<string list of column names>]}
```

### analysisTypes

这是一个属性列表，用于确定有关列返回的信息量，即要对列执行的分析。

默认情况下，将使用“基数”，“间隔”和“最小值”类型。如果不需要属性，则从该列表中省略该属性将导致更有效的查询。

可以通过以下方式在代理配置中设置默认分析类型： `druid.query.segmentMetadata.defaultAnalysisTypes`

列分析的类型如下所述：

#### 基数

- `cardinality`在结果中将返回每列的估计基数。仅与维列相关。

#### MINMAX

- 每列的估计最小/最大值。仅与维列相关。

#### 尺寸

- `size` 在结果中将包含估计的总段字节大小，就像数据以文本格式存储一样

#### 间隔

- `intervals` 在结果中将包含与查询的段关联的间隔列表。

#### timestampSpec

- `timestampSpec`在结果中将包含存储在段中的数据的时间戳规范。如果段的timestampSpec未知或不可合并（如果启用了合并），则此值可以为null。

#### queryGranularity

- `queryGranularity`在结果中将包含存储在段中的数据的查询粒度。如果段的查询粒度未知或不可合并（如果启用了合并），则此值可以为null。

#### 聚合

- `aggregators`在结果中将包含可用于查询度量标准列的聚合器列表。如果聚合器未知或不可合并（如果启用了合并），则此值可能为null。
- 合并可以是严格的或宽松的。有关详细信息，请参阅下面的*lenientAggregatorMerge*。
- 结果的形式是列名到聚合器的映射。

#### 卷起

- `rollup` 在结果中是true / false / null。
- 启用合并时，如果某些是汇总，而其他则不是，则结果为null。

### lenientAggregatorMerge

如果某些段具有未知聚合器，或者如果两个段对同一列使用不兼容的聚合器（例如，longSum更改为doubleSum），则可能会发生跨段的聚合器元数据之间的冲突。

聚合器可以严格合并（默认）或宽松合并。通过严格合并，如果存在任何具有未知聚合器的段或任何类型的冲突，则合并的聚合器列表将是`null`。通过宽松合并，将忽略具有未知聚合器的段，并且聚合器之间的冲突将仅使该特定列的聚合器无效。

特别是，通过宽松的合并，可以使用invidiual列的聚合器`null`。严格合并不会发生这种情况。