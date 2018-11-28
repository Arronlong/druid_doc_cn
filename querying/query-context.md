# 查询上下文

查询上下文用于各种查询配置参数。以下参数适用于所有查询。

| 属性                         | 默认                                      | 描述                                                         |
| ---------------------------- | ----------------------------------------- | ------------------------------------------------------------ |
| 超时                         | `druid.server.http.defaultQueryTimeout`   | 以毫秒为单位的查询超时，超过该超时将取消未完成的查询。0超时意味着`no timeout`。要设置默认超时，请参阅[代理配置](http://druid.io/docs/0.12.3/configuration/index.html#broker) |
| maxScatterGatherBytes        | `druid.server.http.maxScatterGatherBytes` | 从数据节点（例如历史记录和实时进程）收集的最大字节数，以执行查询。此参数可用于`maxScatterGatherBytes`在查询时进一步减少限制。有关详细信息，请参阅[代理配置](http://druid.io/docs/0.12.3/configuration/index.html#broker) |
| 优先                         | `0`                                       | 查询优先级。优先级较高的查询优先于计算资源。                 |
| queryId                      | 自动生成                                  | 为此查询提供的唯一标识符。如果设置或已知查询ID，则可以使用此ID来取消查询 |
| useCache将                   | `true`                                    | 指示是否为此查询利用查询缓存的标志。设置为false时，将禁用从此查询的查询缓存中读取。当设置为true时，Druid使用druid.broker.cache.useCache或druid.historical.cache.useCache来确定是否从查询缓存中读取 |
| populateCache                | `true`                                    | 指示是否将查询结果保存到查询缓存的标志。主要用于调试。设置为false时，会禁用将此查询的结果保存到查询缓存中。当设置为true时，Druid使用druid.broker.cache.populateCache或druid.historical.cache.populateCache来确定是否将此查询的结果保存到查询缓存中 |
| bySegment                    | `false`                                   | 返回“按细分”结果。主要用于调试，将其设置为`true`返回与它们来自的数据段相关联的结果 |
| 敲定                         | `true`                                    | 指示是否“最终确定”聚合结果的标志。主要用于调试。例如，`hyperUnique`当此标志设置为时，聚合器将返回完整的HyperLogLog草图而不是估计的基数`false` |
| chunkPeriod                  | `P0D` （关闭）                            | 在代理节点级别，可以将长间隔查询（任何类型）分解为更短的间隔查询，以并行化比正常更多的合并。分解的查询将使用更大比例的群集资源，但结果可能更快完成。使用ISO 8601期间。例如，如果此属性设置为`P1M`（一个月），则覆盖一年的查询将分为12个较小的查询。代理使用其查询处理执行程序服务来启动查询块的处理，因此请确保在代理上正确配置“druid.processing.numThreads”。[groupBy查询](http://druid.io/docs/0.12.3/querying/groupbyquery.html)默认情况下不支持chunkPeriod，尽管如果使用旧的“v1”引擎它们会这样做。 |
| serializeDateTimeAsLong      | `false`                                   | 如果为true，则在代理返回的结果和代理与计算节点之间的数据传输中将DateTime序列化为long |
| serializeDateTimeAsLongInner | `false`                                   | 如果为true，则在代理和计算节点之间的数据传输中将DateTime序列化 |

此外，某些查询类型提供特定于该查询类型的上下文参数。

### TopN查询

| 属性             | 默认   | 描述                                                         |
| ---------------- | ------ | ------------------------------------------------------------ |
| minTopNThreshold | `1000` | 返回每个段的最高minTopNThreshold本地结果以进行合并以确定全局topN。 |

### 时间序列查询

| 属性             | 默认    | 描述                                                 |
| ---------------- | ------- | ---------------------------------------------------- |
| skipEmptyBuckets | `false` | 禁用时间序列零填充行为，因此仅返回包含结果的存储区。 |

### GroupBy查询

请参阅[GroupBy查询上下文](http://druid.io/docs/0.12.3/querying/groupbyquery.html#query-context)。