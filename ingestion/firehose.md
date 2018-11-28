# 德鲁伊Firehoses

Firehoses用于[本机批量摄取任务](http://druid.io/docs/0.12.3/ingestion/native-batch.html)，由[Tranquility](http://druid.io/docs/0.12.3/ingestion/stream-push.html)自动创建的流推送任务，以及[流 - 拉（已弃用）](http://druid.io/docs/0.12.3/ingestion/stream-pull.html)摄取模型。

它们是可插拔的，因此配置模式可以并且将根据`type`firehose 而变化。

| 领域 | 类型 | 描述                                                         | 需要 |
| ---- | ---- | ------------------------------------------------------------ | ---- |
| 类型 | 串   | 指定firehose的类型。每个值都有自己的配置架构，下面介绍用Druid打包的firehoses。 | 是   |

## 额外的Firehoses

德鲁伊有几种火炬可供使用，有些是作为例子，其他可以直接在生产环境中使用。

有关其他消防服务，请参阅我们的[扩展名单](http://druid.io/docs/0.12.3/development/extensions.html)。

### LocalFirehose

此Firehose可用于从本地磁盘上的文件读取数据。它可以用于POC来摄取磁盘上的数据。示例本地firehose规范如下所示：

```json
{
    "type"    : "local",
    "filter"   : "*.csv",
    "baseDir"  : "/data/directory"
}
```

| 属性    | 描述                                                         | 需要？ |
| ------- | ------------------------------------------------------------ | ------ |
| 类型    | 这应该是“本地的”。                                           | 是     |
| 过滤    | 文件的通配符过滤器。有关更多信息，请参见[此处](http://commons.apache.org/proper/commons-io/apidocs/org/apache/commons/io/filefilter/WildcardFileFilter.html) | 是     |
| BASEDIR | 目录以递归方式搜索要摄取的文件。                             | 是     |

### HttpFirehose

此Firehose可用于通过HTTP从远程站点读取数据。示例http firehose规范如下所示：

```json
{
    "type"    : "http",
    "uris"  : ["http://example.com/uri1", "http://example2.com/uri2"]
}
```

以下配置可以选择用于调整firehose性能。

| 属性                  | 描述                                                         | 默认                      |
| --------------------- | ------------------------------------------------------------ | ------------------------- |
| maxCacheCapacityBytes | 缓存空间的最大大小（以字节为单位）。0表示禁用缓存。在摄取任务完成之前，不会删除缓存文件。 | 1073741824                |
| maxFetchCapacityBytes | 获取空间的最大大小（以字节为单位）。0表示禁用预取。读取后立即删除预取的文件。 | 1073741824                |
| prefetchTriggerBytes  | 触发预取http对象的阈值。                                     | maxFetchCapacityBytes / 2 |
| fetchTimeout          | 获取http对象的超时时间。                                     | 60000                     |
| maxFetchRetry         | 获取http对象的最大重试次数。                                 | 3                         |

### IngestSegmentFirehose

此Firehose可用于读取现有德鲁伊段的数据。它可以用于使用新架构来获取现有的德鲁伊段，并更改段的名称，维度，指标，汇总等。示例摄取firehose规范如下所示 -

```json
{
    "type"    : "ingestSegment",
    "dataSource"   : "wikipedia",
    "interval" : "2013-01-01/2013-01-02"
}
```

| 属性   | 描述                                                         | 需要？ |
| ------ | ------------------------------------------------------------ | ------ |
| 类型   | 这应该是“ingestSegment”。                                    | 是     |
| 数据源 | 定义数据源以从中获取行的字符串，非常类似于关系数据库中的表   | 是     |
| 间隔   | 表示ISO-8601间隔的字符串。这定义了获取数据的时间范围。       | 是     |
| 尺寸   | 要选择的维度列表。如果留空，则不返回任何尺寸。如果保留为null或未定义，则返回所有维度。 | 没有   |
| 指标   | 要选择的指标列表。如果留空，则不返回任何指标。如果保留为null或未定义，则选择所有度量标准。 | 没有   |
| 过滤   | 见[滤波器](http://druid.io/docs/0.12.3/querying/filters.html) | 没有   |

### CombiningFirehose

此firehose可用于组合和合并来自不同firehoses列表的数据。这可以用于合并来自多个firehose的数据。

```json
{
    "type"  :   "combining",
    "delegates" : [ { firehose1 }, { firehose2 }, ..... ]
}
```

| 属性 | 描述                        | 需要？ |
| ---- | --------------------------- | ------ |
| 类型 | 这应该是“结合”              | 是     |
| 代表 | 用于组合数据的firehoses列表 | 是     |

### 流火焰

下面显示的firehoses应该只与[stream-pull（已弃用）](http://druid.io/docs/0.12.3/ingestion/stream-pull.html)摄取模型一起使用，因为它们不适合批量摄取。

EventReceiverFirehose也用于[Tranquility流推送](http://druid.io/docs/0.12.3/ingestion/stream-push.html)自动生成的任务。

#### EventReceiverFirehose

EventReceiverFirehoseFactory可用于使用http端点摄取事件。

```json
{
  "type": "receiver",
  "serviceName": "eventReceiverServiceName",
  "bufferSize": 10000
}
```

使用此firehose时，可以通过向http端点提交POST请求来发送事件：

```
http://<peonHost>:<port>/druid/worker/v1/chat/<eventReceiverServiceName>/push-events/
```

| 属性       | 描述                             | 需要？               |
| ---------- | -------------------------------- | -------------------- |
| 类型       | 这应该是“接收者”                 | 是                   |
| 服务名称   | 用于宣告事件接收器服务端点的名称 | 是                   |
| 缓冲区大小 | firehose用于存储事件的缓冲区大小 | 没有默认值（100000） |

可以通过提交POST请求来指定EventReceiverFirehose的关闭时间

```
http://<peonHost>:<port>/druid/worker/v1/chat/<eventReceiverServiceName>/shutdown?shutoffTime=<shutoffTime>
```

如果未指定shutOffTime，则firehose会立即关闭。

#### TimedShutoffFirehose

这可用于启动将在指定时间关闭的firehose。一个例子如下所示：

```json
{
    "type"  :   "timed",
    "shutoffTime": "2015-08-25T01:26:05.119Z",
    "delegate": {
          "type": "receiver",
          "serviceName": "eventReceiverServiceName",
          "bufferSize": 100000
     }
}
```

| 属性        | 描述                            | 需要？ |
| ----------- | ------------------------------- | ------ |
| 类型        | 这应该是“定时”                  | 是     |
| shutoffTime | 以ISO8601格式关闭firehose的时间 | 是     |
| 代表        | 使用firehose                    | 是     |