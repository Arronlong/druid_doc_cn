# 选择查询

选择查询返回原始德鲁伊行并支持分页。

```json
 {
   "queryType": "select",
   "dataSource": "wikipedia",
   "descending": "false",
   "dimensions":[],
   "metrics":[],
   "granularity": "all",
   "intervals": [
     "2013-01-01/2013-01-02"
   ],
   "pagingSpec":{"pagingIdentifiers": {}, "threshold":5}
 }
```

如果不需要分页，请考虑使用[扫描查询]（scan-query.html）而不是Select查询，并且不需要Select查询提供的严格时间升序或降序排序。扫描查询返回没有分页的结果，并提供比“选择”更“宽松”的排序，但在处理时间和内存要求方面效率明显更高。它还能够返回几乎无限数量的结果。

选择查询有几个主要部分：

| 属性       | 描述                                                         | 需要？ |
| ---------- | ------------------------------------------------------------ | ------ |
| 查询类型   | 该String应始终为“select”; 这是德鲁伊首先想弄清楚如何解释查询 | 是     |
| 数据源     | 定义要查询的数据源的String或Object，非常类似于关系数据库中的表。有关更多信息，请参阅[DataSource](http://druid.io/docs/0.12.3/querying/datasource.html)。 | 是     |
| 间隔       | 表示ISO-8601间隔的JSON对象。这定义了运行查询的时间范围。     | 是     |
| 降         | 是否降序排序结果。默认为`false`（升序）。如果是这样`true`，页面标识符和偏移量将为负值。 | 没有   |
| 过滤       | 见[滤波器](http://druid.io/docs/0.12.3/querying/filters.html) | 没有   |
| 尺寸       | 要选择的JSON维度列表; 或者参见[DimensionSpec](http://druid.io/docs/0.12.3/querying/dimensionspecs.html)了解提取尺寸的方法。如果留空，则返回所有尺寸。 | 没有   |
| 指标       | 要选择的字符串数组。如果留空，则返回所有指标。               | 没有   |
| 粒度       | 定义查询的粒度。见[粒度](http://druid.io/docs/0.12.3/querying/granularities.html)。默认是`Granularity.ALL`。 | 没有   |
| pagingSpec | 一个JSON对象，指示偏移到不同的扫描段。查询结果将返回一个`pagingIdentifiers`值，该值可在下一个查询分页中重用。 | 是     |
| 上下文     | 一个额外的JSON对象，可用于指定某些标志。                     | 没有   |

结果的格式是：

```json
 [{
  "timestamp" : "2013-01-01T00:00:00.000Z",
  "result" : {
    "pagingIdentifiers" : {
      "wikipedia_2012-12-29T00:00:00.000Z_2013-01-10T08:00:00.000Z_2013-01-10T08:13:47.830Z_v9" : 4
    },
    "events" : [ {
      "segmentId" : "wikipedia_editstream_2012-12-29T00:00:00.000Z_2013-01-10T08:00:00.000Z_2013-01-10T08:13:47.830Z_v9",
      "offset" : 0,
      "event" : {
        "timestamp" : "2013-01-01T00:00:00.000Z",
        "robot" : "1",
        "namespace" : "article",
        "anonymous" : "0",
        "unpatrolled" : "0",
        "page" : "11._korpus_(NOVJ)",
        "language" : "sl",
        "newpage" : "0",
        "user" : "EmausBot",
        "count" : 1.0,
        "added" : 39.0,
        "delta" : 39.0,
        "variation" : 39.0,
        "deleted" : 0.0
      }
    }, {
      "segmentId" : "wikipedia_2012-12-29T00:00:00.000Z_2013-01-10T08:00:00.000Z_2013-01-10T08:13:47.830Z_v9",
      "offset" : 1,
      "event" : {
        "timestamp" : "2013-01-01T00:00:00.000Z",
        "robot" : "0",
        "namespace" : "article",
        "anonymous" : "0",
        "unpatrolled" : "0",
        "page" : "112_U.S._580",
        "language" : "en",
        "newpage" : "1",
        "user" : "MZMcBride",
        "count" : 1.0,
        "added" : 70.0,
        "delta" : 70.0,
        "variation" : 70.0,
        "deleted" : 0.0
      }
    }, {
      "segmentId" : "wikipedia_2012-12-29T00:00:00.000Z_2013-01-10T08:00:00.000Z_2013-01-10T08:13:47.830Z_v9",
      "offset" : 2,
      "event" : {
        "timestamp" : "2013-01-01T00:00:00.000Z",
        "robot" : "0",
        "namespace" : "article",
        "anonymous" : "0",
        "unpatrolled" : "0",
        "page" : "113_U.S._243",
        "language" : "en",
        "newpage" : "1",
        "user" : "MZMcBride",
        "count" : 1.0,
        "added" : 77.0,
        "delta" : 77.0,
        "variation" : 77.0,
        "deleted" : 0.0
      }
    }, {
      "segmentId" : "wikipedia_2012-12-29T00:00:00.000Z_2013-01-10T08:00:00.000Z_2013-01-10T08:13:47.830Z_v9",
      "offset" : 3,
      "event" : {
        "timestamp" : "2013-01-01T00:00:00.000Z",
        "robot" : "0",
        "namespace" : "article",
        "anonymous" : "0",
        "unpatrolled" : "0",
        "page" : "113_U.S._73",
        "language" : "en",
        "newpage" : "1",
        "user" : "MZMcBride",
        "count" : 1.0,
        "added" : 70.0,
        "delta" : 70.0,
        "variation" : 70.0,
        "deleted" : 0.0
      }
    }, {
      "segmentId" : "wikipedia_2012-12-29T00:00:00.000Z_2013-01-10T08:00:00.000Z_2013-01-10T08:13:47.830Z_v9",
      "offset" : 4,
      "event" : {
        "timestamp" : "2013-01-01T00:00:00.000Z",
        "robot" : "0",
        "namespace" : "article",
        "anonymous" : "0",
        "unpatrolled" : "0",
        "page" : "113_U.S._756",
        "language" : "en",
        "newpage" : "1",
        "user" : "MZMcBride",
        "count" : 1.0,
        "added" : 68.0,
        "delta" : 68.0,
        "variation" : 68.0,
        "deleted" : 0.0
      }
    } ]
  }
} ]
```

该`threshold`决定有多少点击返回，由偏移索引的每命中。如果`descending`为true，则偏移量将为负值。

以上结果包括：

```json
    "pagingIdentifiers" : {
      "wikipedia_2012-12-29T00:00:00.000Z_2013-01-10T08:00:00.000Z_2013-01-10T08:13:47.830Z_v9" : 4
    },
```

### 结果分页

PagingSpec允许用户指定选择查询的结果应作为分页集返回。

该`threshold`选项控制每个分页结果块中返回的行数。

要启动分页查询，用户应指定具有`threshold`集合和空白`pagingIdentifiers`字段的PagingSpec ，例如：

```json
"pagingSpec":{"pagingIdentifiers": {}, "threshold":5}
```

当查询返回时，结果将包含一个`pagingIndentifers`字段，指示结果集中的当前分页点（标识符和偏移量）。

要检索结果集的下一部分，用户应发出相同的选择查询，但将`pagingIdentifiers`查询字段替换`pagingIdentifiers`为先前结果。

收到空结果集时，返回所有行。

#### fromNext向后兼容性

在旧版本的Druid中，当使用分页选择查询时，`pagingIdentifiers`在提交下一个查询以检索下一组结果之前，用户必须手动将分页偏移量增加1 。默认情况下，此偏移量增量会在当前版本的Druid中自动发生，用户无需修改`pagingIdentifiers`偏移量即可检索下一个结果集。

将`fromNext`PagingSpec 的字段设置为false指示Druid在旧模式下操作，其中用户必须手动递增偏移（或递减查询的递减）。

例如，假设用户发出以下初始分页查询，并带有`fromNext`false：

```json
 {
   "queryType": "select",
   "dataSource": "wikipedia",
   "descending": "false",
   "dimensions":[],
   "metrics":[],
   "granularity": "all",
   "intervals": [
     "2013-01-01/2013-01-02"
   ],
   "pagingSpec":{"fromNext": "false", "pagingIdentifiers": {}, "threshold":5}
 }
```

`fromNext`设置为false 的分页查询返回结果集，其中包含以下内容`pagingIdentifiers`：

```json
    "pagingIdentifiers" : {
      "wikipedia_2012-12-29T00:00:00.000Z_2013-01-10T08:00:00.000Z_2013-01-10T08:13:47.830Z_v9" : 4
    },
```

要检索下一个结果集，必须发送下一个查询，其中分页偏移量（4）递增1。

```json
 {
   "queryType": "select",
   "dataSource": "wikipedia",
   "dimensions":[],
   "metrics":[],
   "granularity": "all",
   "intervals": [
     "2013-01-01/2013-01-02"
   ],
   "pagingSpec":{"fromNext": "false", "pagingIdentifiers": {"wikipedia_2012-12-29T00:00:00.000Z_2013-01-10T08:00:00.000Z_2013-01-10T08:13:47.830Z_v9" : 5}, "threshold":5}
 }
```

请注意，指定`fromNext`选项`pagingSpec`会覆盖`druid.query.select.enableFromNextDefault`服务器配置中的默认值。请参阅[服务器配置](http://druid.io/docs/0.12.3/querying/select-query.html#server-configuration)以获取更多详

### 服务器配置

以下运行时属性适用于选择查询：

| 属性                                       | 描述                                                         | 默认 |
| ------------------------------------------ | ------------------------------------------------------------ | ---- |
| `druid.query.select.enableFromNextDefault` | 如果未指定`fromNext`查询中的属性`pagingSpec`，系统将使用此属性的值作为默认值`fromNext`。默认情况下，此选项为true：默认设置`fromNext`为false 的选项旨在支持部署的向后兼容性，其中某些用户可能仍然期望旧版Druid的行为。 | 真正 |