# 转换规格

变换规范允许Druid在摄取期间过滤和转换输入数据。

## 句法

transformSpec的语法如下所示：

```text
"transformSpec": {
  "transforms: <List of transforms>,
  "filter": <filter>
}
```

| 属性 | 描述                                                         | 需要？ |
| ---- | ------------------------------------------------------------ | ------ |
| 变换 | 要应用于输入行的[变换](http://druid.io/docs/0.12.3/ingestion/transform-spec.html#transforms)列表。 | 没有   |
| 过滤 | 甲[滤波器](http://druid.io/docs/0.12.3/querying/filters.html)将被应用到输入行; 只会提取通过过滤器的行。 | 没有   |

## 变换

该`transforms`列表允许用户指定要对输入数据执行的一组列转换。

转换允许向输入行添加新字段。每个转换都有一个“名称”（新字段的名称），可由DimensionSpecs，AggregatorFactories等引用。

变换表现为“行函数”，将整行作为输入并输出列值。

如果变换与输入行中的字段具有相同的名称，则它将遮蔽原始字段。转换阴影字段仍然可以引用它们阴影的字段。这可以用于“就地”转换字段。

变换确实有一些局限性。它们只能引用实际输入行中的字段; 特别是，他们不能参考其他变换。他们不能删除字段，只添加它们。但是，它们可以使用包含所有空值的另一个字段来遮蔽字段，这将类似于删除字段。

请注意，转换是在过滤器之前应用的。

### 表达变换

德鲁伊目前支持一种变换，表达变换。

表达式转换具有以下语法：

```text
{
  "type": "expression",
  "name": <output field name>,
  "expression": <expr>
}
```

| 属性 | 描述                                                         | 需要？ |
| ---- | ------------------------------------------------------------ | ------ |
| 名称 | 表达式transform的输出字段名称。                              | 是     |
| 表达 | 一个[表达式](http://druid.io/docs/0.12.3/misc/math-expr.html)，将应用于输入行以生成变换输出字段的值。 | 没有   |

例如，以下表达式转换将“foo”添加到`page`输入数据中的列的值，并创建一`fooPage`列。

```text
    {
      "type": "expression",
      "name": "fooPage",
      "expression": "concat('foo' + page)"
    }
```

## 过滤

transformSpec允许Druid在摄取过程中过滤掉输入行。将无法获取无法通过过滤器的行。

可以使用任何德鲁伊标准[滤波器](http://druid.io/docs/0.12.3/querying/filters.html)。

请注意，过滤发生在变换之后，因此如果存在变换，过滤器将对变换的行进行操作，而不是原始输入数据。

例如，以下过滤器将仅摄取`country`列具有值“United States”的输入行：

```text
"filter": {
  "type": "selector",
  "dimension": "country",
  "value": "United States"
}
```

