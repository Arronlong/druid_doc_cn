# 德鲁伊表达

此功能仍处于试验阶段。它尚未针对性能进行优化，并且已知其实施具有显着的低效率。

此表达式语言支持以下运算符（按优先级的降序列出）。

| 运营商                 | 描述         |
| ---------------------- | ------------ |
| ！， -                 | 一元不和减   |
| ^                      | 二元电源操作 |
| *，/，％               | 二进制乘法   |
| +， -                  | 二元添加剂   |
| <，<=，>，> =，==，！= | 二元比较     |
| &&，\                  | \            |

支持长，双和字符串数据类型。如果数字包含点，则将其解释为double，否则将其解释为long。这意味着，总是添加'。' 如果您希望将其解释为double值，请输入您的号码。字符串文字应使用单引号引用。

尚未完全支持多值类型。表达式在多值类型上的行为可能不一致，在这种情况下，您不应该依赖于此行为在将来的版本中保持不变。

表达式可以包含变量。变量名称可能包含字母，数字，'_'和'$'。变量名称不得以数字开头。要转义其他特殊字符，可以使用双引号引用它。

对于逻辑运算符，当且仅当它为正数时才为真（0或负值表示为假）。对于字符串类型，它是'Boolean.valueOf（string）'的评估结果。

可以使用以下内置功能。

## 一般功能

| 名称          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| 投            | cast（expr，'LONG'或'DOUBLE'或'STRING'）返回指定类型的expr。异常可以抛出 |
| 如果          | if（谓词，然后，否则）返回'then'如果'predicate'计算为正数，否则返回'else' |
| NVL           | 如果'expr'为null（或字符串类型为空字符串），则nvl（expr，expr-for-null）返回'expr-for-null' |
| 喜欢          | like（expr，pattern [，escape]）等同于SQL `expr LIKE pattern` |
| case_searched | case_searched（expr1，result1，[[expr2，result2，...]，else-result]） |
| case_simple   | case_simple（expr，value1，result1，[[value2，result2，...]，else-result]） |

## 字符串函数

| 名称           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| CONCAT         | 连接字符串列表                                               |
| 喜欢           | like（expr，pattern [，escape]）等同于SQL `expr LIKE pattern` |
| 抬头           | lookup（expr，lookup-name）在已注册的[查询时查找中查找](http://druid.io/docs/0.12.3/querying/lookups.html) expr |
| REGEXP_EXTRACT | regexp_extract（expr，pattern [，index]）应用正则表达式模式并提取捕获组索引，如果没有匹配则返回null。如果index未指定或为零，则返回与模式匹配的子字符串。 |
| 更换           | replace（expr，pattern，replacement）用替换替换模式          |
| 子             | substring（expr，index，length）的行为类似于java.lang.String的子串 |
| strlen的       | strlen（expr）以UTF-16代码单位返回字符串的长度               |
| strpos         | strpos（haystack，needle）返回大海捞针的位置，索引从0开始。如果未找到针，则函数返回-1。 |
| 修剪           | trim（expr [，chars]）删除前导和尾随字符（`expr`如果它们存在）`chars`。`chars`如果没有提供，则默认为''（空格）。 |
| LTRIM          | ltrim（expr [，chars]）删除前导字符（`expr`如果它们存在）`chars`。`chars`如果没有提供，则默认为''（空格）。 |
| RTRIM          | rtrim（expr [，chars]）删除尾随字符（`expr`如果它们存在）`chars`。`chars`如果没有提供，则默认为''（空格）。 |
| 降低           | lower（expr）将字符串转换为小写                              |
| 上             | upper（expr）将字符串转换为大写                              |

## 时间功能

| 名称              | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| 时间戳            | timestamp（expr [，format-string]）将string expr解析为date，然后从java epoch返回milli-seconds。没有'format-string'，它被视为ISO datetime格式 |
| UNIX_TIMESTAMP    | 与'timestamp'功能相同，但返回秒                              |
| timestamp_ceil    | timestamp_ceil（expr，period，[origin，[timezone]]）对时间戳进行舍入，将其作为新时间戳返回。期间可以是任何ISO8601期间，如P3M（季度）或PT12H（半天）。时区（如果提供）应为时区名称，如“America / Los_Angeles”或偏移量，如“-08：00”。 |
| timestamp_floor   | timestamp_floor（expr，period，[origin，[timezone]]）舍入时间戳，将其作为新时间戳返回。期间可以是任何ISO8601期间，如P3M（季度）或PT12H（半天）。时区（如果提供）应为时区名称，如“America / Los_Angeles”或偏移量，如“-08：00”。 |
| timestamp_shift   | timestamp_shift（expr，period，step，[timezone]）将时间戳移动一个句点（步骤时间），并将其作为新时间戳返回。期间可以是任何ISO8601期间。步骤可能是否定的。时区（如果提供）应为时区名称，如“America / Los_Angeles”或偏移量，如“-08：00”。 |
| timestamp_extract | timestamp_extract（expr，unit，[timezone]）从expr中提取时间部分，并将其作为数字返回。单位可以是EPOCH（自1970-01-01 00:00:00 UTC以来的秒数），秒，分钟，小时，日（星期几），DOW（星期几），DOY（一年中的某一天），周（一周中的[一周](https://en.wikipedia.org/wiki/ISO_week_date)），每月（1到12），季度（1到4）或年份。时区（如果提供）应为时区名称，如“America / Los_Angeles”或偏移量，如“-08：00” |
| timestamp_parse   | timestamp_parse（string expr，[pattern，[timezone]]）使用给定的[Joda DateTimeFormat模式](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)将字符串解析为时间戳。如果未提供模式，则以ISO8601或SQL格式分析时间字符串。时区（如果提供）应为时区名称，如“America / Los_Angeles”或偏移量，如“-08：00”，并将用作不包含时区偏移的字符串的时区。模式和时区必须是文字。无法解析为时间戳的字符串将作为空值返回。 |
| TIMESTAMP_FORMAT  | timestamp_format（expr，[pattern，[timezone]]）将时间戳格式化为具有给定[Joda DateTimeFormat模式](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)的字符串，如果未提供[模式](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)，则格式化为ISO8601。时区（如果提供）应为时区名称，如“America / Los_Angeles”或偏移量，如“-08：00”。模式和时区必须是文字。 |

## 数学函数

有关每个函数的详细说明，请参阅java.lang.Math的javadoc。

| 名称          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| ABS           | abs（x）将返回x的绝对值                                      |
| ACOS          | acos（x）将返回x的反余弦值                                   |
| ASIN          | asin（x）将返回x的反正弦值                                   |
| 晒黑          | atan（x）将返回x的反正切                                     |
| ATAN2         | atan2（y，x）将从直角坐标（x，y）到极坐标（r，theta）的转换返回角度theta |
| CBRT          | cbrt（x）将返回x的立方根                                     |
| 小区          | ceil（x）将返回大于或等于x并且等于数学整数的最小（最接近负无穷大）double值 |
| 复制符号      | copysign（x）将返回带有第二个浮点参数符号的第一个浮点参数    |
| COS           | cos（x）将返回x的三角余弦值                                  |
| 护身用手杖    | cosh（x）将返回x的双曲余弦值                                 |
| DIV           | div（x，y）是x的整数除以y                                    |
| EXP           | exp（x）将Euler的数字提高到x的幂                             |
| 的expm1       | expm1（x）将返回e ^ x-1                                      |
| 地板          | floor（x）将返回小于或等于x并且等于数学整数的最大（最接近正无穷大）double值 |
| getExponent   | getExponent（x）将返回x表示中使用的无偏指数                  |
| hypot将       | hypot（x，y）将返回sqrt（x ^ 2 + y ^ 2）而没有中间溢出或下溢 |
| 日志          | log（x）将返回x的自然对数                                    |
| LOG10         | log10（x）将返回x的基数10对数                                |
| log1p         | log1p（x）是x + 1的自然对数                                  |
| 最大          | max（x，y）将返回两个值中的较大者                            |
| 分            | min（x，y）将返回两个值中较小的一个                          |
| 函数nextafter | nextafter（x，y）将返回y方向上与x相邻的浮点数                |
| 下一部影片    | nextUp（x）将在正无穷大的方向上返回与x相邻的浮点值           |
| POW           | pow（x，y）会将x的值返回到y的幂                              |
| 剩余          | 余数（x，y）将根据IEEE 754标准规定的两个参数返回余数运算     |
| RINT          | rint（x）将返回值与x最接近的值，并且等于数学整数             |
| 回合          | round（x）会将最接近的long值返回到x，并且关系向上舍入        |
| scalb         | scalb（d，sf）将返回d * 2 ^ sf舍入，就好像由单个正确舍入的浮点乘以双值集的成员一样 |
| 正负号        | signum（x）将返回参数x的signum函数                           |
| 罪            | sin（x）将返回角度x的三角正弦                                |
| 双曲正弦      | sinh（x）将返回x的双曲正弦                                   |
| 开方          | sqrt（x）将返回x的正确舍入的正平方根                         |
| 黄褐色        | tan（x）将返回角度x的三角正切                                |
| 正切          | tanh（x）将返回x的双曲正切                                   |
| todegrees     | todegrees（x）将以弧度测量的角度转换为以度为单位测量的近似等效角度 |
| toradians     | toradians（x）将以度为单位测量的角度转换为以弧度为单位测量的近似等效角度 |
| ULP           | ulp（x）将返回参数x的ulp的大小                               |