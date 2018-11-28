# 聚合

聚合可以在摄取时提供，作为摄取规范的一部分，作为在进入德鲁伊之前汇总数据的一种方式。聚合也可以在查询时指定为许多查询的一部分。

可用的聚合是：

### 计数聚合器

`count` 计算与过滤器匹配的德鲁伊行数。

```json
{ "type" : "count", "name" : <output_name> }
```

请注意，计数聚合器计算德鲁伊行的数量，这并不总是反映摄取的原始事件的数量。这是因为可以将Druid配置为在摄取时间内汇总数据。要计算摄取的数据行数，请在摄取时包含计数聚合器，在查询时包含longSum聚合器。

### 总和聚合器

#### `longSum` 聚合

计算值的总和为64位有符号整数

```json
{ "type" : "longSum", "name" : <output_name>, "fieldName" : <metric_name> }
```

`name`- 求和值的输出名称 `fieldName`- 要求的度量列的名称

#### `doubleSum` 聚合

计算并将值的总和存储为64位浮点值。相近`longSum`

```json
{ "type" : "doubleSum", "name" : <output_name>, "fieldName" : <metric_name> }
```

#### `floatSum` 聚合

计算并将值的总和存储为32位浮点值。与`longSum`和相似`doubleSum`

```json
{ "type" : "floatSum", "name" : <output_name>, "fieldName" : <metric_name> }
```

### 最小/最大聚合器

#### `doubleMin` 聚合

`doubleMin` 计算所有度量标准值的最小值和Double.POSITIVE_INFINITY

```json
{ "type" : "doubleMin", "name" : <output_name>, "fieldName" : <metric_name> }
```

#### `doubleMax` 聚合

`doubleMax` 计算所有度量标准值的最大值和Double.NEGATIVE_INFINITY

```json
{ "type" : "doubleMax", "name" : <output_name>, "fieldName" : <metric_name> }
```

#### `floatMin` 聚合

`floatMin` 计算所有度量标准值的最小值和Float.POSITIVE_INFINITY

```json
{ "type" : "floatMin", "name" : <output_name>, "fieldName" : <metric_name> }
```

#### `floatMax` 聚合

`floatMax` 计算所有度量标准值的最大值和Float.NEGATIVE_INFINITY

```json
{ "type" : "floatMax", "name" : <output_name>, "fieldName" : <metric_name> }
```

#### `longMin` 聚合

`longMin` 计算所有度量标准值和Long.MAX_VALUE的最小值

```json
{ "type" : "longMin", "name" : <output_name>, "fieldName" : <metric_name> }
```

#### `longMax` 聚合

`longMax` 计算所有度量标准值的最大值和Long.MIN_VALUE

```json
{ "type" : "longMax", "name" : <output_name>, "fieldName" : <metric_name> }
```

### 第一个/最后一个聚合器

First和Last聚合器不能用于摄取规范，只应作为查询的一部分指定。

请注意，在启用汇总的情况下创建的段上具有第一个/最后一个聚合器的查询将返回已汇总的值，而不是原始摄取数据中的最后一个值。

#### `doubleFirst` 聚合

`doubleFirst` 使用最小时间戳计算度量标准值，如果不存在行，则计算0

```json
{
  "type" : "doubleFirst",
  "name" : <output_name>,
  "fieldName" : <metric_name>
}
```

#### `doubleLast` 聚合

`doubleLast` 使用最大时间戳计算度量标准值，如果不存在行，则计算0

```json
{
  "type" : "doubleLast",
  "name" : <output_name>,
  "fieldName" : <metric_name>
}
```

#### `floatFirst` 聚合

`floatFirst` 使用最小时间戳计算度量标准值，如果不存在行，则计算0

```json
{
  "type" : "floatFirst",
  "name" : <output_name>,
  "fieldName" : <metric_name>
}
```

#### `floatLast` 聚合

`floatLast` 使用最大时间戳计算度量标准值，如果不存在行，则计算0

```json
{
  "type" : "floatLast",
  "name" : <output_name>,
  "fieldName" : <metric_name>
}
```

#### `longFirst` 聚合

`longFirst` 使用最小时间戳计算度量标准值，如果不存在行，则计算0

```json
{
  "type" : "longFirst",
  "name" : <output_name>,
  "fieldName" : <metric_name>
}
```

#### `longLast` 聚合

`longLast` 使用最大时间戳计算度量标准值，如果不存在行，则计算0

```json
{ 
  "type" : "longLast",
  "name" : <output_name>, 
  "fieldName" : <metric_name>,
}
```

### JavaScript聚合器

计算一组列上的任意JavaScript函数（允许使用度量和维度）。您的JavaScript函数应返回浮点值。

```json
{ "type": "javascript",
  "name": "<output_name>",
  "fieldNames"  : [ <column1>, <column2>, ... ],
  "fnAggregate" : "function(current, column1, column2, ...) {
                     <updates partial aggregate (current) based on the current row values>
                     return <updated partial aggregate>
                   }",
  "fnCombine"   : "function(partialA, partialB) { return <combined partial results>; }",
  "fnReset"     : "function()                   { return <initial value>; }"
}
```

**例**

```json
{
  "type": "javascript",
  "name": "sum(log(x)*y) + 10",
  "fieldNames": ["x", "y"],
  "fnAggregate" : "function(current, a, b)      { return current + (Math.log(a) * b); }",
  "fnCombine"   : "function(partialA, partialB) { return partialA + partialB; }",
  "fnReset"     : "function()                   { return 10; }"
}
```

默认情况下禁用基于JavaScript的功能。有关使用德鲁伊JavaScript功能的[指南](http://druid.io/docs/0.12.3/development/javascript.html)，请参阅德鲁伊[JavaScript编程指南](http://druid.io/docs/0.12.3/development/javascript.html)，包括如何启用它的说明。

## 近似聚合

### 基数聚合器

计算一组德鲁伊维度的基数，使用HyperLogLog估计基数。请注意，此聚合器比使用hyperUnique聚合器索引列要慢得多。此聚合器还在维列上运行，这意味着无法从数据集中删除字符串维度以改进汇总。通常，如果您不关心维度的各个值，我们强烈建议您使用hyperUnique聚合器而不是基数聚合器。

```json
{
  "type": "cardinality",
  "name": "<output_name>",
  "fields": [ <dimension1>, <dimension2>, ... ],
  "byRow": <false | true> # (optional, defaults to false),
  "round": <false | true> # (optional, defaults to false)
}
```

“fields”列表的每个单独元素可以是String或[DimensionSpec](http://druid.io/docs/0.12.3/querying/dimensionspecs.html)。字段列表中的字符串维度等同于DefaultDimensionSpec（无转换）。

HyperLogLog算法生成带有一些错误的十进制估计值。可以将“round”设置为true以将估计值四舍五入为整数。请注意，即使使用舍入，基数仍然是估计值。“round”字段仅影响查询时行为，并在摄取时被忽略。

#### 基数按价值计算

当设置`byRow`为`false`（默认值）时，它计算由所有给定维度的所有维度值的并集组成的集合的基数。

- 对于单个维度，这相当于

```sql
SELECT COUNT(DISTINCT(dimension)) FROM <datasource>
```

- 对于多个维度，这相当于类似的东西

```sql
SELECT COUNT(DISTINCT(value)) FROM (
  SELECT dim_1 as value FROM <datasource>
  UNION
  SELECT dim_2 as value FROM <datasource>
  UNION
  SELECT dim_3 as value FROM <datasource>
)
```

#### 行基数

当设置`byRow`为`true`它时，按行计算基数，即不同维度组合的基数。这相当于类似的东西

```sql
SELECT COUNT(*) FROM ( SELECT DIM1, DIM2, DIM3 FROM <datasource> GROUP BY DIM1, DIM2, DIM3 )
```

**例**

确定人们居住或来自的不同国家的数量。

```json
{
  "type": "cardinality",
  "name": "distinct_countries",
  "fields": [ "country_of_origin", "country_of_residence" ]
}
```

确定不同人的数量（即名字和姓氏的组合）。

```json
{
  "type": "cardinality",
  "name": "distinct_people",
  "fields": [ "first_name", "last_name" ],
  "byRow" : true
}
```

确定姓氏的不同起始字符数

```json
{
  "type": "cardinality",
  "name": "distinct_last_name_first_char",
  "fields": [
    {
     "type" : "extraction",
     "dimension" : "last_name",
     "outputName" :  "last_name_first_char",
     "extractionFn" : { "type" : "substring", "index" : 0, "length" : 1 }
    }
  ],
  "byRow" : true
}
```

### HyperUnique聚合器

使用[HyperLogLog](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf)计算在索引时已聚合为“hyperUnique”度量的维度的估计基数。

```json
{ 
  "type" : "hyperUnique",
  "name" : <output_name>,
  "fieldName" : <metric_name>,
  "isInputHyperUnique" : false,
  "round" : false
}
```

可以将“isInputHyperUnique”设置为true以索引预先计算的HLL（预期来自druid-hll的Base64编码输出）。“isInputHyperUnique”字段仅影响摄取时行为，并在查询时被忽略。

HyperLogLog算法生成带有一些错误的十进制估计值。可以将“round”设置为true以将估计值四舍五入为整数。请注意，即使使用舍入，基数仍然是估计值。“round”字段仅影响查询时行为，并在摄取时被忽略。

有关更多近似聚合器，请参阅[theta草图](http://druid.io/docs/0.12.3/development/extensions-core/datasketches-aggregators.html)。

## 杂项聚合

### 过滤的聚合器

过滤的聚合器包装任何给定的聚合器，但仅聚合给定维度过滤器匹配的值。

这使得可以同时计算过滤和未过滤聚合的结果，而不必发出多个查询，并将这两个结果用作后聚合的一部分。

*注意：*如果只需要过滤结果，请考虑将过滤器放在查询本身上，这样会快得多，因为它不需要扫描所有数据。

```json
{
  "type" : "filtered",
  "filter" : {
    "type" : "selector",
    "dimension" : <dimension>,
    "value" : <dimension value>
  }
  "aggregator" : <aggregation>
}
```