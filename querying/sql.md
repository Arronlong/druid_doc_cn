# SQL

内置SQL是一项[实验性](http://druid.io/docs/0.12.3/development/experimental.html)功能。此处描述的API可能会发生变化。

Druid SQL是一个内置的SQL层，是Druid原生基于JSON的查询语言的替代品，由基于[Apache Calcite](https://calcite.apache.org/)的解析器和规划器提供支持。Druid SQL将SQL转换为查询代理（您查询的第一个节点）上的本机Druid查询，然后将其作为本机Druid查询传递给数据节点。除了在代理上转换SQL的（轻微）开销之外，与本机查询相比，没有额外的性能损失。

要启用Druid SQL，请确保已`druid.sql.enable = true`在common.runtime.properties或代理的runtime.properties中设置。

## 查询语法

每个德鲁伊数据源都显示为“德鲁伊”模式中的表格。这也是默认模式，因此可以将德鲁伊数据源引用为`druid.dataSourceName`或者简单引用`dataSourceName`。

可以选择使用双引号引用数据源和列名等标识符。要在标识符中转义双引号，请使用另一个双引号，例如`"My ""very own"" identifier"`。所有标识符都区分大小写，并且不执行隐式大小写转换。

文字字符串应引用单引号，如`'foo'`。具有Unicode转义的文字字符串可以写成`U&'fo\00F6'`，其中十六进制的字符代码以反斜杠为前缀。文字数字可以用`100`（表示整数），`100.0`（表示浮点值）或`1.0e5`（科学记数法）等形式编写。文字时间戳可以写成`TIMESTAMP '2000-01-01 00:00:00'`。文字间隔，用于时间算法，可以这样写`INTERVAL '1' HOUR`，`INTERVAL '1 02:03' DAY TO MINUTE`，`INTERVAL '1-2' YEAR TO MONTH`，等等。

Druid SQL支持具有以下结构的SELECT查询：

```text
[ EXPLAIN PLAN FOR ]
[ WITH tableName [ ( column1, column2, ... ) ] AS ( query ) ]
SELECT [ ALL | DISTINCT ] { * | exprs }
FROM table
[ WHERE expr ]
[ GROUP BY exprs ]
[ HAVING expr ]
[ ORDER BY expr [ ASC | DESC ], expr [ ASC | DESC ], ... ]
[ LIMIT limit ]
```

到任何一个数据源德，像FROM子句是指`druid.foo`，一个[INFORMATION_SCHEMA表](http://druid.io/docs/0.12.3/querying/sql.html#retrieving-metadata)，子查询，或共表表达在所提供的WITH子句。如果FROM子句引用子查询或common-table-expression，并且两个查询级别都是聚合，并且它们无法组合到单个聚合级别，则整个查询将作为[嵌套GroupBy](http://druid.io/docs/0.12.3/querying/groupbyquery.html#nested-groupbys)执行。

WHERE子句引用FROM表中的列，并将转换[为本机过滤器](http://druid.io/docs/0.12.3/querying/filters.html)。WHERE子句也可以引用子查询，例如`WHERE col1 IN (SELECT foo FROM ...)`。像这样的查询作为[半连接](http://druid.io/docs/0.12.3/querying/sql.html#query-execution)执行，如下所述。

GROUP BY子句引用FROM表中的列。使用GROUP BY，DISTINCT或任何聚合函数将使用Druid的[三种本机聚合查询类型](http://druid.io/docs/0.12.3/querying/sql.html#query-execution)之一触发[聚合查询](http://druid.io/docs/0.12.3/querying/sql.html#query-execution)。GROUP BY可以引用表达式或select子句序号位置（比如`GROUP BY 2`按第二个选定列分组）。

HAVING子句引用执行GROUP BY后出现的列。它可用于过滤分组表达式或聚合值。它只能与GROUP BY一起使用。

ORDER BY子句引用执行GROUP BY后出现的列。它可用于根据分组表达式或聚合值对结果进行排序。ORDER BY可以引用表达式或select子句序号位置（比如`ORDER BY 2`按第二个选定列排序）。对于非聚合查询，ORDER BY只能按`__time`列排序。对于聚合查询，ORDER BY可以按任何列排序。

LIMIT子句可用于限制返回的行数。它可以与任何查询类型一起使用。对于使用本机TopN查询类型而不是本机GroupBy查询类型运行的查询，它会下推到数据节点。未来版本的Druid也将支持使用本机GroupBy查询类型下推限制。如果您注意到添加限制并未对性能产生太大影响，那么德鲁伊可能不会降低您的查询限制。

将“EXPLAIN PLAN FOR”添加到任何查询的开头，以查看它将如何作为本机Druid查询运行。在这种情况下，查询实际上不会被执行。

### 聚合功能

聚合函数可以出现在任何查询的SELECT子句中。可以使用类似语法过滤任何聚合器 `AGG(expr) FILTER(WHERE whereExpr)`。过滤的聚合器仅聚合与其过滤器匹配的行。同一SQL查询中的两个聚合器可能具有不同的筛选器。

只有COUNT聚合可以接受DISTINCT。

| 功能                                               | 笔记                                                         |
| -------------------------------------------------- | ------------------------------------------------------------ |
| `COUNT(*)`                                         | 计算行数。                                                   |
| `COUNT(DISTINCT expr)`                             | 计算expr的不同值，可以是string，numeric或hyperUnique。默认情况下，这是近似值，使用[HyperLogLog](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf)的变体。要获得准确的计数，请将“useApproximateCountDistinct”设置为“false”。如果这样做，expr必须是字符串或数字，因为使用hyperUnique列无法进行精确计数。另见`APPROX_COUNT_DISTINCT(expr)`。在精确模式下，每个查询只允许一个不同的计数。 |
| `SUM(expr)`                                        | 总和数。                                                     |
| `MIN(expr)`                                        | 采用最少的数字。                                             |
| `MAX(expr)`                                        | 取最大数字。                                                 |
| `AVG(expr)`                                        | 平均数。                                                     |
| `APPROX_COUNT_DISTINCT(expr)`                      | 计算expr的不同值，可以是常规列或hyperUnique列。无论“useApproximateCountDistinct”的值如何，这始终是近似值。另见`COUNT(DISTINCT expr)`。 |
| `APPROX_QUANTILE(expr, probability, [resolution])` | 计算numeric或approxHistogram exprs上的近似分位数。“概率”应该在0和1之间（不包括）。“分辨率”是用于计算的质心数。分辨率越高，结果越精确，但开销也越大。如果未提供，则默认分辨率为50. 必须加载[近似直方图扩展](http://druid.io/docs/0.12.3/development/extensions-core/approximate-histograms.html)才能使用此功能。 |

### 数字函数

数字函数将返回64位整数或64位浮点数，具体取决于它们的输入。

| 功能                       | 笔记                                                         |
| -------------------------- | ------------------------------------------------------------ |
| `ABS(expr)`                | 绝对值。                                                     |
| `CEIL(expr)`               | 天花板。                                                     |
| `EXP(expr)`                | e到expr的力量。                                              |
| `FLOOR(expr)`              | 地板。                                                       |
| `LN(expr)`                 | 对数（基数e）。                                              |
| `LOG10(expr)`              | 对数（基数10）。                                             |
| `POWER(expr, power)`       | expr to a power。                                            |
| `SQRT(expr)`               | 平方根。                                                     |
| `TRUNCATE(expr[, digits])` | 将expr截断为特定的小数位数。如果数字为负数，则会截断小数点左侧的许多位置。如果未指定，数字默认为零。 |
| `TRUNC(expr[, digits])`    | 同义词`TRUNCATE`。                                           |
| `x + y`                    | 加成。                                                       |
| `x - y`                    | 减法。                                                       |
| `x * y`                    | 乘法。                                                       |
| `x / y`                    | 师。                                                         |
| `MOD(x, y)`                | 模数（x的余数除以y）。                                       |

### 字符串函数

字符串函数接受字符串，并返回适合该函数的类型。

| 功能                                     | 笔记                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| `x \                                     | \                                                            |
| `LENGTH(expr)`                           | 以UTF-16代码单位表示的expr长度。                             |
| `CHAR_LENGTH(expr)`                      | 同义词`LENGTH`。                                             |
| `CHARACTER_LENGTH(expr)`                 | 同义词`LENGTH`。                                             |
| `STRLEN(expr)`                           | 同义词`LENGTH`。                                             |
| `LOOKUP(expr, lookupName)`               | 在已注册的[查询时查找表中查找](http://druid.io/docs/0.12.3/querying/lookups.html) expr 。 |
| `LOWER(expr)`                            | 全部以小写形式返回expr。                                     |
| `REGEXP_EXTRACT(expr, pattern, [index])` | 应用正则表达式模式并提取捕获组，如果没有匹配则为null。如果index未指定或为零，则返回与模式匹配的子字符串。 |
| `REPLACE(expr, pattern, replacement)`    | 用expr中的替换替换模式，并返回结果。                         |
| `STRPOS(haystack, needle)`               | 返回haystack中针的索引，从1开始。如果未找到针，则返回0。     |
| `SUBSTRING(expr, index, [length])`       | 返回expr的子字符串，从index开始，具有最大长度，均以UTF-16代码单位测量。 |
| `SUBSTR(expr, index, [length])`          | SUBSTRING的同义词。                                          |
| `TRIM（[BOTH \                           | 领导 \                                                       |
| `BTRIM(expr[, chars])`                   | 替代形式`TRIM(BOTH <chars> FROM <expr>`）。                  |
| `LTRIM(expr[, chars])`                   | 替代形式`TRIM(LEADING <chars> FROM <expr>`）。               |
| `RTRIM(expr[, chars])`                   | 替代形式`TRIM(TRAILING <chars> FROM <expr>`）。              |
| `UPPER(expr)`                            | 以全部大写形式返回expr。                                     |

### 时间功能

时间函数可以与Druid `__time`列一起使用，任何列通过使用`MILLIS_TO_TIMESTAMP`函数存储毫秒时间戳，或者通过使用函数存储字符串时间戳的任何列`TIME_PARSE` 。默认情况下，时间操作使用UTC时区。您可以通过将连接上下文参数“sqlTimeZone”设置为另一个时区的名称（如“America / Los_Angeles”）或“-08：00”之类的偏移来更改时区。如果需要在同一查询中混合多个时区，或者如果需要使用连接时区以外的时区，则某些功能还会将时区作为参数接受。这些参数始终优先于连接时区。

| 功能                                                         | 笔记                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `CURRENT_TIMESTAMP`                                          | 连接时区中的当前时间戳。                                     |
| `CURRENT_DATE`                                               | 连接时区中的当前日期。                                       |
| `DATE_TRUNC(<unit>, <timestamp_expr>)`                       | 将时间戳缩小，将其作为新时间戳返回。单位可以是“毫秒”，“秒”，“分钟”，“小时”，“日”，“周”，“月”，“季度”，“年”，“十年”，“世纪”或“千禧年” ”。 |
| `TIME_FLOOR(<timestamp_expr>, <period>, [<origin>, [<timezone>]])` | 将时间戳缩小，将其作为新时间戳返回。期间可以是任何ISO8601期间，如P3M（季度）或PT12H（半天）。时区（如果提供）应为时区名称，如“America / Los_Angeles”或偏移量，如“-08：00”。此功能类似`FLOOR`但更灵活。 |
| `TIME_SHIFT(<timestamp_expr>, <period>, <step>, [<timezone>])` | 按时间段（步骤时间）移动时间戳，将其作为新时间戳返回。期间可以是任何ISO8601期间。步骤可能是否定的。时区（如果提供）应为时区名称，如“America / Los_Angeles”或偏移量，如“-08：00”。 |
| `TIME_EXTRACT(<timestamp_expr>, [<unit>, [<timezone>]])`     | 从expr中提取时间部分，将其作为数字返回。单位可以是EPOCH，SECOND，MINUTE，HOUR，DAY（日期），DOW（星期几），DOY（一年中的某一天），WEEK（星期一[周](https://en.wikipedia.org/wiki/ISO_week_date)），MONTH（1到12），QUARTER（1通过4），或年。时区（如果提供）应为时区名称，如“America / Los_Angeles”或偏移量，如“-08：00”。此功能类似`EXTRACT`但更灵活。单位和时区必须是文字，必须提供引用，如`TIME_EXTRACT(__time, 'HOUR')`或`TIME_EXTRACT(__time, 'HOUR', 'America/Los_Angeles')`。 |
| `TIME_PARSE(<string_expr>, [<pattern>, [<timezone>]])`       | 使用给定的[Joda DateTimeFormat模式](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)将字符串解析为时间戳，或者`2000-01-02T03:04:05Z`如果未提供[模式](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)，则使用ISO8601（例如）。时区（如果提供）应为时区名称，如“America / Los_Angeles”或偏移量，如“-08：00”，并将用作不包含时区偏移的字符串的时区。模式和时区必须是文字。无法解析为时间戳的字符串将返回NULL。 |
| `TIME_FORMAT(<timestamp_expr>, [<pattern>, [<timezone>]])`   | 将时间戳格式化为具有给定[Joda DateTimeFormat模式](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)的字符串，或者`2000-01-02T03:04:05Z`如果未提供[模式](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)，则将ISO8601（例如）格式化。时区（如果提供）应为时区名称，如“America / Los_Angeles”或偏移量，如“-08：00”。模式和时区必须是文字。 |
| `MILLIS_TO_TIMESTAMP(millis_expr)`                           | 将自纪元以来的毫秒数转换为时间戳。                           |
| `TIMESTAMP_TO_MILLIS(timestamp_expr)`                        | 将时间戳转换为自纪元以来的毫秒数。                           |
| `EXTRACT(<unit> FROM timestamp_expr)`                        | 从expr中提取时间部分，将其作为数字返回。单位可以是EPOCH，SECOND，MINUTE，HOUR，DAY（日期），DOW（星期几），DOY（一年中的某一天），WEEK（一年中的一周），MONTH，QUARTER或YEAR。单位必须不加引用，如`EXTRACT(HOUR FROM __time)`。 |
| `FLOOR(timestamp_expr TO <unit>)`                            | 将时间戳缩小，将其作为新时间戳返回。单位可以是秒，分钟，小时，天，周，月，季或年。 |
| `CEIL(timestamp_expr TO <unit>)`                             | 舍入时间戳，将其作为新时间戳返回。单位可以是秒，分钟，小时，天，周，月，季或年。 |
| `TIMESTAMPADD(<unit>, <count>, <timestamp>)`                 | 相当于`timestamp + count * INTERVAL '1' UNIT`。              |
| `timestamp_expr {+ \                                         | - } `                                                        |

### 比较运算符

| 功能                              | 笔记                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| `x = y`                           | 等于。                                                       |
| `x <> y`                          | 不，等于。                                                   |
| `x > y`                           | 比...更棒。                                                  |
| `x >= y`                          | 大于或等于。                                                 |
| `x < y`                           | 少于。                                                       |
| `x <= y`                          | 小于或等于。                                                 |
| `x BETWEEN y AND z`               | 相当于`x >= y AND x <= z`。                                  |
| `x NOT BETWEEN y AND z`           | 相当于`x < y OR x > z`。                                     |
| `x LIKE pattern [ESCAPE esc]`     | 如果x匹配SQL LIKE模式（带有可选的转义），则为true。          |
| `x NOT LIKE pattern [ESCAPE esc]` | 如果x与SQL LIKE模式（带有可选的转义）不匹配，则为True。      |
| `x IS NULL`                       | 如果x为NULL或空字符串，则为True。                            |
| `x IS NOT NULL`                   | 如果x既不是NULL也不是空字符串，则为真。                      |
| `x IS TRUE`                       | 如果x为真，则为真。                                          |
| `x IS NOT TRUE`                   | 如果x不为真，则为真。                                        |
| `x IS FALSE`                      | 如果x为假，则为真。                                          |
| `x IS NOT FALSE`                  | 如果x不为假，则为真。                                        |
| `x IN (values)`                   | 如果x是列出的值之一，则为真。                                |
| `x NOT IN (values)`               | 如果x不是列出的值之一，则为真。                              |
| `x IN (subquery)`                 | 如果子查询返回x，则返回true。有关Druid SQL如何处理的详细信息，请参阅上面的[语法和执行](http://druid.io/docs/0.12.3/querying/sql.html#syntax-and-execution)`IN (subquery)`。 |
| `x NOT IN (subquery)`             | 如果子查询未返回x，则为True。有关Druid SQL如何处理的详细信息，请参阅[语法和执行](http://druid.io/docs/0.12.3/querying/sql.html#syntax-and-execution)`IN (subquery)`。 |
| `x AND y`                         | 布尔AND。                                                    |
| `x OR y`                          | 布尔OR。                                                     |
| `NOT x`                           | 布尔NOT。                                                    |

### 其他功能

| 功能                                                         | 笔记                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `CAST(value AS TYPE)`                                        | 将值转换为其他类型。有关Druid SQL如何处理CAST的详细信息，请参阅[数据类型和强制类型转换](http://druid.io/docs/0.12.3/querying/sql.html#data-types-and-casts)。 |
| `CASE expr WHEN value1 THEN result1 \[ WHEN value2 THEN result2 ... \] \[ ELSE resultN \] END` | 简单的案例。                                                 |
| `CASE WHEN boolean_expr1 THEN result1 \[ WHEN boolean_expr2 THEN result2 ... \] \[ ELSE resultN \] END` | 搜索案例。                                                   |
| `NULLIF(value1, value2)`                                     | 如果value1和value2匹配则返回NULL，否则返回value1。           |
| `COALESCE(value1, value2, ...)`                              | 返回既不是NULL也不是空字符串的第一个值。                     |

### 不支持的功能

Druid不支持所有SQL功能，包括：

- OVER子句和分析函数，如`LAG`和`LEAD`。
- JOIN子句，除了如上所述的半连接之外。
- OFFSET条款。
- DDL和DML。

此外，SQL语言不支持某些Druid功能。一些不受支持的德鲁伊功能包括：

- [多值维度](http://druid.io/docs/0.12.3/querying/multi-value-dimensions.html)。
- [DataSketches聚合器](http://druid.io/docs/0.12.3/development/extensions-core/datasketches-aggregators.html)。

## 数据类型和强制转换

Druid本身支持五种基本列类型：“long”（64位符号int），“float”（32位浮点数），“double”（64位浮点数）“string”（UTF-8编码字符串）和“复杂” （对于更多异国情调的数据类型，如hyperUnique和approxHistogram列，可以全部捕获）。时间戳（包括`__time`列）存储为long，其值为自1970年1月1日UTC以来的毫秒数。

在运行时，德鲁伊可能会为某些运算符（如SUM聚合器）将32位浮点数扩展为64位。反过来不会发生：64位浮点数不会缩小到32位。

Druid通常可以互换地处理NULL和空字符串，而不是根据SQL标准。因此，Druid SQL只对NULL有部分支持。例如，表达式`col IS NULL`和`col = ''`是等价的，如果双方将评估为true `col`包含一个空字符串。同样，如果是空字符串，表达式`COALESCE(col1, col2)`将返回。当聚合器计算所有行时， 聚合器将计算expr既不为空也不为空字符串的行数。Druid中的字符串列是NULLable。数字列不是NULL; 如果查询Druid数据源的所有段中不存在的数字列，则对于来自这些段的行，它将被视为零。`col2``col1``COUNT(*)``COUNT(expr)`

对于数学运算，如果表达式中涉及的所有操作数都是整数，则Druid SQL将使用整数数学运算。否则，德鲁伊将切换到浮点数学。您可以通过将一个操作数强制转换为FLOAT来强制执行此操作。

下表描述了SQL类型在查询运行时期间如何映射到Druid类型。除了表中提到的异常之外，具有相同Druid运行时类型的两种SQL类型之间的转换将不起作用。两种具有不同Druid运行时类型的SQL类型之间的转换将在Druid中生成运行时强制转换。如果某个值无法正确转换为其他值`CAST('foo' AS BIGINT)`，则运行时将替换默认值。转换为非可空类型的NULL值也将替换为默认值（例如，转换为数字的空值将转换为零）。

| SQL类型   | 德鲁伊运行时类型 | 默认值                             | 笔记                                                         |
| --------- | ---------------- | ---------------------------------- | ------------------------------------------------------------ |
| CHAR      | 串               | `''`                               |                                                              |
| VARCHAR   | 串               | `''`                               | Druid STRING列报告为VARCHAR                                  |
| DECIMAL   | 双               | `0.0`                              | DECIMAL使用浮点数，而不是定点数学                            |
| 浮动      | 浮动             | `0.0`                              | 德鲁伊FLOAT柱报告为FLOAT                                     |
| 真实      | 双               | `0.0`                              |                                                              |
| 双        | 双               | `0.0`                              | Druid DOUBLE列报告为DOUBLE                                   |
| 布尔      | 长               | `false`                            |                                                              |
| TINYINT   | 长               | `0`                                |                                                              |
| SMALLINT  | 长               | `0`                                |                                                              |
| 整数      | 长               | `0`                                |                                                              |
| BIGINT    | 长               | `0`                                | 德鲁伊LONG列（除外`__time`）报告为BIGINT                     |
| TIMESTAMP | 长               | `0`，意思是1970-01-01 00:00:00 UTC | 德鲁伊的`__time`专栏报道为TIMESTAMP。字符串和时间戳类型之间的转换假定标准SQL格式，例如`2000-01-02 03:04:05`，*不是* ISO8601格式。要处理其他格式，请使用其中一个[时间函数](http://druid.io/docs/0.12.3/querying/sql.html#time-functions) |
| 日期      | 长               | `0`，意思是1970-01-01              | 将TIMESTAMP转换为DATE会将时间戳向下舍入到最近的一天。字符串和日期类型之间的转换假定标准SQL格式，例如`2000-01-02`。要处理其他格式，请使用其中一个[时间函数](http://druid.io/docs/0.12.3/querying/sql.html#time-functions) |
| 其他      | 复杂             | 没有                               | 可以代表各种德鲁伊列类型，例如hyperUnique，approxHistogram等 |

## 查询执行

没有聚合的查询将使用德鲁伊的[扫描](http://druid.io/docs/0.12.3/querying/scan-query.html)或[选择](http://druid.io/docs/0.12.3/querying/select-query.html)本机查询类型。尽可能使用扫描，因为它通常比Select更高性能和更高效。但是，在一种情况下使用Select：当查询包含a时`ORDER BY __time`，因为Scan没有排序功能。

聚合查询（使用GROUP BY，DISTINCT或任何聚合函数）将使用Druid的三种本机聚合查询类型之一。两个（Timeseries和TopN）专门用于特定类型的聚合，而另一个（GroupBy）是通用的。

- [时间序列](http://druid.io/docs/0.12.3/querying/timeseriesquery.html)用于GROUP BY `FLOOR(__time TO <unit>)`或`TIME_FLOOR(__time, period)`没有其他分组表达式的查询，没有HAVING或LIMIT子句，没有嵌套，也没有ORDER BY，或者按GROUP BY中的相同表达式排序的ORDER BY。它还使用Timeseries进行具有聚合功能但没有GROUP BY的“总计”查询。此查询类型利用了德鲁伊段按时间排序的事实。
- 默认情况下，[TopN](http://druid.io/docs/0.12.3/querying/topnquery.html)用于按单个表达式分组的查询，具有ORDER BY和LIMIT子句，没有HAVING子句，并且不嵌套。但是，在某些情况下，TopN查询类型将提供近似排名和结果; 如果你想避免这种情况，请将“useApproximateTopN”设置为“false”。TopN结果始终在内存中计算。有关更多详细信息，请参阅TopN文档。
- [GroupBy](http://druid.io/docs/0.12.3/querying/groupbyquery.html)用于所有其他聚合，包括任何嵌套聚合查询。Druid的GroupBy是一个传统的聚合引擎：它提供精确的结果和排名，并支持各种功能。如果可以，GroupBy会在内存中聚合，但如果没有足够的内存来完成查询，它可能会溢出到磁盘。如果ORDER BY GROUP BY子句中的相同表达式，或者根本没有ORDER BY，则结果将通过代理从数据节点流回。如果您的查询具有ORDER BY引用未出现在GROUP BY子句中的表达式（如聚合函数），则代理将在内存中实现结果列表，最多为LIMIT（如果有）。有关调整性能和内存使用的详细信息，请参阅GroupBy文档。

如果您的查询执行嵌套聚合（FROM子句中的聚合子查询），那么Druid将作为[嵌套的GroupBy](http://druid.io/docs/0.12.3/querying/groupbyquery.html#nested-groupbys)执行它 。在嵌套的GroupBys中，最内层的聚合是分布式的，但除此之外的所有外部聚合都在查询代理上本地发生。

包含WHERE子句的半连接查询`col IN (SELECT expr FROM ...)`是使用特殊进程执行的。代理将首先将子查询转换为GroupBy以查找不同的值`expr`。然后，代理将子查询重写为文字过滤器，`col IN (val1, val2, ...)`并运行外部查询。配置参数druid.sql.planner.maxSemiJoinRowsInMemory控制将为此类计划实现的最大值数。

对于所有本机查询类型，`__time`列上的过滤器将尽可能地转换为顶级查询“间隔”，这允许Druid使用其全局时间索引来快速修剪必须扫描的数据集。此外，德鲁伊将使用每个数据节点的本地索引来进一步加速WHERE评估。这通常可以用于涉及对单列的引用和函数的布尔组合的过滤器，例如 `WHERE col1 = 'a' AND col2 = 'b'`但不是`WHERE col1 = col2`。

### 近似算法

在某些情况下，德鲁伊SQL将使用近似算法：

- `COUNT(DISTINCT col)`默认情况下，聚合函数使用[HyperLogLog](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf)的变体， [HyperLogLog](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf)是一种快速近似的不同计数算法。如果通过查询上下文或通过代理配置将“useApproximateCountDistinct”设置为“false”，Druid SQL将切换到完全不同的计数。
- 使用ORDER BY和LIMIT对单个列进行GROUP BY查询可以使用TopN引擎执行，该引擎使用近似算法。如果通过查询上下文或通过代理配置将“useApproximateTopN”设置为“false”，Druid SQL将切换到精确的分组算法。
- 无论配置如何，APPROX_COUNT_DISTINCT和APPROX_QUANTILE聚合函数始终使用近似算法。

## 客户端API

### 通过HTTP的JSON

您可以通过发布到端点使用JSON over HTTP进行Druid SQL查询`/druid/v2/sql/`。请求应该是带有“查询”字段的JSON对象，例如`{"query" : "SELECT COUNT(*) FROM data_source WHERE foo = 'bar'"}`。

结果有两种格式：“对象”（默认; JSON对象的JSON数组）和“数组”（JSON数组的JSON数组）。在“对象”表单中，每行的字段名称将与SQL查询中的列名称匹配。在“数组”形式中，每行的值以SQL查询中指定的顺序返回。

您可以使用*curl*从命令行发送SQL查询：

```bash
$ cat query.json
{"query":"SELECT COUNT(*) AS TheCount FROM data_source"}

$ curl -XPOST -H'Content-Type: application/json' http://BROKER:8082/druid/v2/sql/ -d @query.json
[{"TheCount":24433}]
```

通过查询[“INFORMATION_SCHEMA”表，](http://druid.io/docs/0.12.3/querying/sql.html#retrieving-metadata)可以通过HTTP API获得元数据。

最后，您还可以通过添加“上下文”映射来提供[连接上下文参数](http://druid.io/docs/0.12.3/querying/sql.html#connection-context)，例如：

```json
{
  "query" : "SELECT COUNT(*) FROM data_source WHERE foo = 'bar' AND __time > TIMESTAMP '2000-01-01 00:00:00'",
  "context" : {
    "sqlTimeZone" : "America/Los_Angeles"
  }
}
```

### JDBC

您可以使用[Avatica JDBC驱动程序](https://calcite.apache.org/avatica/downloads/)进行Druid SQL查询。下载Avatica客户端jar后，将其添加到类路径并使用连接字符串`jdbc:avatica:remote:url=http://BROKER:8082/druid/v2/sql/avatica/`。

示例代码：

```java
// Connect to /druid/v2/sql/avatica/ on your broker.
String url = "jdbc:avatica:remote:url=http://localhost:8082/druid/v2/sql/avatica/";

// Set any connection context parameters you need here (see "Connection context" below).
// Or leave empty for default behavior.
Properties connectionProperties = new Properties();

try (Connection connection = DriverManager.getConnection(url, connectionProperties)) {
  try (
      final Statement statement = client.createStatement();
      final ResultSet resultSet = statement.executeQuery(query)
  ) {
    while (resultSet.next()) {
      // Do something
    }
  }
}
```

可以使用`connection.getMetaData()`或通过查询 [“INFORMATION_SCHEMA”表](http://druid.io/docs/0.12.3/querying/sql.html#retrieving-metadata)在JDBC上使用表元数据。参数化查询（使用`?`或其他占位符）无法正常工作，因此请避免使用。

#### 连接粘性

Druid的JDBC服务器不共享代理之间的连接状态。这意味着如果您使用JDBC并拥有多个Druid代理，则应该连接到特定代理，或者使用启用了粘性会话的负载均衡器。

Druid Router节点在平衡JDBC请求时提供连接粘性。有关详细信息，请参阅[路由器](http://druid.io/docs/0.12.3/development/router.html)文档。

请注意，非JDBC [JSON over HTTP](http://druid.io/docs/0.12.3/querying/sql.html#json-over-http) API是无状态的，不需要粘性。

### 连接上下文

Druid SQL支持在客户端上设置连接参数。下表中的参数会影响SQL计划。您提供的所有其他上下文参数将附加到Druid查询，并可能影响它们的运行方式。有关可能选项的详细信息，请参阅 [查询上下文](http://druid.io/docs/0.12.3/querying/query-context.html)

连接上下文可以指定为JDBC连接属性，也可以指定为JSON API中的“上下文”对象。

| 参数                          | 描述                                                         | 默认值                                                |
| ----------------------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
| `sqlTimeZone`                 | 设置此连接的时区，这将影响时间函数和时间戳文字的行为方式。应该是时区名称，如“America / Los_Angeles”或偏移量，如“-08：00”。 | 世界标准时间                                          |
| `useApproximateCountDistinct` | 是否使用近似基数算法`COUNT(DISTINCT foo)`。                  | 代理上的druid.sql.planner.useApproximateCountDistinct |
| `useApproximateTopN`          | 当SQL查询可以表达时，是否使用近似[TopN查询](http://druid.io/docs/0.12.3/querying/topnquery.html)。如果为false，则将使用确切的[GroupBy查询](http://druid.io/docs/0.12.3/querying/groupbyquery.html)。 | 代理上的druid.sql.planner.useApproximateTopN          |
| `useFallback`                 | 是否评估代理上的操作时，它们不能表示为德鲁伊查询。建议不要将此选项用于生产，因为它可以生成不可伸缩的查询计划。如果为false，则无法转换为Druid查询的SQL查询将失败。 | 经纪人的druid.sql.planner.useFallback                 |

### 检索元数据

Druid代理从群集中加载的段中推断每个dataSource的表和列元数据，并使用它来规划SQL查询。此元数据在代理启动时缓存，并在后台通过[SegmentMetadata查询](http://druid.io/docs/0.12.3/querying/segmentmetadataquery.html)定期更新 。后台元数据刷新由进入和退出群集的段触发，也可以通过配置进行限制。

您可以使用JDBC `connection.getMetaData()`或通过下面描述的INFORMATION_SCHEMA表通过JDBC访问表和列元数据。例如，要检索Druid数据源“foo”的元数据，请使用以下查询：

```sql
SELECT * FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_SCHEMA = 'druid' AND TABLE_NAME = 'foo'
```

### SCHEMATA表

| 柱                            | 笔记   |
| ----------------------------- | ------ |
| CATALOG_NAME                  | 没用过 |
| SCHEMA_NAME                   |        |
| SCHEMA_OWNER                  | 没用过 |
| DEFAULT_CHARACTER_SET_CATALOG | 没用过 |
| DEFAULT_CHARACTER_SET_SCHEMA  | 没用过 |
| DEFAULT_CHARACTER_SET_NAME    | 没用过 |
| SQL_PATH                      | 没用过 |

### 表格表

| 柱            | 笔记                    |
| ------------- | ----------------------- |
| TABLE_CATALOG | 没用过                  |
| TABLE_SCHEMA  |                         |
| TABLE_NAME    |                         |
| TABLE_TYPE    | “TABLE”或“SYSTEM_TABLE” |

### COLUMNS表

| 柱                       | 笔记                                   |
| ------------------------ | -------------------------------------- |
| TABLE_CATALOG            | 没用过                                 |
| TABLE_SCHEMA             |                                        |
| TABLE_NAME               |                                        |
| COLUMN_NAME              |                                        |
| ORDINAL_POSITION         |                                        |
| COLUMN_DEFAULT           | 没用过                                 |
| IS_NULLABLE              |                                        |
| 数据类型                 |                                        |
| CHARACTER_MAXIMUM_LENGTH | 没用过                                 |
| CHARACTER_OCTET_LENGTH   | 没用过                                 |
| NUMERIC_PRECISION        |                                        |
| NUMERIC_PRECISION_RADIX  |                                        |
| NUMERIC_SCALE            |                                        |
| DATETIME_PRECISION       |                                        |
| CHARACTER_SET_NAME       |                                        |
| COLLATION_NAME           |                                        |
| JDBC_TYPE                | 从java.sql.Types输入代码（德鲁伊扩展） |

## 服务器配置

通过代理上的以下属性配置Druid SQL服务器。

| 属性                                            | 描述                                                         | 默认   |
| ----------------------------------------------- | ------------------------------------------------------------ | ------ |
| `druid.sql.enable`                              | 是否要启用SQL，包括后台元数据获取。如果为false，则会覆盖所有其他与SQL相关的属性，并完全禁用SQL元数据，服务和计划。 | 假     |
| `druid.sql.avatica.enable`                      | 是否启用JDBC查询`/druid/v2/sql/avatica/`。                   | 真正   |
| `druid.sql.avatica.maxConnections`              | Avatica服务器的最大打开连接数。这些不是HTTP连接，而是可以跨多个HTTP连接的逻辑客户端连接。 | 50     |
| `druid.sql.avatica.maxRowsPerFrame`             | 在单个JDBC框架中返回的最大行数。将此属性设置为-1表示不应应用行限制。客户端可以选择在其请求中指定行限制; 如果客户端指定行限制，则将使用客户端提供的限制的较小值`maxRowsPerFrame`。 | 100000 |
| `druid.sql.avatica.maxStatementsPerConnection`  | 每个Avatica客户端连接的最大同时打开语句数。                  | 1      |
| `druid.sql.avatica.connectionIdleTimeout`       | Avatica客户端连接空闲超时。                                  | PT5M   |
| `druid.sql.http.enable`                         | 是否通过HTTP查询启用JSON`/druid/v2/sql/`。                   | 真正   |
| `druid.sql.planner.maxQueryCount`               | 要发出的最大查询数，包括嵌套查询。设置为1表示禁用子查询，或设置为0表示无限制。 | 8      |
| `druid.sql.planner.maxSemiJoinRowsInMemory`     | 内存中用于执行两阶段半连接查询的最大行数，例如`SELECT * FROM Employee WHERE DeptName IN (SELECT DeptName FROM Dept)`。 | 100000 |
| `druid.sql.planner.maxTopNLimit`                | [TopN查询的](http://druid.io/docs/0.12.3/querying/topnquery.html)最大阈值。相反，将计划更高的限制作为[GroupBy查询](http://druid.io/docs/0.12.3/querying/groupbyquery.html)。 | 100000 |
| `druid.sql.planner.metadataRefreshPeriod`       | 节流元数据刷新。                                             | PT1M   |
| `druid.sql.planner.selectPageSize`              | [选择查询的](http://druid.io/docs/0.12.3/querying/select-query.html)页面大小阈值。对于较大的结果集的选择查询将使用分页背靠背发出。 | 1000   |
| `druid.sql.planner.useApproximateCountDistinct` | 是否使用近似基数算法`COUNT(DISTINCT foo)`。                  | 真正   |
| `druid.sql.planner.useApproximateTopN`          | 当SQL查询可以表达时，是否使用近似[TopN查询](http://druid.io/docs/0.12.3/querying/topnquery.html)。如果为false，则将使用确切的[GroupBy查询](http://druid.io/docs/0.12.3/querying/groupbyquery.html)。 | 真正   |
| `druid.sql.planner.useFallback`                 | 是否评估代理上的操作时，它们不能表示为德鲁伊查询。建议不要将此选项用于生产，因为它可以生成不可伸缩的查询计划。如果为false，则无法转换为Druid查询的SQL查询将失败。 | 假     |