# 摄取的数据格式

Druid可以以JSON，CSV或分隔形式（如TSV）或任何自定义格式摄取非规范化数据。虽然文档中的大多数示例都使用JSON格式的数据，但配置Druid以获取任何其他分隔数据并不困难。我们欢迎对新格式的任何贡献。

有关其他数据格式，请参阅我们的[扩展列表](http://druid.io/docs/0.12.3/development/extensions.html)。

## 格式化数据

以下示例显示了Druid本机支持的数据格式：

*JSON*

```json
{"timestamp": "2013-08-31T01:02:33Z", "page": "Gypsy Danger", "language" : "en", "user" : "nuclear", "unpatrolled" : "true", "newPage" : "true", "robot": "false", "anonymous": "false", "namespace":"article", "continent":"North America", "country":"United States", "region":"Bay Area", "city":"San Francisco", "added": 57, "deleted": 200, "delta": -143}
{"timestamp": "2013-08-31T03:32:45Z", "page": "Striker Eureka", "language" : "en", "user" : "speed", "unpatrolled" : "false", "newPage" : "true", "robot": "true", "anonymous": "false", "namespace":"wikipedia", "continent":"Australia", "country":"Australia", "region":"Cantebury", "city":"Syndey", "added": 459, "deleted": 129, "delta": 330}
{"timestamp": "2013-08-31T07:11:21Z", "page": "Cherno Alpha", "language" : "ru", "user" : "masterYi", "unpatrolled" : "false", "newPage" : "true", "robot": "true", "anonymous": "false", "namespace":"article", "continent":"Asia", "country":"Russia", "region":"Oblast", "city":"Moscow", "added": 123, "deleted": 12, "delta": 111}
{"timestamp": "2013-08-31T11:58:39Z", "page": "Crimson Typhoon", "language" : "zh", "user" : "triplets", "unpatrolled" : "true", "newPage" : "false", "robot": "true", "anonymous": "false", "namespace":"wikipedia", "continent":"Asia", "country":"China", "region":"Shanxi", "city":"Taiyuan", "added": 905, "deleted": 5, "delta": 900}
{"timestamp": "2013-08-31T12:41:27Z", "page": "Coyote Tango", "language" : "ja", "user" : "cancer", "unpatrolled" : "true", "newPage" : "false", "robot": "true", "anonymous": "false", "namespace":"wikipedia", "continent":"Asia", "country":"Japan", "region":"Kanto", "city":"Tokyo", "added": 1, "deleted": 10, "delta": -9}
```

*CSV*

```text
2013-08-31T01:02:33Z,"Gypsy Danger","en","nuclear","true","true","false","false","article","North America","United States","Bay Area","San Francisco",57,200,-143
2013-08-31T03:32:45Z,"Striker Eureka","en","speed","false","true","true","false","wikipedia","Australia","Australia","Cantebury","Syndey",459,129,330
2013-08-31T07:11:21Z,"Cherno Alpha","ru","masterYi","false","true","true","false","article","Asia","Russia","Oblast","Moscow",123,12,111
2013-08-31T11:58:39Z,"Crimson Typhoon","zh","triplets","true","false","true","false","wikipedia","Asia","China","Shanxi","Taiyuan",905,5,900
2013-08-31T12:41:27Z,"Coyote Tango","ja","cancer","true","false","true","false","wikipedia","Asia","Japan","Kanto","Tokyo",1,10,-9
```

*TSV（定界）*

```text
2013-08-31T01:02:33Z    "Gypsy Danger"  "en"    "nuclear"   "true"  "true"  "false" "false" "article"   "North America" "United States" "Bay Area"  "San Francisco" 57  200 -143
2013-08-31T03:32:45Z    "Striker Eureka"    "en"    "speed" "false" "true"  "true"  "false" "wikipedia" "Australia" "Australia" "Cantebury" "Syndey"    459 129 330
2013-08-31T07:11:21Z    "Cherno Alpha"  "ru"    "masterYi"  "false" "true"  "true"  "false" "article"   "Asia"  "Russia"    "Oblast"    "Moscow"    123 12  111
2013-08-31T11:58:39Z    "Crimson Typhoon"   "zh"    "triplets"  "true"  "false" "true"  "false" "wikipedia" "Asia"  "China" "Shanxi"    "Taiyuan"   905 5   900
2013-08-31T12:41:27Z    "Coyote Tango"  "ja"    "cancer"    "true"  "false" "true"  "false" "wikipedia" "Asia"  "Japan" "Kanto" "Tokyo" 1   10  -9
```

请注意，CSV和TSV数据不包含列标题。指定要摄取的数据时，这一点很重要。

## 自定义格式

Druid支持自定义数据格式，可以使用`Regex`解析器或`JavaScript`解析器来解析这些格式。请注意，使用任何这些解析器来解析数据都不如编写本机Java解析器或使用外部流处理器那样高效。我们欢迎新的Parsers的贡献。

## 组态

所有形式的德鲁伊摄取都需要某种形式的模式对象。要使用的`parseSpec`条目指定要摄取的数据的格式`dataSchema`。

### JSON

```json
  "parseSpec":{
    "format" : "json",
    "timestampSpec" : {
      "column" : "timestamp"
    },
    "dimensionSpec" : {
      "dimensions" : ["page","language","user","unpatrolled","newPage","robot","anonymous","namespace","continent","country","region","city"]
    }
  }
```

如果你有嵌套的JSON，[德鲁伊可以自动为你扁平化](http://druid.io/docs/0.12.3/ingestion/flatten-json.html)。

### CSV

```json
  "parseSpec": {
    "format" : "csv",
    "timestampSpec" : {
      "column" : "timestamp"
    },
    "columns" : ["timestamp","page","language","user","unpatrolled","newPage","robot","anonymous","namespace","continent","country","region","city","added","deleted","delta"],
    "dimensionsSpec" : {
      "dimensions" : ["page","language","user","unpatrolled","newPage","robot","anonymous","namespace","continent","country","region","city"]
    }
  }
```

#### CSV索引任务

如果输入文件包含标题，则该`columns`字段是可选字段，您无需进行设置。相反，您可以将该`hasHeaderRow`字段设置为true，这使得Druid会自动从标题中提取列信息。否则，您必须设置该`columns`字段并确保该字段必须以相同的顺序与输入数据的列匹配。

此外，您可以通过`skipHeaderRows`在parseSpec中设置来跳过某些标题行。如果同时设置了两个`skipHeaderRows`和`hasHeaderRow`选项， `skipHeaderRows`则首先应用。例如，如果设置`skipHeaderRows`为2并且`hasHeaderRow`为true，则Druid将跳过前两行，然后从第三行提取列信息。

请注意，`hasHeaderRow`并`skipHeaderRows`仅适用于非Hadoop的批处理指数的任务有效。其他类型的索引任务将失败并出现异常。

#### 其他CSV摄取任务

`columns`必须包含该字段，并确保字段的顺序与输入数据的列按相同顺序匹配。

### TSV（定界）

```json
  "parseSpec": {
    "format" : "tsv",
    "timestampSpec" : {
      "column" : "timestamp"
    },
    "columns" : ["timestamp","page","language","user","unpatrolled","newPage","robot","anonymous","namespace","continent","country","region","city","added","deleted","delta"],
    "delimiter":"|",
    "dimensionsSpec" : {
      "dimensions" : ["page","language","user","unpatrolled","newPage","robot","anonymous","namespace","continent","country","region","city"]
    }
  }
```

请务必将数据更改`delimiter`为适当的分隔符。与CSV一样，您必须指定要编制索引的列和列的子集。

#### TSV（定界）索引任务

如果输入文件包含标题，则该`columns`字段是可选字段，您无需进行设置。相反，您可以将该`hasHeaderRow`字段设置为true，这使得Druid会自动从标题中提取列信息。否则，您必须设置该`columns`字段并确保该字段必须以相同的顺序与输入数据的列匹配。

此外，您可以通过`skipHeaderRows`在parseSpec中设置来跳过某些标题行。如果同时设置了两个`skipHeaderRows`和`hasHeaderRow`选项， `skipHeaderRows`则首先应用。例如，如果设置`skipHeaderRows`为2并且`hasHeaderRow`为true，则Druid将跳过前两行，然后从第三行提取列信息。

请注意，`hasHeaderRow`并`skipHeaderRows`仅适用于非Hadoop的批处理指数的任务有效。其他类型的索引任务将失败并出现异常。

#### 其他TSV（定界）摄取任务

`columns`必须包含该字段，并确保字段的顺序与输入数据的列按相同顺序匹配。

### 正则表达式

```json
  "parseSpec":{
    "format" : "regex",
    "timestampSpec" : {
      "column" : "timestamp"
    },        
    "dimensionsSpec" : {
      "dimensions" : [<your_list_of_dimensions>]
    },
    "columns" : [<your_columns_here>],
    "pattern" : <regex pattern for partitioning data>
  }
```

该`columns`字段必须以相同的顺序匹配正则表达式匹配组的列。如果未提供列，则将分配默认列名称（“column_1”，“column2”，...“column_n”）。确保您的列名包含所有维度。

### JavaScript的

```json
  "parseSpec":{
    "format" : "javascript",
    "timestampSpec" : {
      "column" : "timestamp"
    },        
    "dimensionsSpec" : {
      "dimensions" : ["page","language","user","unpatrolled","newPage","robot","anonymous","namespace","continent","country","region","city"]
    },
    "function" : "function(str) { var parts = str.split(\"-\"); return { one: parts[0], two: parts[1] } }"
  }
```

请注意，JavaScript解析器必须完全解析数据并将其作为`{key:value}`JS逻辑中的格式返回。这意味着必须在此处完成任何展平或解析多维值。

默认情况下禁用基于JavaScript的功能。有关使用德鲁伊JavaScript功能的[指南](http://druid.io/docs/0.12.3/development/javascript.html)，请参阅德鲁伊[JavaScript编程指南](http://druid.io/docs/0.12.3/development/javascript.html)，包括如何启用它的说明。

### 多值维度

尺寸可以具有TSV和CSV数据的多个值。要指定多值维度的分隔符，请在`listDelimiter`中设置`parseSpec`。

JSON数据也可以包含多值维度。必须将维度的多个值格式化为摄取数据中的JSON数组。无需其他`parseSpec`配置。