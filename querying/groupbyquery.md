# groupBy查询

这些类型的查询采用groupBy查询对象并返回JSON对象数组，其中每个对象表示查询要求的分组。

注意：如果您正在使用时间聚合作为唯一的分组，或者在单个维度上执行有序的组，请考虑[Timeseries](http://druid.io/docs/0.12.3/querying/timeseriesquery.html)和[TopN](http://druid.io/docs/0.12.3/querying/topnquery.html)查询以及groupBy。在某些情况下，他们的表现可能更好。有关详细信息，请参阅下面的 [替代](http://druid.io/docs/0.12.3/querying/groupbyquery.html#alternatives)

示例groupBy查询对象如下所示：

```json
{
  "queryType": "groupBy",
  "dataSource": "sample_datasource",
  "granularity": "day",
  "dimensions": ["country", "device"],
  "limitSpec": { "type": "default", "limit": 5000, "columns": ["country", "data_transfer"] },
  "filter": {
    "type": "and",
    "fields": [
      { "type": "selector", "dimension": "carrier", "value": "AT&T" },
      { "type": "or", 
        "fields": [
          { "type": "selector", "dimension": "make", "value": "Apple" },
          { "type": "selector", "dimension": "make", "value": "Samsung" }
        ]
      }
    ]
  },
  "aggregations": [
    { "type": "longSum", "name": "total_usage", "fieldName": "user_count" },
    { "type": "doubleSum", "name": "data_transfer", "fieldName": "data_transfer" }
  ],
  "postAggregations": [
    { "type": "arithmetic",
      "name": "avg_usage",
      "fn": "/",
      "fields": [
        { "type": "fieldAccess", "fieldName": "data_transfer" },
        { "type": "fieldAccess", "fieldName": "total_usage" }
      ]
    }
  ],
  "intervals": [ "2012-01-01T00:00:00.000/2012-01-03T00:00:00.000" ],
  "having": {
    "type": "greaterThan",
    "aggregation": "total_usage",
    "value": 100
  }
}
```

groupBy查询有11个主要部分：

| 属性             | 描述                                                         | 需要？ |
| ---------------- | ------------------------------------------------------------ | ------ |
| 查询类型         | 该String应始终为“groupBy”; 这是德鲁伊首先想弄清楚如何解释查询 | 是     |
| 数据源           | 定义要查询的数据源的String或Object，非常类似于关系数据库中的表。有关更多信息，请参阅[DataSource](http://druid.io/docs/0.12.3/querying/datasource.html)。 | 是     |
| 尺寸             | 要进行groupBy的JSON维度列表; 或者参见[DimensionSpec](http://druid.io/docs/0.12.3/querying/dimensionspecs.html)了解提取尺寸的方法。 | 是     |
| limitSpec        | 请参阅[LimitSpec](http://druid.io/docs/0.12.3/querying/limitspec.html)。 | 没有   |
| 有               | 见[有](http://druid.io/docs/0.12.3/querying/having.html)。   | 没有   |
| 粒度             | 定义查询的粒度。见[粒度](http://druid.io/docs/0.12.3/querying/granularities.html) | 是     |
| 过滤             | 见[滤波器](http://druid.io/docs/0.12.3/querying/filters.html) | 没有   |
| 聚合             | 请参阅[聚合](http://druid.io/docs/0.12.3/querying/aggregations.html) | 没有   |
| postAggregations | 请参阅[后聚合](http://druid.io/docs/0.12.3/querying/post-aggregations.html) | 没有   |
| 间隔             | 表示ISO-8601间隔的JSON对象。这定义了运行查询的时间范围。     | 是     |
| 上下文           | 一个额外的JSON对象，可用于指定某些标志。                     | 没有   |

要将它们全部拉到一起，上面的查询将返回*n \* m个*数据点，最多为5000个点，其中n是`country`维度的基数，m是维度的基数`device`，每天在2012-01-01之间和2012-01-03，从`sample_datasource`表中。每个数据点包含数据点`total_usage`的值是否大于100 的（长）总和，（双）总和`data_transfer`和（双）结果`total_usage`除以`data_transfer`特定分组的`country`和的过滤器组`device`。输出如下：

```json
[ 
  {
    "version" : "v1",
    "timestamp" : "2012-01-01T00:00:00.000Z",
    "event" : {
      "country" : <some_dim_value_one>,
      "device" : <some_dim_value_two>,
      "total_usage" : <some_value_one>,
      "data_transfer" :<some_value_two>,
      "avg_usage" : <some_avg_usage_value>
    }
  }, 
  {
    "version" : "v1",
    "timestamp" : "2012-01-01T00:00:12.000Z",
    "event" : {
      "dim1" : <some_other_dim_value_one>,
      "dim2" : <some_other_dim_value_two>,
      "sample_name1" : <some_other_value_one>,
      "sample_name2" :<some_other_value_two>,
      "avg_usage" : <some_other_avg_usage_value>
    }
  },
...
]
```

### 多值维度上的行为

groupBy查询可以对多值维度进行分组。在对多值维度进行分组时，匹配行中的*所有*值将用于为每个值生成一个组。查询可以返回比行更多的组。例如，在尺寸的GROUPBY `tags`与过滤器`"t1" AND "t3"`将匹配仅ROW1，并与三个组生成结果：`t1`，`t2`，和`t3`。如果您只需要包含与过滤器匹配的值，则可以使用[过滤的dimensionSpec](http://druid.io/docs/0.12.3/querying/dimensionspecs.html#filtered-dimensionspecs)。这也可以提高性能。

有关详细信息，请参阅[多值维度](http://druid.io/docs/0.12.3/querying/multi-value-dimensions.html)。

### 实施细节

#### 策略

可以使用两种不同的策略执行GroupBy查询。集群的默认策略由代理上的“druid.query.groupBy.defaultStrategy”运行时属性确定。这可以在查询上下文中使用“groupByStrategy”覆盖。如果既未设置上下文字段也未设置属性，则将使用“v2”策略。

- “v2”是默认设计，旨在提供更好的性能和内存管理。此策略使用完全堆外映射生成每段结果。数据节点使用完全堆外并发事实映射与堆上字符串字典相结合来合并每段结果。这可能可选地涉及溢出到磁盘。数据节点将排序结果返回给代理，代理使用N路合并合并结果流。如有必要，代理会实现结果（例如，如果查询对除其维度以外的列进行排序）。否则，它会在合并时将结果传回。
- “v1”是一个遗留引擎，它使用部分在堆上的映射（维度键和映射本身）和部分堆外（聚合值）在数据节点（历史，实时，中间管理器）上生成每段结果。然后，数据节点使用德鲁伊的索引机制合并每段结果。默认情况下，此合并是多线程的，但可以选择是单线程。经纪人再次使用德鲁伊的索引机制合并最终结果集。代理合并始终是单线程的。因为代理使用索引机制合并结果，所以它必须在返回任何结果之前实现完整的结果集。在数据节点和代理上，默认情况下合并索引完全在堆上，但它可以选择在堆外存储聚合值。

#### v1和v2之间的差异

查询API和结果在两个引擎之间是兼容的; 但是，从集群配置角度来看，存在一些差异：

- groupBy v1使用基于行的限制（maxResults）控制资源使用，而groupBy v2使用基于字节的限制。另外，groupBy v1在堆上合并结果，而groupBy v2在堆外合并结果。这些因素意味着内存调整和资源限制在v1和v2之间表现不同。特别是，由于这个原因，一些可以在一个引擎中成功完成的查询可能超出资源限制并且与另一个引擎失败。有关详细信息，请参阅“内存调整和资源限制”部分。
- groupBy v1对并发运行的查询数量没有限制，而groupBy v2通过使用有限大小的合并缓冲池来控制内存使用。默认情况下，合并缓冲区的数量是处理线程数的1/4。您可以根据需要调整此值以平衡并发和内存使用情况。
- groupBy v1支持代理或历史节点上的缓存，而groupBy v2仅支持历史节点上的缓存。
- groupBy v1支持使用[chunkPeriod](http://druid.io/docs/0.12.3/querying/query-context.html)并行化代理上的合并，而groupBy v2忽略chunkPeriod。
- groupBy v2支持基于阵列的聚合和基于散列的聚合。仅当分组键是单个索引字符串列时，才使用基于阵列的聚合。在基于数组的聚合中，字典编码的值用作索引，因此可以直接访问数组中的聚合值，而无需基于散列查找存储区。

#### 内存调整和资源限制

使用groupBy v2时，三个参数控制资源使用和限制：

- druid.processing.buffer.sizeBytes：每个查询用于聚合的堆外哈希表的大小，以字节为单位。至多druid.processing.numMergeBuffers将立即创建，这也作为并发运行的groupBy查询数量的上限。
- druid.query.groupBy.maxMergingDictionarySize：对字符串进行分组时使用的堆上字典的大小，每个查询，以字节为单位。请注意，这是基于字典大小的粗略估计，而不是实际大小。
- druid.query.groupBy.maxOnDiskStorage：每个查询用于聚合的磁盘空间量，以字节为单位。默认情况下，这是0，这意味着聚合将不使用磁盘。

如果maxOnDiskStorage为0（缺省值），则超出堆上字典限制或堆外聚合表限制的查询将失败，并显示“超出资源限制”错误，该错误描述了超出的限制。

如果maxOnDiskStorage大于0，则超出内存限制的查询将开始使用磁盘进行聚合。在这种情况下，当堆上字典或堆外哈希表填满时，部分聚合的记录将被排序并刷新到磁盘。然后，两个内存中的结构将被清除以进一步聚合。然后继续超过maxOnDiskStorage的查询将失败并显示“超出资源限制”错误，指示它们的磁盘空间不足。

对于groupBy v2，集群运算符应确保堆外哈希表和堆上合并字典不会超过可用内存以获得最大可能的并发查询负载（由druid.processing.numMergeBuffers提供）。看看[德鲁伊使用了多少直接记忆？](http://druid.io/docs/0.12.3/operations/performance-faq.html)更多细节。

使用groupBy v1时，所有聚合都在堆上完成，资源限制通过参数druid.query.groupBy.maxResults完成。这是结果集中最大结果数的上限。超出此限制的查询将失败，并显示“超出资源限制”错误，表明它们超出了行限制。集群运算符应确保堆上聚合不会超出预期并发查询负载的可用JVM堆空间。

#### groupBy v2的性能调整

##### 限制下推优化

德鲁伊将`limit`groupBy中的规范推向历史上的段，尽可能早期修剪不必要的中间结果，并尽量减少转移到经纪人的数据量。默认情况下，仅当`orderBy`规范中的所有字段都是分组键的子集时才应用此技术。这是因为`limitPushDown`如果`orderBy`规范包含不在分组键中的任何字段，则不能保证确切的结果。但是，即使在这种情况下，如果您可以牺牲一些准确性来快速查询处理（如topN查询），也可以启用此技术。见`forceLimitPushDown`在[先进GROUPBY V2配置](http://druid.io/docs/0.12.3/querying/groupbyquery.html#groupby-v2-configurations)。

##### 优化哈希表

groupBy v2引擎使用开放寻址哈希表进行聚合。哈希表初始化为给定的初始桶号，并在缓冲区满时逐渐增长。在散列冲突中，使用线性探测技术。

默认的初始桶数为1024，哈希表的默认最大加载因子为0.7。如果您可以在哈希表中看到太多冲突，则可以调整这些数字。见`bufferGrouperInitialBuckets`和`bufferGrouperMaxLoadFactor`在[高级GROUPBY V2配置](http://druid.io/docs/0.12.3/querying/groupbyquery.html#groupby-v2-configurations)。

##### 并行组合

一旦历史使用散列表完成聚合，它会对聚合进行排序并合并它们，然后再发送给代理以在代理中进行N路合并聚合。默认情况下，历史记录使用所有可用的处理线程（由配置`druid.processing.numThreads`）进行聚合，但使用单个线程进行排序和合并聚合，这是一个http线程，用于将数据发送给代理。

这是为了防止一些繁重的groupBy查询阻止其他查询。在Druid中，处理线程在所有提交的查询之间共享，并且它们*不可中断*。这意味着，如果繁重的查询占用所有可用的处理线程，则可能会阻止所有其他查询，直到重度查询完成。GroupBy查询通常比timeseries或topN查询花费更长的时间，他们应该尽快释放处理线程。

但是，您可能会关心某些非常繁重的groupBy查询的性能。通常，重组groupBy查询的性能瓶颈是合并已排序的聚合。在这种情况下，您也可以使用处理线程。这称为*并行组合*。要启用并行结合，看到`numParallelCombineThreads`在[高级GROUPBY V2配置](http://druid.io/docs/0.12.3/querying/groupbyquery.html#groupby-v2-configurations)。请注意，只有在数据实际溢出时才能启用并行组合（请参阅[内存调整和资源限制](http://druid.io/docs/0.12.3/querying/groupbyquery.html#memory-tuning-and-resource-limits)）。

启用并行组合后，groupBy v2引擎可以创建合并树以合并已排序的聚合。树的每个中间节点是合并来自子节点的聚合的线程。叶节点线程从包括溢出表的哈希表中读取和合并聚合。通常，叶节点比中间节点慢，因为它们需要从磁盘读取数据。因此，默认情况下，较少的线程用于中间节点。您可以更改intermeidate节点的程度。见`intermediateCombineDegree`在[高级GROUPBY V2配置](http://druid.io/docs/0.12.3/querying/groupbyquery.html#groupby-v2-configurations)。

#### 备择方案

在某些情况下，其他查询类型可能是比groupBy更好的选择。

- 对于没有“维度”的查询（即仅按时间分组），时间[序列查询](http://druid.io/docs/0.12.3/querying/timeseriesquery.html)通常比groupBy快。主要区别在于它以完全流式方式实现（利用了段已经按时排序的事实），并且不需要使用哈希表进行合并。
- 对于具有单个“维度”元素的查询（即按一个字符串维度分组），[TopN查询](http://druid.io/docs/0.12.3/querying/topnquery.html) 有时会比groupBy快。如果您按指标排序并找到可接受的近似结果，则尤其如此。

#### 嵌套的groupBys

对于“v1”和“v2”，嵌套的groupBys（类型为“query”的dataSource）的执行方式不同。代理首先以通常的方式运行内部groupBy查询。然后，“v1”策略使用德鲁伊的索引机制在堆上实现内部查询的结果，并对这些物化结果运行外部查询。“v2”策略使用可能溢出到磁盘的堆外事实映射和堆上字符串字典在内部查询的结果流上运行外部查询。两种策略都以单线程方式在代理上执行外部查询。

#### 配置

本节介绍groupBy查询的配置。您可以通过将它们添加到查询上下文中来将它们添加到运行时属性或特定于查询的配置中来设置系统范围的配置。所有运行时属性都以前缀为前缀`druid.query.groupBy`。

#### 通常调整的配置

##### groupBy v2的配置

支持的运行时属性

| 属性                                           | 描述                                                         | 默认      |
| ---------------------------------------------- | ------------------------------------------------------------ | --------- |
| `druid.query.groupBy.maxMergingDictionarySize` | 合并期间用于字符串字典的最大堆空间量（大约）。当字典超过此大小时，将触发溢出到磁盘。 | 亿        |
| `druid.query.groupBy.maxOnDiskStorage`         | 合并缓冲区或字典填满时，用于将结果集溢出到磁盘的最大磁盘空间量（每次查询）。超过此限制的查询将失败。设置为零以禁用磁盘溢出。 | 0（禁用） |

支持的查询上下文：

| 键                         | 描述                                                         |
| -------------------------- | ------------------------------------------------------------ |
| `maxMergingDictionarySize` | 可用于降低`druid.query.groupBy.maxMergingDictionarySize`此查询的值。 |
| `maxOnDiskStorage`         | 可用于降低`druid.query.groupBy.maxOnDiskStorage`此查询的值。 |

#### 高级配置

##### 所有groupBy策略的常见配置

支持的运行时属性

| 属性                                  | 描述                   | 默认 |
| ------------------------------------- | ---------------------- | ---- |
| `druid.query.groupBy.defaultStrategy` | 默认groupBy查询策略。  | V2   |
| `druid.query.groupBy.singleThreaded`  | 使用单个线程合并结果。 | 假   |

支持的查询上下文：

| 键                        | 描述                                                  |
| ------------------------- | ----------------------------------------------------- |
| `groupByStrategy`         | 覆盖`druid.query.groupBy.defaultStrategy`此查询的值。 |
| `groupByIsSingleThreaded` | 覆盖`druid.query.groupBy.singleThreaded`此查询的值。  |

##### GroupBy v2配置

支持的运行时属性

| 属性                                              | 描述                                                         | 默认      |
| ------------------------------------------------- | ------------------------------------------------------------ | --------- |
| `druid.query.groupBy.bufferGrouperInitialBuckets` | 用于分组结果的堆外哈希表中的初始桶数。设置为0以使用合理的默认值（1024）。 | 0         |
| `druid.query.groupBy.bufferGrouperMaxLoadFactor`  | 用于分组结果的堆外哈希表的最大加载因子。当负载系数超过此大小时，表将增长或溢出到磁盘。设置为0以使用合理的默认值（0.7）。 | 0         |
| `druid.query.groupBy.forceHashAggregation`        | 强制使用基于散列的聚合。                                     | 假        |
| `druid.query.groupBy.intermediateCombineDegree`   | 组合树中组合在一起的中间节点数。更高的度数将需要更少的线程，如果服务器具有足够强大的CPU核心，则可以通过减少太多线程的开销来帮助提高查询性能。 | 8         |
| `druid.query.groupBy.numParallelCombineThreads`   | 提示并行组合线程的数量。这应该大于1以打开并行组合功能。用于并行组合的实际线程数是min（`druid.query.groupBy.numParallelCombineThreads`，`druid.processing.numThreads`）。 | 1（禁用） |

支持的查询上下文：

| 键                            | 描述                                                         | 默认 |
| ----------------------------- | ------------------------------------------------------------ | ---- |
| `bufferGrouperInitialBuckets` | 覆盖`druid.query.groupBy.bufferGrouperInitialBuckets`此查询的值。 | 没有 |
| `bufferGrouperMaxLoadFactor`  | 覆盖`druid.query.groupBy.bufferGrouperMaxLoadFactor`此查询的值。 | 没有 |
| `forceHashAggregation`        | 覆盖的值`druid.query.groupBy.forceHashAggregation`           | 没有 |
| `intermediateCombineDegree`   | 覆盖的值`druid.query.groupBy.intermediateCombineDegree`      | 没有 |
| `numParallelCombineThreads`   | 覆盖的值`druid.query.groupBy.numParallelCombineThreads`      | 没有 |
| `sortByDimsFirst`             | 首先按维度值排序结果，然后按时间戳排序。                     | 假   |
| `forceLimitPushDown`          | 当orderby中的所有字段都是分组键的一部分时，代理会将限制应用程序下推到历史节点。当排序顺序使用不在分组键中的字段时，应用此优化可能会导致具有未知准确度的近似结果，因此在这种情况下默认情况下禁用此优化。启用此上下文标志会打开包含非分组键列的limit / orderbys的限制下推。 | 假   |

##### GroupBy v1配置

支持的运行时属性

| 属性                                      | 描述                                                         | 默认   |
| ----------------------------------------- | ------------------------------------------------------------ | ------ |
| `druid.query.groupBy.maxIntermediateRows` | 每段细分引擎的最大中间行数。这是一个不会施加硬限制的调整参数; 相反，它可能会将合并工作从每段引擎转移到整体合并索引。超过此限制的查询不会失败。 | 50000  |
| `druid.query.groupBy.maxResults`          | 最大结果数。超过此限制的查询将失败。                         | 500000 |

支持的查询上下文：

| 键                    | 描述                                                         | 默认 |
| --------------------- | ------------------------------------------------------------ | ---- |
| `maxIntermediateRows` | 可用于降低`druid.query.groupBy.maxIntermediateRows`此查询的值。 | 没有 |
| `maxResults`          | 可用于降低`druid.query.groupBy.maxResults`此查询的值。       | 没有 |
| `useOffheap`          | 设置为true以在合并结果时在堆外存储聚合。                     | 假   |