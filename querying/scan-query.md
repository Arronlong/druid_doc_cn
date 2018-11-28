# 扫描查询

扫描查询以流模式返回原始德鲁伊行。

```json
 {
   "queryType": "scan",
   "dataSource": "wikipedia",
   "resultFormat": "list",
   "columns":[],
   "intervals": [
     "2013-01-01/2013-01-02"
   ],
   "batchSize":20480,
   "limit":5
 }
```

扫描查询有几个主要部分：

| 属性           | 描述                                                         | 需要？ |
| -------------- | ------------------------------------------------------------ | ------ |
| 查询类型       | 该字符串应始终为“扫描”; 这是德鲁伊首先想弄清楚如何解释查询   | 是     |
| 数据源         | 定义要查询的数据源的String或Object，非常类似于关系数据库中的表。有关更多信息，请参阅[DataSource](http://druid.io/docs/0.12.3/querying/datasource.html)。 | 是     |
| 间隔           | 表示ISO-8601间隔的JSON对象。这定义了运行查询的时间范围。     | 是     |
| 的resultFormat | 结果如何表示，列表或compactedList或valueVector。目前只`list`和`compactedList`支持。默认是`list` | 没有   |
| 过滤           | 见[滤波器](http://druid.io/docs/0.12.3/querying/filters.html) | 没有   |
| 列             | 要扫描的维度和指标的String数组。如果留空，则返回所有维度和指标。 | 没有   |
| BATCHSIZE      | 返回客户端之前缓冲了多少行。默认是`20480`                    | 没有   |
| 限制           | 要返回多少行。如果未指定，则将返回所有行。                   | 没有   |
| 遗产           | 返回结果与传统的“scan-query”contrib扩展一致。默认为设置的值`druid.query.scan.legacy`，而默认值为false。有关详细信息，请参阅[旧版模式](http://druid.io/docs/0.12.3/querying/scan-query.html#legacy-mode) | 没有   |
| 上下文         | 一个额外的JSON对象，可用于指定某些标志。                     | 没有   |

## 示例结果

resultFormat等于的结果格式`list`：

```json
 [{
    "segmentId" : "wikipedia_editstream_2012-12-29T00:00:00.000Z_2013-01-10T08:00:00.000Z_2013-01-10T08:13:47.830Z_v9",
    "columns" : [
      "timestamp",
      "robot",
      "namespace",
      "anonymous",
      "unpatrolled",
      "page",
      "language",
      "newpage",
      "user",
      "count",
      "added",
      "delta",
      "variation",
      "deleted"
    ],
    "events" : [ {
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
    }, {
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
    }, {
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
    }, {
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
    }, {
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
    } ]
} ]
```

resultFormat等于的结果格式`compactedList`：

```json
 [{
    "segmentId" : "wikipedia_editstream_2012-12-29T00:00:00.000Z_2013-01-10T08:00:00.000Z_2013-01-10T08:13:47.830Z_v9",
    "columns" : [
      "timestamp", "robot", "namespace", "anonymous", "unpatrolled", "page", "language", "newpage", "user", "count", "added", "delta", "variation", "deleted"
    ],
    "events" : [
     ["2013-01-01T00:00:00.000Z", "1", "article", "0", "0", "11._korpus_(NOVJ)", "sl", "0", "EmausBot", 1.0, 39.0, 39.0, 39.0, 0.0],
     ["2013-01-01T00:00:00.000Z", "0", "article", "0", "0", "112_U.S._580", "en", "1", "MZMcBride", 1.0, 70.0, 70.0, 70.0, 0.0],
     ["2013-01-01T00:00:00.000Z", "0", "article", "0", "0", "113_U.S._243", "en", "1", "MZMcBride", 1.0, 77.0, 77.0, 77.0, 0.0],
     ["2013-01-01T00:00:00.000Z", "0", "article", "0", "0", "113_U.S._73", "en", "1", "MZMcBride", 1.0, 70.0, 70.0, 70.0, 0.0],
     ["2013-01-01T00:00:00.000Z", "0", "article", "0", "0", "113_U.S._756", "en", "1", "MZMcBride", 1.0, 68.0, 68.0, 68.0, 0.0]
    ]
} ]
```

选择查询和扫描查询之间的最大区别在于，扫描查询在将行返回到客户端之前不会保留内存中的所有行。
如果select查询需要太多行，则会导致内存压力。
扫描查询没有此问题。
扫描查询可以返回所有行而不发出另一个分页查询，这在直接查询历史或实时节点时非常有用。

## 传统模式

Scan查询支持传统模式，该模式旨在实现与以前的scan-query contrib扩展的协议兼容性。在传统模式下，您可以预期以下行为更改：

- 该**时间列返回为“时间戳”，而不是“**时间”。这将优先于您可能拥有的名为“timestamp”的任何其他列。
- 即使您没有特别要求，__time列也会包含在列列表中。
- 时间戳以ISO8601时间字符串而不是整数（自1970-01-01 00:00:00 UTC后的毫秒数）返回。

传统模式可以通过传递`"legacy" : true`查询JSON或通过设置 `druid.query.scan.legacy = true`德鲁伊节点来触发。如果您之前使用的是scan-query contrib扩展，则迁移的最佳方法是在滚动升级期间激活旧模式，然后在升级完成后将其关闭。