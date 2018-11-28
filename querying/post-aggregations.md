# 后聚合

后聚合是在德鲁伊出来的聚合值上应该进行的处理规范。如果您将后聚合作为查询的一部分包含在内，请确保包含后聚合器所需的所有聚合器。

有几个后聚合器可用。

### 算术后聚合器

算术后聚合器将提供的函数从左到右应用于给定字段。这些字段可以是聚合器或其他后聚合器。

支持的功能有`+`，`-`，`*`，`/`，和`quotient`。

**注意**：

- `/``0`除非除以`0`分子，否则除非总是返回。
- `quotient` 除法与常规浮点除法相似

算术后聚合器也可以指定一个`ordering`，它定义排序结果时结果值的顺序（例如，这对topN查询很有用）：

- 如果未`null`指定排序（或），则使用默认浮点排序。
- `numericFirst`排序总是首先返回有限值，然后是`NaN`最后的无限值。

算术后聚合的语法是：

```json
postAggregation : {
  "type"  : "arithmetic",
  "name"  : <output_name>,
  "fn"    : <arithmetic_function>,
  "fields": [<post_aggregator>, <post_aggregator>, ...],
  "ordering" : <null (default), or "numericFirst">
}
```

### 现场访问器后聚合器

这些后聚合器返回指定[聚合器](http://druid.io/docs/0.12.3/querying/aggregations.html)生成的值。

`fieldName`指查询的[聚合](http://druid.io/docs/0.12.3/querying/aggregations.html)部分中给出的[聚合器](http://druid.io/docs/0.12.3/querying/aggregations.html)的输出名称。对于复合聚合器，如“基数”和“hyperUnique”，`type`后聚合器确定后聚合器将返回的内容。使用类型“fieldAccess”返回原始聚合对象，或使用类型“finalizingFieldAccess”返回最终值，例如估计的基数。

```json
{ "type" : "fieldAccess", "name": <output_name>, "fieldName" : <aggregator_name> }
```

要么

```json
{ "type" : "finalizingFieldAccess", "name": <output_name>, "fieldName" : <aggregator_name> }
```

### 不断的后聚合器

常量后聚合器始终返回指定的值。

```json
{ "type"  : "constant", "name"  : <output_name>, "value" : <numerical_value> }
```

### 最好的/最少的后聚合器

`doubleGreatest`并`longGreatest`计算所有字段的最大值和Double.NEGATIVE_INFINITY。 `doubleLeast`并`longLeast`计算所有字段的最小值和Double.POSITIVE_INFINITY。

`doubleMax`聚合器和`doubleGreatest`后聚合器之间的区别在于`doubleMax`返回一个特定列的所有行`doubleGreatest`的最高值，同时返回一行中多个列的最高值。这些类似于SQL [MAX](https://dev.mysql.com/doc/refman/5.7/en/group-by-functions.html#function_max)和 [GREATEST](https://dev.mysql.com/doc/refman/5.7/en/comparison-operators.html#function_greatest)函数。

例：

```json
{
  "type"  : "doubleGreatest",
  "name"  : <output_name>,
  "fields": [<post_aggregator>, <post_aggregator>, ...]
}
```

### JavaScript后聚合器

将提供的JavaScript函数应用于给定字段。字段以给定顺序作为参数传递给JavaScript函数。

```json
postAggregation : {
  "type": "javascript",
  "name": <output_name>,
  "fieldNames" : [<aggregator_name>, <aggregator_name>, ...],
  "function": <javascript function>
}
```

JavaScript聚合器示例：

```json
{
  "type": "javascript",
  "name": "absPercent",
  "fieldNames": ["delta", "total"],
  "function": "function(delta, total) { return 100 * Math.abs(delta) / total; }"
}
```

默认情况下禁用基于JavaScript的功能。有关使用德鲁伊JavaScript功能的[指南](http://druid.io/docs/0.12.3/development/javascript.html)，请参阅德鲁伊[JavaScript编程指南](http://druid.io/docs/0.12.3/development/javascript.html)，包括如何启用它的说明。

### HyperUnique Cardinality后聚合器

hyperUniqueCardinality post聚合器用于包装hyperUnique对象，以便可以在post聚合中使用它。

```json
{
  "type"  : "hyperUniqueCardinality",
  "name": <output name>,
  "fieldName"  : <the name field value of the hyperUnique aggregator>
}
```

它可以用于样本计算，如下所示：

```json
  "aggregations" : [{
    {"type" : "count", "name" : "rows"},
    {"type" : "hyperUnique", "name" : "unique_users", "fieldName" : "uniques"}
  }],
  "postAggregations" : [{
    "type"   : "arithmetic",
    "name"   : "average_users_per_row",
    "fn"     : "/",
    "fields" : [
      { "type" : "hyperUniqueCardinality", "fieldName" : "unique_users" },
      { "type" : "fieldAccess", "name" : "rows", "fieldName" : "rows" }
    ]
  }]
```

此后聚合器将继承其引用的聚合器的舍入行为。请注意，此继承仅在您直接引用聚合器时才有效。例如，通过另一个后聚合器将导致用户指定的舍入行为丢失并默认为“无舍入”。

## 示例用法

在这个例子中，让我们使用post聚合器计算一个简单的百分比。让我们假设我们的数据集有一个名为“total”的指标。

查询JSON的格式如下：

```json
{
  ...
  "aggregations" : [
    { "type" : "count", "name" : "rows" },
    { "type" : "doubleSum", "name" : "tot", "fieldName" : "total" }
  ],
  "postAggregations" : [{
    "type"   : "arithmetic",
    "name"   : "average",
    "fn"     : "*",
    "fields" : [
       { "type"   : "arithmetic",
         "name"   : "div",
         "fn"     : "/",
         "fields" : [
           { "type" : "fieldAccess", "name" : "tot", "fieldName" : "tot" },
           { "type" : "fieldAccess", "name" : "rows", "fieldName" : "rows" }
         ]
       },
       { "type" : "constant", "name": "const", "value" : 100 }
    ]
  }]
  ...
}
```