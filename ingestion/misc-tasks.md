# 其他任务

## 版本转换器任务

转换任务套件采用活动段，并使用新的IndexSpec重新压缩它们。这在执行从简化到咆哮的迁移或向旧段添加维度压缩等活动时非常方便。

成功后，新段将与`_converted`附加的旧段具有相同的版本。可以针对相同数据源多次针对相同间隔运行转换任务。每次执行都会将另一个执行添加`_converted`到段的版本中

有两种类型的转换任务。一个是Hadoop转换任务，另一个是索引服务转换任务。Hadoop转换任务在hadoop集群上运行，只需在索引服务上留下任务监视器（类似于hadoop批处理任务）。索引服务转换任务在索引服务上运行实际转换。

### Hadoop转换段任务

```json
{
  "type": "hadoop_convert_segment",
  "dataSource":"some_datasource",
  "interval":"2013/2015",
  "indexSpec":{"bitmap":{"type":"concise"},"dimensionCompression":"lz4","metricCompression":"lz4"},
  "force": true,
  "validate": false,
  "distributedSuccessCache":"hdfs://some-hdfs-nn:9000/user/jobrunner/cache",
  "jobPriority":"VERY_LOW",
  "segmentOutputPath":"s3n://somebucket/somekeyprefix"
}
```

值如下所述。

| 领域                      | 类型                                              | 描述                                                         | 需要                         |
| ------------------------- | ------------------------------------------------- | ------------------------------------------------------------ | ---------------------------- |
| `type`                    | 串                                                | 转换任务标识符                                               | 是：`hadoop_convert_segment` |
| `dataSource`              | 串                                                | 用于搜索段的数据源                                           | 是                           |
| `interval`                | 间隔字符串                                        | 数据源中用于查找段的间隔                                     | 是                           |
| `indexSpec`               | JSON                                              | 索引的压缩规范                                               | 是                           |
| `force`                   | 布尔                                              | 强制转换任务继续，即使二进制版本指示它最近已更新（您可能想要这样做） | 没有                         |
| `validate`                | 布尔                                              | 在报告任务成功之前，在旧段和新段之间运行验证                 | 没有                         |
| `distributedSuccessCache` | URI                                               | hadoop应放置中间文件的位置。                                 | 是                           |
| `jobPriority`             | `org.apache.hadoop.mapred.JobPriority` 作为字符串 | 为hadoop作业设置的优先级                                     | 没有                         |
| `segmentOutputPath`       | URI                                               | 要放置的段的基本uri。与其他位置相同的格式需要段输出路径      | 是                           |

### 索引服务转换段任务

```json
{
  "type": "convert_segment",
  "dataSource":"some_datasource",
  "interval":"2013/2015",
  "indexSpec":{"bitmap":{"type":"concise"},"dimensionCompression":"lz4","metricCompression":"lz4"},
  "force": true,
  "validate": false
}
```

| 领域         | 类型       | 描述                                                         | 必填（默认）           |
| ------------ | ---------- | ------------------------------------------------------------ | ---------------------- |
| `type`       | 串         | 转换任务标识符                                               | 是： `convert_segment` |
| `dataSource` | 串         | 用于搜索段的数据源                                           | 是                     |
| `interval`   | 间隔字符串 | 数据源中用于查找段的间隔                                     | 是                     |
| `indexSpec`  | JSON       | 索引的压缩规范                                               | 是                     |
| `force`      | 布尔       | 强制转换任务继续，即使二进制版本指示它最近已更新（您可能想要这样做） | 不（假）               |
| `validate`   | 布尔       | 在报告任务成功之前，在旧段和新段之间运行验证                 | 不（真）               |

与hadoop转换任务不同，索引服务任务从索引服务的配置中提取其输出路径。

#### IndexSpec

indexSpec定义了在索引时使用的段存储格式选项，例如位图类型和列压缩格式。indexSpec是可选的，如果未指定，将使用默认参数。

| 领域                 | 类型 | 描述                                                         | 需要                 |
| -------------------- | ---- | ------------------------------------------------------------ | -------------------- |
| 位图                 | 宾语 | 位图索引的压缩格式。应该是一个JSON对象; 请参阅下面的选项。   | 不（默认为简明）     |
| dimensionCompression | 串   | 维列的压缩格式。选择`LZ4`，`LZF`或`uncompressed`。           | 不（默认== `LZ4`）   |
| metricCompression    | 串   | 度量标准列的压缩格式。选择`LZ4`，`LZF`，`uncompressed`，或`none`。 | 不（默认== `LZ4`）   |
| longEncoding         | 串   | 类型为long的度量和维度列的编码格式。选择`auto`或`longs`。`auto`根据列基数使用偏移量或查找表对值进行编码，并以可变大小存储它们。`longs`将值存储为每个8字节。 | 不（默认== `longs`） |

##### 位图类型

对于简明位图：

| 领域 | 类型 | 描述              | 需要 |
| ---- | ---- | ----------------- | ---- |
| 类型 | 串   | 一定是`concise`。 | 是   |

对于咆哮的位图：

| 领域                       | 类型 | 描述                               | 需要                |
| -------------------------- | ---- | ---------------------------------- | ------------------- |
| 类型                       | 串   | 一定是`roaring`。                  | 是                  |
| compressRunOnSerialization | 布尔 | 使用行程编码，估计其空间效率更高。 | 不（默认== `true`） |

## Noop任务

这些任务开始，睡眠一段时间，仅用于测试。可用的语法是：

```json
{
    "type": "noop",
    "id": <optional_task_id>,
    "interval" : <optional_segment_interval>,
    "runTime" : <optional_millis_to_sleep>,
    "firehose": <optional_firehose_to_test_connect>
}
```

## 细分合并任务（已弃用）

### 附加任务

追加任务将一个段列表附加到一个段中（一个接一个）。语法是：

```json
{
    "type": "append",
    "id": <task_id>,
    "dataSource": <task_datasource>,
    "segments": <JSON list of DataSegment objects to append>,
    "aggregations": <optional list of aggregators>,
    "context": <task context>
}
```

### 合并任务

合并任务将段列表合并在一起。合并任何常见时间戳。如果在提取过程中禁用汇总，则不会合并公共时间戳，并且按时间戳对行进行重新排序。

语法是：

```json
{
    "type": "merge",
    "id": <task_id>,
    "dataSource": <task_datasource>,
    "aggregations": <list of aggregators>,
    "rollup": <whether or not to rollup data during a merge>,
    "segments": <JSON list of DataSegment objects to merge>,
    "context": <task context>
}
```

### 相同的间隔合并任务

相同间隔合并任务是合并任务的快捷方式，区间中的所有段都将合并。

语法是：

```json
{
    "type": "same_interval_merge",
    "id": <task_id>,
    "dataSource": <task_datasource>,
    "aggregations": <list of aggregators>,
    "rollup": <whether or not to rollup data during a merge>,
    "interval": <DataSegment objects in this interval are going to be merged>,
    "context": <task context>
}
```