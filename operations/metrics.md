# 德鲁伊指标

德鲁伊生成与查询，摄取和协调相关的指标。

度量标准作为JSON对象发布到运行时日志文件或HTTP（发布到Apache Kafka等服务）。默认情况下禁用公制发射。

所有德鲁伊指标都有一组共同的字段：

- `timestamp` - 度量标准的创建时间
- `metric` - 指标的名称
- `service` - 发出度量标准的服务名称
- `host` - 发出度量标准的主机名
- `value` - 与度量标准关联的一些数值

指标可能具有超出上述范围的其他维度。

大多数度量值重置每个发射周期。默认情况下，德鲁伊发射时间为1分钟，可以通过设置属性来更改`druid.monitoring.emissionPeriod`。

## 可用指标

## 查询指标

### 经纪人

| 公                         | 描述                                                         | 外形尺寸                                                     | 正常值 |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------ |
| `query/time`               | 完成查询所花费的毫秒数。                                     | 常用：dataSource，type，interval，hasFilters，duration，context，remoteAddress，id。聚合查询：numMetrics，numComplexMetrics。GroupBy：numDimensions。TopN：阈值，维度。 | <1s    |
| `query/bytes`              | 查询响应中返回的字节数。                                     | 常用：dataSource，type，interval，hasFilters，duration，context，remoteAddress，id。聚合查询：numMetrics，numComplexMetrics。GroupBy：numDimensions。TopN：阈值，维度。 |        |
| `query/node/time`          | 查询各个历史/实时节点所花费的毫秒数。                        | id，status，server。                                         | <1s    |
| `query/node/bytes`         | 查询各个历史/实时节点返回的字节数。                          | id，status，server。                                         |        |
| `query/node/ttfb`          | 第一个字节的时间。经纪人开始接收来自各个历史/实时节点的响应之前经过的毫秒数。 | id，status，server。                                         | <1s    |
| `query/intervalChunk/time` | 仅在启用间隔分块时才会发出。查询间隔块所需的毫秒数。         | id，status，chunkInterval（如果启用了间隔分块）。            | <1s    |
| `query/success/count`      | 成功处理的查询数                                             | 仅当包含QueryCountStatsMonitor模块时，此度量标准才可用。     |        |
| `query/failed/count`       | 失败的查询数                                                 | 仅当包含QueryCountStatsMonitor模块时，此度量标准才可用。     |        |
| `query/interrupted/count`  | 由于取消或超时而中断的查询数                                 | 仅当包含QueryCountStatsMonitor模块时，此度量标准才可用。     |        |

### 历史的

| 公                           | 描述                                                         | 外形尺寸                                                     | 正常值    |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | --------- |
| `query/time`                 | 完成查询所花费的毫秒数。                                     | 常用：dataSource，type，interval，hasFilters，duration，context，remoteAddress，id。聚合查询：numMetrics，numComplexMetrics。GroupBy：numDimensions。TopN：阈值，维度。 | <1s       |
| `query/segment/time`         | 查询单个段所花费的毫秒数。包括从磁盘分段的页面时间。         | id，status，segment。                                        | 几百毫秒  |
| `query/wait/time`            | 等待扫描段的毫秒数。                                         | id，segment。                                                | <几百毫秒 |
| `segment/scan/pending`       | 队列中等待扫描的段数。                                       |                                                              | 接近0     |
| `query/segmentAndCache/time` | 查询单个段或命中高速缓存所花费的毫秒数（如果在历史节点上启用）。 | id，segment。                                                | 几百毫秒  |
| `query/cpu/time`             | 完成查询所需的CPU时间的微秒数                                | 常用：dataSource，type，interval，hasFilters，duration，context，remoteAddress，id。聚合查询：numMetrics，numComplexMetrics。GroupBy：numDimensions。TopN：阈值，维度。 | 不定      |
| `query/success/count`        | 成功处理的查询数                                             | 仅当包含QueryCountStatsMonitor模块时，此度量标准才可用。     |           |
| `query/failed/count`         | 失败的查询数                                                 | 仅当包含QueryCountStatsMonitor模块时，此度量标准才可用。     |           |
| `query/interrupted/count`    | 由于取消或超时而中断的查询数                                 | 仅当包含QueryCountStatsMonitor模块时，此度量标准才可用。     |           |

### 即时的

| 公                        | 描述                         | 外形尺寸                                                     | 正常值   |
| ------------------------- | ---------------------------- | ------------------------------------------------------------ | -------- |
| `query/time`              | 完成查询所花费的毫秒数。     | 常用：dataSource，type，interval，hasFilters，duration，context，remoteAddress，id。聚合查询：numMetrics，numComplexMetrics。GroupBy：numDimensions。TopN：阈值，维度。 | <1s      |
| `query/wait/time`         | 等待扫描段的毫秒数。         | id，segment。                                                | 几百毫秒 |
| `segment/scan/pending`    | 队列中等待扫描的段数。       |                                                              | 接近0    |
| `query/success/count`     | 成功处理的查询数             | 仅当包含QueryCountStatsMonitor模块时，此度量标准才可用。     |          |
| `query/failed/count`      | 失败的查询数                 | 仅当包含QueryCountStatsMonitor模块时，此度量标准才可用。     |          |
| `query/interrupted/count` | 由于取消或超时而中断的查询数 | 仅当包含QueryCountStatsMonitor模块时，此度量标准才可用。     |          |

### 码头

| 公                         | 描述               | 正常值                 |
| -------------------------- | ------------------ | ---------------------- |
| `jetty/numOpenConnections` | 开放式码头连接数。 | 没有比码头数量多得多。 |

### 高速缓存

| 公                    | 描述                     | 正常值 |
| --------------------- | ------------------------ | ------ |
| `query/cache/delta/*` | 自上次发射以来缓存指标。 |        |
| `query/cache/total/*` | 总缓存指标。             |        |

| 公              | 描述                   | 外形尺寸 | 正常值 |
| --------------- | ---------------------- | -------- | ------ |
| `*/numEntries`  | 缓存条目数。           |          | 变化。 |
| `*/sizeBytes`   | 缓存条目的字节大小。   |          | 变化。 |
| `*/hits`        | 缓存命中数。           |          | 变化。 |
| `*/misses`      | 缓存未命中数。         |          | 变化。 |
| `*/evictions`   | 缓存驱逐的数量。       |          | 变化。 |
| `*/hitRate`     | 缓存命中率。           |          | 〜40％ |
| `*/averageByte` | 平均缓存条目字节大小。 |          | 变化。 |
| `*/timeouts`    | 缓存超时数。           |          | 0      |
| `*/errors`      | 缓存错误数。           |          | 0      |

#### Memcached仅指标

Memcached客户端指标按以下方式报告。这些指标直接来自客户端，而不是来自缓存检索层。

| 公                            | 描述                                                         | 外形尺寸 | 正常值 |
| ----------------------------- | ------------------------------------------------------------ | -------- | ------ |
| `query/cache/memcached/total` | 缓存memcached唯一的度量标准（仅当`druid.cache.type=memcached`）作为其实际值 | 变量     | N / A  |
| `query/cache/memcached/delta` | 将memcached唯一的缓存度量（仅当`druid.cache.type=memcached`）作为先前事件发射的增量 | 变量     | N / A  |

## 摄取指标

仅当RealtimeMetricsMonitor包含在Realtime节点的监视器列表中时，这些度量标准才可用。这些指标是每个排放期的增量。

| 公                             | 描述                                                         | 外形尺寸 | 正常值                                            |
| ------------------------------ | ------------------------------------------------------------ | -------- | ------------------------------------------------- |
| `ingest/events/thrownAway`     | 被拒绝的事件数，因为它们在windowPeriod之外。                 | 数据源。 | 0                                                 |
| `ingest/events/unparseable`    | 由于事件不可解析而被拒绝的事件数。                           | 数据源。 | 0                                                 |
| `ingest/events/processed`      | 每个排放期成功处理的事件数。                                 | 数据源。 | 等于每个排放期的事件数量。                        |
| `ingest/rows/output`           | 持续存在的德鲁伊行数。                                       | 数据源。 | 您的累积事件数量。                                |
| `ingest/persists/count`        | 持续发生的次数。                                             | 数据源。 | 取决于配置。                                      |
| `ingest/persists/time`         | 花费中间持续时间的毫秒数。                                   | 数据源。 | 取决于配置。一般最多几分钟。                      |
| `ingest/persists/cpu`          | 用于进行中间持久化的纳秒时间的CPU时间。                      | 数据源。 | 取决于配置。一般最多几分钟。                      |
| `ingest/persists/backPressure` | 创建持久性任务并阻止等待它们完成所花费的毫秒数。             | 数据源。 | 0或非常低                                         |
| `ingest/persists/failed`       | 失败的持续数量。                                             | 数据源。 | 0                                                 |
| `ingest/handoff/failed`        | 失败的切换次数。                                             | 数据源。 | 0                                                 |
| `ingest/merge/time`            | 花费合并中间段的毫秒数                                       | 数据源。 | 取决于配置。一般最多几分钟。                      |
| `ingest/merge/cpu`             | 用于合并中间段的Cpu时间（纳秒）。                            | 数据源。 | 取决于配置。一般最多几分钟。                      |
| `ingest/handoff/count`         | 发生的切换次数。                                             | 数据源。 | 变化。如果集群正常运行，则每个段粒度周期一般大于0 |
| `ingest/sink/count`            | 没有交付的水槽数量。                                         | 数据源。 | 1〜3个                                            |
| `ingest/events/messageGap`     | 事件中的数据时间与当前系统时间之间的时间间隔。               | 数据源。 | 大于0，取决于事件中携带的时间                     |
| `ingest/kafka/lag`             | 适用于Kafka索引服务。Kafka索引任务消耗的偏移与所有分区中Kafka代理的最新偏移之间的总滞后。该指标的最短发射周期为一分钟。 | 数据源。 | 大于0，不应该是一个非常高的数字                   |

注意：如果JVM不支持当前线程的CPU时间测量，则ingest / merge / cpu和ingest / persists / cpu将为0。

### 索引服务

| 公                    | 描述                                                | 外形尺寸                           | 正常值 |
| --------------------- | --------------------------------------------------- | ---------------------------------- | ------ |
| `task/run/time`       | 运行任务所花费的毫秒数。                            | dataSource，taskType，taskStatus。 | 变化。 |
| `segment/added/bytes` | 创建的新段的大小（以字节为单位）                    | dataSource，taskType，interval。   | 变化。 |
| `segment/moved/bytes` | 通过“移动任务”移动/存档的段的大小（以字节为单位）。 | dataSource，taskType，interval。   | 变化。 |
| `segment/nuked/bytes` | 通过Kill Task删除的段的大小（以字节为单位）。       | dataSource，taskType，interval。   | 变化。 |

## 协调

这些指标适用于德鲁伊协调员，每次协调员运行协调逻辑时都会重置。

| 公                              | 描述                                                         | 外形尺寸     | 正常值 |
| ------------------------------- | ------------------------------------------------------------ | ------------ | ------ |
| `segment/assigned/count`        | 分配给群集中的段数。                                         | 层。         | 变化。 |
| `segment/moved/count`           | 在群集中移动的段数。                                         | 层。         | 变化。 |
| `segment/dropped/count`         | 由于被蒙上阴影而丢弃的细分数量。                             | 层。         | 变化。 |
| `segment/deleted/count`         | 由于规则而丢弃的段数。                                       | 层。         | 变化。 |
| `segment/unneeded/count`        | 由于标记为未使用而丢弃的段数。                               | 层。         | 变化。 |
| `segment/cost/raw`              | 用于成本平衡。托管细分的原始成本。                           | 层。         | 变化。 |
| `segment/cost/normalization`    | 用于成本平衡。托管网段的规范化。                             | 层。         | 变化。 |
| `segment/cost/normalized`       | 用于成本平衡。托管细分的标准化成本。                         | 层。         | 变化。 |
| `segment/loadQueue/size`        | 要加载的段的字节大小。                                       | 服务器。     | 变化。 |
| `segment/loadQueue/failed`      | 无法加载的段数。                                             | 服务器。     | 0      |
| `segment/loadQueue/count`       | 要加载的段数。                                               | 服务器。     | 变化。 |
| `segment/dropQueue/count`       | 要删除的段数。                                               | 服务器。     | 变化。 |
| `segment/size`                  | 可用段的字节大小。                                           | 数据源。     | 变化。 |
| `segment/count`                 | 可用细分数量。                                               | 数据源。     | <max   |
| `segment/overShadowed/count`    | 覆盖过多的细分数量。                                         |              | 变化。 |
| `segment/unavailable/count`     | 在应该在群集中加载的段可用于查询之前，要加载的段数（不包括副本）。 | 数据源。     | 0      |
| `segment/underReplicated/count` | 在应该在群集中加载的段可用于查询之前，要加载的段（包括副本）数。 | 层，数据源。 | 0      |

如果在协调器[动态配置中](http://druid.io/docs/0.12.3/configuration/coordinator.html#dynamic-configuration)`emitBalancingStats`设置为，则类的[日志条目](http://druid.io/docs/0.12.3/configuration/logging.html)将具有关于平衡决策的额外信息。`true``io.druid.server.coordinator.helper.DruidCoordinatorLogger`

## 总体健康

### 历史的

| 公                      | 描述                                   | 外形尺寸                     | 正常值 |
| ----------------------- | -------------------------------------- | ---------------------------- | ------ |
| `segment/max`           | 段可用的最大字节数限制。               |                              | 变化。 |
| `segment/used`          | 用于服务段的字节数。                   | dataSource，tier，priority。 | <max   |
| `segment/usedPercent`   | 服务细分使用的空间百分比。             | dataSource，tier，priority。 | <100％ |
| `segment/count`         | 服务段数。                             | dataSource，tier，priority。 | 变化。 |
| `segment/pendingDelete` | 等待清除的段的磁盘大小（以字节为单位） | 变化。                       |        |

### JVM

仅当包含JVMMonitor模块时，这些度量标准才可用。

| 公                        | 描述             | 外形尺寸             | 正常值         |
| ------------------------- | ---------------- | -------------------- | -------------- |
| `jvm/pool/committed`      | 承诺的池。       | poolKind，poolName。 | 接近最大游泳池 |
| `jvm/pool/init`           | 初始池。         | poolKind，poolName。 | 变化。         |
| `jvm/pool/max`            | 最大游泳池。     | poolKind，poolName。 | 变化。         |
| `jvm/pool/used`           | 使用的池。       | poolKind，poolName。 | <max pool      |
| `jvm/bufferpool/count`    | 缓冲池计数。     | bufferPoolName。     | 变化。         |
| `jvm/bufferpool/used`     | 使用Bufferpool。 | bufferPoolName。     | 接近容量       |
| `jvm/bufferpool/capacity` | 缓冲池容量。     | bufferPoolName。     | 变化。         |
| `jvm/mem/init`            | 初始记忆。       | memKind。            | 变化。         |
| `jvm/mem/max`             | 最大内存。       | memKind。            | 变化。         |
| `jvm/mem/used`            | 用过的记忆。     | memKind。            | <最大记忆      |
| `jvm/mem/committed`       | 承诺的记忆。     | memKind。            | 接近最大记忆   |
| `jvm/gc/count`            | 垃圾收集计数。   | gcName。             | <100           |
| `jvm/gc/time`             | 垃圾收集时间。   | gcName。             | <1s            |

### EventReceiverFirehose

仅当包含EventReceiverFirehoseMonitor模块时，以下度量标准才可用。

| 公                       | 描述                                      | 外形尺寸                                          | 正常值                       |
| ------------------------ | ----------------------------------------- | ------------------------------------------------- | ---------------------------- |
| `ingest/events/buffered` | EventReceiverFirehose缓冲区中排队的事件数 | serviceName，dataSource，taskId，bufferCapacity。 | 等于缓冲队列中的当前事件数。 |
| `ingest/bytes/received`  | EventReceiverFirehose接收的字节数。       | serviceName，dataSource，taskId。                 | 变化。                       |

## SYS

这些指标仅在包含SysMonitor模块时可用。

| 公                     | 描述                                                 | 外形尺寸                                                     | 正常值 |
| ---------------------- | ---------------------------------------------------- | ------------------------------------------------------------ | ------ |
| `sys/swap/free`        | 免费交换。                                           |                                                              | 变化。 |
| `sys/swap/max`         | 最大交换。                                           |                                                              | 变化。 |
| `sys/swap/pageIn`      | 在交换中分页。                                       |                                                              | 变化。 |
| `sys/swap/pageOut`     | 分页交换。                                           |                                                              | 变化。 |
| `sys/disk/write/count` | 写入磁盘。                                           | fsDevName，fsDirName，fsTypeName，fsSysTypeName，fsOptions。 | 变化。 |
| `sys/disk/read/count`  | 从磁盘读取。                                         | fsDevName，fsDirName，fsTypeName，fsSysTypeName，fsOptions。 | 变化。 |
| `sys/disk/write/size`  | 写入磁盘的字节数。我们可以用来确定关于段的分页数。   | fsDevName，fsDirName，fsTypeName，fsSysTypeName，fsOptions。 | 变化。 |
| `sys/disk/read/size`   | 从磁盘读取的字节数。我们可以用来确定关于段的分页数。 | fsDevName，fsDirName，fsTypeName，fsSysTypeName，fsOptions。 | 变化。 |
| `sys/net/write/size`   | 写入网络的字节。                                     | netName，netAddress，netHwaddr                               | 变化。 |
| `sys/net/read/size`    | 从网络读取的字节数。                                 | netName，netAddress，netHwaddr                               | 变化。 |
| `sys/fs/used`          | 使用的文件系统字节。                                 | fsDevName，fsDirName，fsTypeName，fsSysTypeName，fsOptions。 | <max   |
| `sys/fs/max`           | Filesystesm字节最大值                                | fsDevName，fsDirName，fsTypeName，fsSysTypeName，fsOptions。 | 变化。 |
| `sys/mem/used`         | 使用的内存。                                         |                                                              | <max   |
| `sys/mem/max`          | 内存最大                                             |                                                              | 变化。 |
| `sys/storage/used`     | 使用的磁盘空间。                                     | fsDirName。                                                  | 变化。 |
| `sys/cpu`              | 使用的CPU。                                          | cpuName，cpuTime。                                           | 变化。 |