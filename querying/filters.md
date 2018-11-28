# 查询过滤器

过滤器是一个JSON对象，指示应在查询计算中包含哪些数据行。它本质上等同于SQL中的WHERE子句。德鲁伊支持以下类型的过滤器。

### 选择器过滤器

最简单的过滤器是选择器过滤器。选择器过滤器将使特定维度与特定值匹配。选择器过滤器可用作更复杂的过滤器布尔表达式的基础过滤器。

SELECTOR过滤器的语法如下：

```json
"filter": { "type": "selector", "dimension": <dimension_string>, "value": <dimension_value_string> }
```

这相当于`WHERE <dimension_string> = '<dimension_value_string>'`。

选择器过滤器支持使用提取功能，有关详细信息，请参阅[使用提取功能过滤](http://druid.io/docs/0.12.3/querying/filters.html#filtering-with-extraction-functions)。

### 列比较过滤器

列比较过滤器类似于选择器过滤器，而是将维度相互比较。例如：

```json
"filter": { "type": "columnComparison", "dimensions": [<dimension_a>, <dimension_b>] }
```

这相当于`WHERE <dimension_a> = <dimension_b>`。

`dimensions`是[DimensionSpecs](http://druid.io/docs/0.12.3/querying/dimensionspecs.html)列表，可以根据需要应用提取功能。

### 正则表达式过滤器

正则表达式过滤器类似于选择器过滤器，但使用正则表达式。它将指定的维度与给定的模式匹配。该模式可以是任何标准[Java正则表达式](http://docs.oracle.com/javase/6/docs/api/java/util/regex/Pattern.html)。

```json
"filter": { "type": "regex", "dimension": <dimension_string>, "pattern": <pattern_string> }
```

正则表达式过滤器支持使用提取函数，有关详细信息，请参阅使用提取函数[过滤](http://druid.io/docs/0.12.3/querying/filters.html#filtering-with-extraction-functions)。

### 逻辑表达式过滤器

#### 和

AND过滤器的语法如下：

```json
"filter": { "type": "and", "fields": [<filter>, <filter>, ...] }
```

字段中的过滤器可以是此页面上定义的任何其他过滤器。

#### 要么

OR过滤器的语法如下：

```json
"filter": { "type": "or", "fields": [<filter>, <filter>, ...] }
```

字段中的过滤器可以是此页面上定义的任何其他过滤器。

#### 不

NOT过滤器的语法如下：

```json
"filter": { "type": "not", "field": <filter> }
```

字段中指定的过滤器可以是此页面上定义的任何其他过滤器。

### JavaScript过滤器

JavaScript过滤器将维度与指定的JavaScript函数谓词进行匹配。过滤器匹配函数返回true的值。

该函数采用单个参数（维度值），并返回true或false。

```json
{
  "type" : "javascript",
  "dimension" : <dimension_string>,
  "function" : "function(value) { <...> }"
}
```

**示例** 以下内容匹配和`name`之间的维度的任何维度值`'bar'``'foo'`

```json
{
  "type" : "javascript",
  "dimension" : "name",
  "function" : "function(x) { return(x >= 'bar' && x <= 'foo') }"
}
```

JavaScript过滤器支持使用提取函数，有关详细信息，请参阅使用提取函数[过滤](http://druid.io/docs/0.12.3/querying/filters.html#filtering-with-extraction-functions)。

默认情况下禁用基于JavaScript的功能。有关使用德鲁伊JavaScript功能的[指南](http://druid.io/docs/0.12.3/development/javascript.html)，请参阅德鲁伊[JavaScript编程指南](http://druid.io/docs/0.12.3/development/javascript.html)，包括如何启用它的说明。

### 提取过滤器

提取过滤器现已弃用。指定了提取功能的选择器过滤器提供相同的功能，应该使用它。

提取过滤器使用某些特定的[提取功能](http://druid.io/docs/0.12.3/querying/dimensionspecs.html#extraction-functions)匹配维度。以下过滤的量，提取函数具有变换条目中的值相匹配`input_key=output_value`，其中 `output_value`等于滤波器`value`和`input_key`呈现为尺寸。

**示例** 以下内容匹配`[product_1, product_3, product_5]`列中的维值`product`

```json
{
    "filter": {
        "type": "extraction",
        "dimension": "product",
        "value": "bar_1",
        "extractionFn": {
            "type": "lookup",
            "lookup": {
                "type": "map",
                "map": {
                    "product_1": "bar_1",
                    "product_5": "bar_1",
                    "product_3": "bar_1"
                }
            }
        }
    }
}
```

### 搜索过滤器

搜索过滤器可用于过滤部分字符串匹配。

```json
{
    "filter": {
        "type": "search",
        "dimension": "product",
        "query": {
          "type": "insensitive_contains",
          "value": "foo" 
        }        
    }
}
```

| 属性         | 描述                                                         | 需要？ |
| ------------ | ------------------------------------------------------------ | ------ |
| 类型         | 此字符串应始终为“搜索”。                                     | 是     |
| 尺寸         | 执行搜索的维度。                                             | 是     |
| 询问         | 搜索类型的JSON对象。请参阅下面的详细信息。                   | 是     |
| extractionFn | [提取功能](http://druid.io/docs/0.12.3/querying/filters.html#filtering-with-extraction-functions)以应用于维度 | 没有   |

搜索过滤器支持使用提取功能，有关详细信息，请参阅[使用提取功能过滤](http://druid.io/docs/0.12.3/querying/filters.html#filtering-with-extraction-functions)。

#### 搜索查询规范

##### 包含

| 属性       | 描述                                     | 需要？         |
| ---------- | ---------------------------------------- | -------------- |
| 类型       | 此字符串应始终为“包含”。                 | 是             |
| 值         | 用于运行搜索的String值。                 | 是             |
| 区分大小写 | 是否应将两个字符串作为区分大小写进行比较 | 不（默认==假） |

##### 不敏感的包含

| 属性 | 描述                                     | 需要？ |
| ---- | ---------------------------------------- | ------ |
| 类型 | 此String应始终为“insensitive_contains”。 | 是     |
| 值   | 用于运行搜索的String值。                 | 是     |

请注意，“insensitive_contains”搜索等同于“包含”搜索“caseSensitive”：false（或未提供）。

##### 分段

| 属性       | 描述                                                         | 需要？ |
| ---------- | ------------------------------------------------------------ | ------ |
| 类型       | 此字符串应始终为“片段”。                                     | 是     |
| 值         | 用于运行搜索的JSON数组String值。                             | 是     |
| 区分大小写 | 是否应将字符串作为区分大小写进行比较。默认值：false（不敏感） | 没有   |

### 在过滤器中

在filter中可以用来表达以下SQL查询：

```sql
 SELECT COUNT(*) AS 'Count' FROM `table` WHERE `outlaw` IN ('Good', 'Bad', 'Ugly')
```

IN过滤器的语法如下：

```json
{
    "type": "in",
    "dimension": "outlaw",
    "values": ["Good", "Bad", "Ugly"]
}
```

IN过滤器支持使用提取功能，有关详细信息，请参阅[使用提取功能过滤](http://druid.io/docs/0.12.3/querying/filters.html#filtering-with-extraction-functions)。

### 像过滤器

类似过滤器可用于基本通配符搜索。它们等同于SQL LIKE运算符。支持的特殊字符是“％”（匹配任意数量的字符）和“_”（匹配任何一个字符）。

| 属性         | 类型                                                         | 描述                              | 需要？ |
| ------------ | ------------------------------------------------------------ | --------------------------------- | ------ |
| 类型         | 串                                                           | 这应该总是“喜欢”。                | 是     |
| 尺寸         | 串                                                           | 要过滤的维度                      | 是     |
| 图案         | 串                                                           | LIKE模式，例如“foo％”或“___bar”。 | 是     |
| 逃逸         | 串                                                           | 可用于转义特殊字符的转义字符。    | 没有   |
| extractionFn | [提取功能](http://druid.io/docs/0.12.3/querying/filters.html#filtering-with-extraction-functions) | 提取功能以应用于维度              | 没有   |

类似过滤器支持使用提取函数，有关详细信息，请参阅使用提取函数[过滤](http://druid.io/docs/0.12.3/querying/filters.html#filtering-with-extraction-functions)。

此Like过滤器表示条件`last_name LIKE "D%"`（即last_name以“D”开头）。

```json
{
    "type": "like",
    "dimension": "last_name",
    "pattern": "D%"
}
```

### 绑定过滤器

绑定过滤器可用于过滤维度值的范围。它可用于比较滤波，如大于，小于，大于或等于，小于或等于，和“之间”（如果设置“低”和“上”）。

| 属性         | 类型                                                         | 描述                                                         | 需要？                    |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------- |
| 类型         | 串                                                           | 这应该总是“绑定”。                                           | 是                        |
| 尺寸         | 串                                                           | 要过滤的维度                                                 | 是                        |
| 降低         | 串                                                           | 过滤器的下限                                                 | 没有                      |
| 上           | 串                                                           | 过滤器的上限                                                 | 没有                      |
| lowerStrict  | 布尔                                                         | 对下限执行严格比较（“<”而不是“<=”）                          | 不，默认：false           |
| upperStrict  | 布尔                                                         | 对上限执行严格比较（“>”而不是“> =”）                         | 不，默认：false           |
| 排序         | 串                                                           | 指定在将值与边界进行比较时要使用的排序顺序。可以是以下值之一：“lexicographic”，“alphanumeric”，“numeric”，“strlen”。有关详细信息，请参阅[排序顺序](http://druid.io/docs/0.12.3/querying/sorting-orders.html)。 | 不，默认：“lexicographic” |
| extractionFn | [提取功能](http://druid.io/docs/0.12.3/querying/filters.html#filtering-with-extraction-functions) | 提取功能以应用于维度                                         | 没有                      |

绑定过滤器支持使用提取函数，有关详细信息，请参阅使用提取函数[过滤](http://druid.io/docs/0.12.3/querying/filters.html#filtering-with-extraction-functions)。

以下绑定过滤器表示条件`21 <= age <= 31`： `json { "type": "bound", "dimension": "age", "lower": "21", "upper": "31" , "ordering": "numeric" }`

此过滤器`foo <= name <= hoo`使用默认的词典排序顺序表示条件。 `json { "type": "bound", "dimension": "name", "lower": "foo", "upper": "hoo" }`

使用严格边界，此过滤器表示条件 `21 < age < 31` `json { "type": "bound", "dimension": "age", "lower": "21", "lowerStrict": true, "upper": "31" , "upperStrict": true, "ordering": "numeric" }`

用户还可以通过省略“上”或“下”来指定单侧界限。此过滤器表示`age < 31`。 `json { "type": "bound", "dimension": "age", "upper": "31" , "upperStrict": true, "ordering": "numeric" }`

同样，此过滤器表示 `age >= 18` `json { "type": "bound", "dimension": "age", "lower": "18" , "ordering": "numeric" }`

### 间隔过滤器

间隔过滤器对包含长毫秒值的列启用范围过滤，边界指定为ISO 8601时间间隔。它适用于`__time`列，长度量标准列和维度，其值可以解析为长毫秒。

此过滤器将ISO 8601间隔转换为长毫秒开始/结束范围，并转换为这些毫秒范围内的过滤器的OR，并进行数值比较。Bound过滤器将具有左闭和右开匹配（即start <= time <end）。

| 属性         | 类型                                                         | 描述                                                         | 需要？ |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------ |
| 类型         | 串                                                           | 这应该始终是“间隔”。                                         | 是     |
| 尺寸         | 串                                                           | 要过滤的维度                                                 | 是     |
| 间隔         | 排列                                                         | 包含ISO-8601间隔字符串的JSON数组。这定义了要过滤的时间范围。 | 是     |
| extractionFn | [提取功能](http://druid.io/docs/0.12.3/querying/filters.html#filtering-with-extraction-functions) | 提取功能以应用于维度                                         | 没有   |

间隔过滤器支持使用提取功能，有关详细信息，请参阅[使用提取功能过滤](http://druid.io/docs/0.12.3/querying/filters.html#filtering-with-extraction-functions)。

如果对此过滤器使用提取函数，则提取函数应输出可解析为长毫秒的值。

以下示例过滤了2014年10月1日至7日和2014年11月15日至16日的时间范围。 `json { "type" : "interval", "dimension" : "__time", "intervals" : [ "2014-10-01T00:00:00.000Z/2014-10-07T00:00:00.000Z", "2014-11-15T00:00:00.000Z/2014-11-16T00:00:00.000Z" ] }`

上面的过滤器相当于Bound过滤器的以下OR：

```json
{
    "type": "or",
    "fields": [
      {
        "type": "bound",
        "dimension": "__time",
        "lower": "1412121600000",
        "lowerStrict": false,
        "upper": "1412640000000" ,
        "upperStrict": true,
        "ordering": "numeric"
      },
      {
         "type": "bound",
         "dimension": "__time",
         "lower": "1416009600000",
         "lowerStrict": false,
         "upper": "1416096000000" ,
         "upperStrict": true,
         "ordering": "numeric"
      }
    ]
}
```

### 使用提取函数进行过滤

除“空间”过滤器之外的所有过滤器都支持提取功能。通过在过滤器上设置“extractionFn”字段来定义提取功能。有关[提取功能](http://druid.io/docs/0.12.3/querying/dimensionspecs.html#extraction-functions)的更多详细信息，请参见[提取功](http://druid.io/docs/0.12.3/querying/dimensionspecs.html#extraction-functions)

如果指定，则在应用过滤器之前，将使用提取函数来变换输入值。下面的示例显示了一个与提取功能相结合的选择器过滤器。此过滤器将根据查找映射中定义的值转换输入值; 然后，转换后的值将与字符串“bar_1”匹配。

**示例** 以下内容匹配`[product_1, product_3, product_5]`列中的维值`product`

```json
{
    "filter": {
        "type": "selector",
        "dimension": "product",
        "value": "bar_1",
        "extractionFn": {
            "type": "lookup",
            "lookup": {
                "type": "map",
                "map": {
                    "product_1": "bar_1",
                    "product_5": "bar_1",
                    "product_3": "bar_1"
                }
            }
        }
    }
}
```

## 列类型

Druid支持对timestamp，string，long和float列进行过滤。

请注意，只有字符串列具有位图索引。因此，筛选其他列类型的查询将需要扫描这些列。

### 过滤数字列

过滤数字列时，您可以像过滤字符串一样编写过滤器。在大多数情况下，您的过滤器将转换为数字谓词，并将直接应用于数字列值。在某些情况下（例如“正则表达式”过滤器），数字列值将在扫描期间转换为字符串。

例如，过滤特定值，`myFloatColumn = 10.1`：

```json
"filter": {
  "type": "selector",
  "dimension": "myFloatColumn",
  "value": "10.1"
}
```

过滤一系列值，`10 <= myFloatColumn < 20`：

```json
"filter": {
  "type": "bound",
  "dimension": "myFloatColumn",
  "ordering": "numeric",
  "lowerBound": "10",
  "lowerStrict": false,
  "upperBound": "20",
  "upperStrict": true
}
```

### 过滤时间戳列

查询过滤器也可以应用于时间戳列。timestamp列具有长毫秒值。要引用timestamp列，请使用字符串`__time`作为维度名称。与数字维度一样，应指定时间戳过滤器，就好像时间戳值是字符串一样。

如果用户希望使用特定格式，时区或区域设置来解释时间戳，则[时间格式提取功能](http://druid.io/docs/0.12.3/querying/dimensionspecs.html#time-format-extraction-function)非常有用。

例如，过滤长时间戳值：

```json
"filter": {
  "type": "selector",
  "dimension": "__time",
  "value": "124457387532"
}
```

在星期几过滤：

```json
"filter": {
  "type": "selector",
  "dimension": "__time",
  "value": "Friday",
  "extractionFn": {
    "type": "timeFormat",
    "format": "EEEE",
    "timeZone": "America/New_York",
    "locale": "en"
  }
}
```

过滤一组ISO 8601间隔：

```json

{
    "type" : "interval",
    "dimension" : "__time",
    "intervals" : [
      "2014-10-01T00:00:00.000Z/2014-10-07T00:00:00.000Z",
      "2014-11-15T00:00:00.000Z/2014-11-16T00:00:00.000Z"
    ]
}
```