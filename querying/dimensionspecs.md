# 转换维度值

可以在查询中使用以下JSON字段来操作维值。

## DimensionSpec

`DimensionSpec`■定义在聚合之前如何转换维度值。

### 默认尺寸标准

按原样返回维值，并可选择重命名维。

```json
{
  "type" : "default",
  "dimension" : <dimension>,
  "outputName": <output_name>,
  "outputType": <"STRING"|"LONG"|"FLOAT">
}
```

在数字列上指定DimensionSpec时，用户应在`outputType`字段中包含列的类型。如果未指定，则`outputType`默认为STRING。

有关更多详细信息，请参阅[输出类型](http://druid.io/docs/0.12.3/querying/dimensionspecs.html#output-types)部分。

### 提取尺寸规格

返回使用给定[提取函数](http://druid.io/docs/0.12.3/querying/dimensionspecs.html#extraction-functions)转换的维值。

```json
{
  "type" : "extraction",
  "dimension" : <dimension>,
  "outputName" :  <output_name>,
  "outputType": <"STRING"|"LONG"|"FLOAT">,
  "extractionFn" : <extraction_function>
}
```

`outputType`也可以在ExtractionDimensionSpec中指定，以便在合并之前将类型转换应用于结果。如果未指定，则`outputType`默认为STRING。

有关更多详细信息，请参阅[输出类型](http://druid.io/docs/0.12.3/querying/dimensionspecs.html#output-types)部分。

### 过滤的DimensionSpecs

这些仅适用于多值维度。如果您在德鲁伊中有一行具有值为[“v1”，“v2”，“v3”]的多值维度，并且您通过该维度发送groupBy / topN查询分组，并使用值“v1”的[查询过滤器](http://druid.io/docs/0.12.3/querying/filters.html)。在响应中，您将获得包含“v1”，“v2”和“v3”的3行。对于某些用例，此行为可能不直观。

这是因为“查询过滤器”在位图内部使用，仅用于匹配要包含在查询结果处理中的行。对于多值维度，“查询过滤器”的行为类似于包含检查，它将使行与维值[“v1”，“v2”，“v3”]匹配。有关详细信息，请参阅[细分中](http://druid.io/docs/0.12.3/design/segments.html) “多值列” [部分](http://druid.io/docs/0.12.3/design/segments.html)。然后groupBy / topN处理管道“爆炸”所有多值维度，每行产生3行“v1”，“v2”和“v3”。

除了有效选择要处理的行的“查询过滤器”之外，您还可以使用过滤的维度规范来过滤多值维度的值中的特定值。这些dimensionSpecs采用委托DimensionSpec和过滤条件。从“爆炸”行中，在查询结果中仅返回与给定过滤条件匹配的行。

以下过滤的维度规范根据“isWhitelist”属性值充当值的白名单或黑名单。

```json
{ "type" : "listFiltered", "delegate" : <dimensionSpec>, "values": <array of strings>, "isWhitelist": <optional attribute for true/false, default is true> }
```

以下过滤后的维度规范仅保留与正则表达式匹配的值。请注意，`listFiltered`比这更快，并且应该将其用于白名单或黑名单用例。

```json
{ "type" : "regexFiltered", "delegate" : <dimensionSpec>, "pattern": <java regex pattern> }
```

有关更多详细信息和示例，请参阅[多值维度](http://druid.io/docs/0.12.3/querying/multi-value-dimensions.html)。

### 查找DimensionSpecs

查找是一个[实验性](http://druid.io/docs/0.12.3/development/experimental.html)功能。

Lookup DimensionSpecs可用于直接将查找实现定义为维度规范。一般来说，有两种不同类型的查找实现。第一种类型在查询时传递，如`map`实现。

```json
{
  "type":"lookup",
  "dimension":"dimensionName",
  "outputName":"dimensionOutputName",
  "replaceMissingValueWith":"missing_value",
  "retainMissingValue":false,
  "lookup":{"type": "map", "map":{"key":"value"}, "isOneToOne":false}
}
```

的性能`retainMissingValue`和`replaceMissingValueWith`可以在查询时可以指定暗示如何处理缺失值。设置`replaceMissingValueWith`为`""`与设置`null`或省略属性具有相同的效果。`retainMissingValue`如果在查找中找不到维度，则设置为true将使用维度的原始值。默认值为，`replaceMissingValueWith = null`并且`retainMissingValue = false`导致缺失值被视为缺失。

设置`retainMissingValue = true`并指定a 是非法的`replaceMissingValueWith`。

`optimize`可以提供属性以允许优化基于查找的提取过滤器（默认情况下`optimize = true`）。

由于其大小而无法在查询时传递的第二种类型将基于已经通过配置文件或/和协调器注册的外部查找表或资源。

```json
{
  "type":"lookup",
  "dimension":"dimensionName",
  "outputName":"dimensionOutputName",
  "name":"lookupName"
}
```

## 输出类型

维度规范提供了一个选项，用于指定列值的输出类型。这是必要的，因为具有给定名称的列可能在不同的段中具有不同的值类型;结果将转换为`outputType`合并前指定的类型。

请注意，并非DimensionSpec目前都支持所有用例`outputType`，下表显示了哪些用例支持此选项：

| 查询类型      | 支持的？ |
| ------------- | -------- |
| GroupBy（v1） | 没有     |
| GroupBy（v2） | 是       |
| TOPN          | 是       |
| 搜索          | 没有     |
| 选择          | 没有     |
| 基数聚合器    | 没有     |

## 提取功能

提取函数定义应用于每个维值的转换。

转换可以应用于常规（字符串）维度以及特殊`__time`维度，它根据查询[聚合粒度](http://druid.io/docs/0.12.3/querying/granularities.html)表示当前时间桶。

**注意**：对于采用字符串值的函数（例如正则表达式）， 在传递给提取函数之前， `__time`维度值将以[ISO-8601格式](https://en.wikipedia.org/wiki/ISO_8601)进行格式化。

### 正则表达式提取函数

返回给定正则表达式的第一个匹配组。如果没有匹配，则返回维度值。

```json
{
  "type" : "regex",
  "expr" : <regular_expression>,
  "index" : <group to extract, default 1>
  "replaceMissingValue" : true,
  "replaceMissingValueWith" : "foobar"
}
```

例如，使用`"expr" : "(\\w\\w\\w).*"`将改变 `'Monday'`，`'Tuesday'`，`'Wednesday'`成`'Mon'`，`'Tue'`，`'Wed'`。

如果设置了“index”，它将控制从匹配中提取哪个组。索引零提取与整个模式匹配的字符串。

如果`replaceMissingValue`属性为true，则提取函数会将与正则表达式模式不匹配的维值转换为用户指定的String。默认值是`false`。

该`replaceMissingValueWith`属性设置将替换不匹配维度值的String，如果`replaceMissingValue`为true。如果`replaceMissingValueWith`未指定，则不匹配的维值将替换为空值。

例如，如果`expr`是`"(a\w+)"`在例如上述JSON，匹配单词开始以字母a的正则表达式`a`，提取功能将转换像尺寸值`banana`到`foobar`。

### 部分提取功能

如果正则表达式匹配，则返回维度值，否则返回null。

```json
{ "type" : "partial", "expr" : <regular_expression> }
```

### 搜索查询提取功能

如果给定[`SearchQuerySpec`](http://druid.io/docs/0.12.3/querying/searchqueryspec.html) 匹配，则返回维度值，否则返回null。

```json
{ "type" : "searchQuery", "query" : <search_query_spec> }
```

### 子串提取功能

返回从提供的索引和所需长度开始的维值的子字符串。索引和长度都以字符串中存在的Unicode代码单元的数量来度量，就像它以UTF-16编码一样。请注意，某些Unicode字符可能由两个代码单元表示。这与Java String类的“substring”方法的行为相同。

如果所需长度超过维度值的长度，则将返回从索引处开始的字符串的剩余部分。如果index大于维值的长度，则返回null。

```json
{ "type" : "substring", "index" : 1, "length" : 4 }
```

子字符串可以省略长度以从index开始返回维值的其余部分，如果index大于维值的长度，则可以为null。

```json
{ "type" : "substring", "index" : 3 }
```

### Strlen提取功能

返回维度值的长度，以字符串中存在的Unicode代码单元数量度量，就像它以UTF-16编码一样。请注意，某些Unicode字符可能由两个代码单元表示。这与Java String类的“length”方法的行为相同。

空字符串被视为长度为零。

```json
{ "type" : "strlen" }
```

### 时间格式提取功能

返回根据给定格式字符串，时区和语言环境格式化的维值。

对于`__time`维值，这将格式化[聚合粒度所](http://druid.io/docs/0.12.3/querying/granularities.html)分配的时间值

对于常规维度，它假定字符串的格式为 [ISO-8601日期和时间格式](https://en.wikipedia.org/wiki/ISO_8601)。

- `format`：结果维值的日期时间格式，在[Joda Time DateTimeFormat中](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)，或null以使用默认的ISO8601格式。
- `locale`：语言环境（语言和国家）使用，给出作为[IETF BCP 47语言标记](http://www.oracle.com/technetwork/java/javase/java8locales-2095355.html#util-text)，例如`en-US`，`en-GB`，`fr-FR`，`fr-CA`，等。
- `timeZone`：在[IANA tz数据库格式中](http://en.wikipedia.org/wiki/List_of_tz_database_time_zones)使用的时区，例如`Europe/Berlin`（这可能与聚合时区不同）
- `granularity`：[粒度](http://druid.io/docs/0.12.3/querying/granularities.html)在格式化之前使用，或省略不应用任何粒度。
- `asMillis`：boolean value，设置为true以将输入字符串视为millis而不是ISO8601字符串。此外，如果`format`为null或未指定，则输出将以毫秒而不是ISO8601。

```json
{ "type" : "timeFormat",
  "format" : <output_format> (optional),
  "timeZone" : <time_zone> (optional, default UTC),
  "locale" : <locale> (optional, default current locale),
  "granularity" : <granularity> (optional, default none) },
  "asMillis" : <true or false> (optional) }
```

例如，以下维度规范以法语返回蒙特利尔的星期几：

```json
{
  "type" : "extraction",
  "dimension" : "__time",
  "outputName" :  "dayOfWeek",
  "extractionFn" : {
    "type" : "timeFormat",
    "format" : "EEEE",
    "timeZone" : "America/Montreal",
    "locale" : "fr"
  }
}
```

### 时间分析提取功能

使用给定的输入格式将维度值解析为时间戳，并使用给定的输出格式返回格式。

请注意，如果您正在使用`__time`维度，则应考虑使用 [时间提取功能，而](http://druid.io/docs/0.12.3/querying/dimensionspecs.html#time-format-extraction-function)不是直接使用时间值而不是字符串值。

时间格式在[SimpleDateFormat文档](http://icu-project.org/apiref/icu4j/com/ibm/icu/text/SimpleDateFormat.html)中描述

```json
{ "type" : "time",
  "timeFormat" : <input_format>,
  "resultFormat" : <output_format> }
```

### Javascript提取功能

返回由给定JavaScript函数转换的维值。

对于常规维度，输入值将作为字符串传递。

对于`__time`维度，输入值作为表示自1970年1月1日UTC以来的毫秒数的数字传递。

常规维度的示例

```json
{
  "type" : "javascript",
  "function" : "function(str) { return str.substr(0, 3); }"
}
{
  "type" : "javascript",
  "function" : "function(str) { return str + '!!!'; }",
  "injective" : true
}
```

属性`injective`指定javascript函数是否保留唯一性。默认值`false`意味着不保留唯一性

`__time`尺寸示例：

```json
{
  "type" : "javascript",
  "function" : "function(t) { return 'Second ' + Math.floor((t % 60000) / 1000); }"
}
```

默认情况下禁用基于JavaScript的功能。有关使用德鲁伊JavaScript功能的[指南](http://druid.io/docs/0.12.3/development/javascript.html)，请参阅德鲁伊[JavaScript编程指南](http://druid.io/docs/0.12.3/development/javascript.html)，包括如何启用它的说明。

### 注册查找提取功能

查找是德鲁伊中的一个概念，其中维度值（可选）被替换为新值。有关使用查找的更多文档，请参阅[查找](http://druid.io/docs/0.12.3/querying/lookups.html)。“registeredLookup”提取功能允许您引用已在群集范围配置中注册的查找。

一个例子：

```json
{
  "type":"registeredLookup",
  "lookup":"some_lookup_name",
  "retainMissingValue":true
}
```

的性能`retainMissingValue`和`replaceMissingValueWith`可以在查询时可以指定暗示如何处理缺失值。设置`replaceMissingValueWith`为`""`与设置`null`或省略属性具有相同的效果。`retainMissingValue`如果在查找中找不到维度，则设置为true将使用维度的原始值。默认值为，`replaceMissingValueWith = null`并且`retainMissingValue = false`导致缺失值被视为缺失。

设置`retainMissingValue = true`并指定a 是非法的`replaceMissingValueWith`。

一个属性`injective`可以覆盖查找自己是否是单 [射的意义](http://druid.io/docs/0.12.3/querying/lookups.html#query-execution)。如果未指定，Druid将使用已注册的群集范围查找配置。

`optimize`可以提供属性以允许优化基于查找的提取过滤器（默认情况下`optimize = true`）。优化层将在代理上运行，它将重写提取过滤器作为选择器过滤器的子句。例如以下过滤器

```json
{
    "filter": {
        "type": "selector",
        "dimension": "product",
        "value": "bar_1",
        "extractionFn": {
            "type": "registeredLookup",
            "optimize": true,
            "lookup": "some_lookup_name"
        }
    }
}
```

将被重写为以下更简单的查询，假设查找将“product_1”和“product_3”映射到值“bar_1”：

```json
{
   "filter":{
      "type":"or",
      "fields":[
         {
            "filter":{
               "type":"selector",
               "dimension":"product",
               "value":"product_1"
            }
         },
         {
            "filter":{
               "type":"selector",
               "dimension":"product",
               "value":"product_3"
            }
         }
      ]
   }
}
```

通过将空字符串指定为查找文件中的键，可以将空维值映射到特定值。这允许区分空维度和导致null的查找。例如，`{"":"bar","bat":"baz"}`使用维值指定`[null, "foo", "bat"]`并替换缺失值`"oof"`将产生结果`["bar", "oof", "baz"]`。省略空字符串键将导致缺失值接管。例如，`{"bat":"baz"}`使用维值指定`[null, "foo", "bat"]`并替换缺失值`"oof"`将产生结果`["oof", "oof", "baz"]`。

### 内联查找提取功能

查找是德鲁伊中的一个概念，其中维度值（可选）被替换为新值。有关使用查找的更多文档，请参阅[查找](http://druid.io/docs/0.12.3/querying/lookups.html)。“查找”提取功能允许您指定内联查找映射，而无需在群集范围的配置中注册一个。

例子：

```json
{
  "type":"lookup",
  "lookup":{
    "type":"map",
    "map":{"foo":"bar", "baz":"bat"}
  },
  "retainMissingValue":true,
  "injective":true
}
{
  "type":"lookup",
  "lookup":{
    "type":"map",
    "map":{"foo":"bar", "baz":"bat"}
  },
  "retainMissingValue":false,
  "injective":false,
  "replaceMissingValueWith":"MISSING"
}
```

内联查找应该是类型`map`。

属性`retainMissingValue`，`replaceMissingValueWith`，`injective`，和`optimize`类似的行为在 [注册查询提取功能](http://druid.io/docs/0.12.3/querying/dimensionspecs.html#registered-lookup-extraction-function)。

### 级联提取功能

提供提取功能的链式执行。

属性`extractionFns`包含任何提取函数的数组，以数组索引顺序执行。

链接[正则表达式提取函数](http://druid.io/docs/0.12.3/querying/dimensionspecs.html#regular-expression-extraction-function)，[javascript提取函数](http://druid.io/docs/0.12.3/querying/dimensionspecs.html#javascript-extraction-function)和[子字符串提取函数的](http://druid.io/docs/0.12.3/querying/dimensionspecs.html#substring-extraction-function)示例如下。

```json
{
  "type" : "cascade", 
  "extractionFns": [
    { 
      "type" : "regex", 
      "expr" : "/([^/]+)/", 
      "replaceMissingValue": false,
      "replaceMissingValueWith": null
    },
    { 
      "type" : "javascript", 
      "function" : "function(str) { return \"the \".concat(str) }" 
    },
    { 
      "type" : "substring", 
      "index" : 0, "length" : 7 
    }
  ]
}
```

它将按指定的顺序使用指定的提取函数转换维值。例如，`'/druid/prod/historical'`转换`'the dru'`为正则表达式提取函数首先将其转换为`'druid'`然后，javascript提取函数将其转换为`'the druid'`，最后，子字符串提取函数将其转换为`'the dru'`。

### 字符串格式提取功能

返回根据给定格式字符串格式化的维值。

```json
{ "type" : "stringFormat", "format" : <sprintf_expression>, "nullHandling" : <optional attribute for handling null value> }
```

例如，如果要在实际维度值之前和之后连接“[”和“]”，则需要将“[％s]”指定为格式字符串。“nullHandling”可以是一个`nullString`，`emptyString`或`returnNull`。用“[％s]的”格式中，每个配置将导致`[null]`，`[]`，`null`。默认是`nullString`。

### 上部和下部提取功能。

以全大写或小写形式返回维值。可选地，用户可以指定要使用的语言以执行上变换或下变换

```json
{
  "type" : "upper",
  "locale":"fr"
}
```

或者不设置“locale”（在这种情况下，是此Java虚拟机实例的默认语言环境的当前值。）

```json
{
  "type" : "lower"
}
```

### 铲斗提取功能

桶提取功能用于通过将它们转换为相同的基值来在给定大小的每个范围中存储数值。非数字值将转换为null。

- `size` ：桶的大小（可选，默认为1）
- `offset` ：桶的偏移量（可选，默认为0）

以下提取函数从2开始创建5个桶。在这种情况下，[2,7]范围内的值将转换为2，[7,12]中的值将转换为7，等等。

```json
{
  "type" : "bucket",
  "size" : 5,
  "offset" : 2
}
```