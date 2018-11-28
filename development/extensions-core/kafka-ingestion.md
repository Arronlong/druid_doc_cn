# 卡夫卡索引服务

Kafka索引服务可以在Overlord上配置*主管*，通过管理Kafka索引任务的创建和生命周期来促进Kafka的摄取。这些索引任务使用Kafka自己的分区和偏移机制读取事件，因此能够提供完全一次摄取的保证。他们还能够从Kafka读取非近期事件，并且不受其他摄取机制强加的窗口期限的影响。主管监督索引任务的状态，以协调切换，管理故障并确保维护可伸缩性和复制要求。

此服务在`druid-kafka-indexing-service`核心扩展中提供（请参阅 [包括扩展](http://druid.io/docs/0.12.3/operations/including-extensions.html)）。请注意，Kafka索引服务目前被指定为*实验性功能*，并受到通常的[实验性警告的](http://druid.io/docs/0.12.3/development/experimental.html)约束 。

Kafka索引服务使用Kafka 0.10.x中引入的Java使用者。由于此版本中存在协议更改，因此Kafka 0.10.x消费者可能与较旧的代理不兼容。在使用此服务之前，请确保您的Kafka代理是0.10.x或更高版本。如果您使用的是旧版本的kafka经纪人，请参阅[Kafka升级指南](https://kafka.apache.org/documentation/#upgrade)。

## 提交主管规格

Kafka索引服务要求将`druid-kafka-indexing-service`扩展加载到霸主和中间管理者身上。通过HTTP POST提交管理程序规范来启动dataSource的主管 `http://<OVERLORD_IP>:<OVERLORD_PORT>/druid/indexer/v1/supervisor`，例如：

```text
curl -X POST -H 'Content-Type: application/json' -d @supervisor-spec.json http://localhost:8090/druid/indexer/v1/supervisor
```

示例主管规范如下所示：

```json
{
  "type": "kafka",
  "dataSchema": {
    "dataSource": "metrics-kafka",
    "parser": {
      "type": "string",
      "parseSpec": {
        "format": "json",
        "timestampSpec": {
          "column": "timestamp",
          "format": "auto"
        },
        "dimensionsSpec": {
          "dimensions": [],
          "dimensionExclusions": [
            "timestamp",
            "value"
          ]
        }
      }
    },
    "metricsSpec": [
      {
        "name": "count",
        "type": "count"
      },
      {
        "name": "value_sum",
        "fieldName": "value",
        "type": "doubleSum"
      },
      {
        "name": "value_min",
        "fieldName": "value",
        "type": "doubleMin"
      },
      {
        "name": "value_max",
        "fieldName": "value",
        "type": "doubleMax"
      }
    ],
    "granularitySpec": {
      "type": "uniform",
      "segmentGranularity": "HOUR",
      "queryGranularity": "NONE"
    }
  },
  "tuningConfig": {
    "type": "kafka",
    "maxRowsPerSegment": 5000000
  },
  "ioConfig": {
    "topic": "metrics",
    "consumerProperties": {
      "bootstrap.servers": "localhost:9092"
    },
    "taskCount": 1,
    "replicas": 1,
    "taskDuration": "PT1H"
  }
}
```

## 主管配置

| 领域           | 描述                                                         | 需要 |
| -------------- | ------------------------------------------------------------ | ---- |
| `type`         | 主管类型，这应该是`kafka`。                                  | 是   |
| `dataSchema`   | 摄取过程中Kafka索引任务将使用的模式，请参阅[Ingestion Spec DataSchema](http://druid.io/docs/0.12.3/ingestion/ingestion-spec.html#dataschema)。 | 是   |
| `tuningConfig` | 用于配置管理程序和索引任务的KafkaSupervisorTuningConfig，请参见下文。 | 没有 |
| `ioConfig`     | 用于配置管理程序和索引任务的KafkaSupervisorIOConfig，请参见下文。 | 是   |

### KafkaSupervisorTuningConfig

tuningConfig是可选的，如果未指定tuningConfig，则将使用默认参数。

| 领域                           | 类型        | 描述                                                         | 需要                                                         |
| ------------------------------ | ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `type`                         | 串          | 索引任务类型，这应该始终是`kafka`。                          | 是                                                           |
| `maxRowsInMemory`              | 整数        | 持久化之前要聚合的行数。此数字是后聚合行，因此它不等于输入事件的数量，而是等于这些事件导致的聚合行数。这用于管理所需的JVM堆大小。索引的最大堆内存使用量使用maxRowsInMemory *（2 + maxPendingPersists）进行缩放。 | 不（默认== 75000）                                           |
| `maxRowsPerSegment`            | 整数        | 要聚合到段中的行数; 这个数字是后聚合行。                     | 不（默认== 5000000）                                         |
| `intermediatePersistPeriod`    | ISO8601期间 | 决定中间体持续发生率的时期。                                 | 不（默认== PT10M）                                           |
| `maxPendingPersists`           | 整数        | 可以挂起但未启动的最大持久数。如果新的中间持久性将超过此限制，则摄取将阻塞，直到当前运行的持久性结束。索引的最大堆内存使用量使用maxRowsInMemory *（2 + maxPendingPersists）进行缩放。 | no（默认== 0，表示一个persist可以与摄取同时运行，并且没有一个可以排队） |
| `indexSpec`                    | 宾语        | 调整数据的索引方式，有关详细信息，请参阅下面的“IndexSpec”。  | 没有                                                         |
| `reportParseExceptions`        | 布尔        | 如果为true，则将抛出解析期间遇到的异常并停止摄取; 如果为false，将跳过不可解析的行和字段。 | 不（默认==假）                                               |
| `handoffConditionTimeout`      | 长          | 等待段切换的毫秒数。它必须> = 0，其中0表示永远等待。         | 不（默认== 0）                                               |
| `resetOffsetAutomatically`     | 布尔        | 是否在尝试获取的下一个偏移量小于该特定分区的最早可用偏移量时重置消费者偏移量。根据（见下文）的`useEarliestOffset`属性，消费者偏移将重置为最早或最近的偏移`KafkaSupervisorIOConfig`。这种情况通常发生在卡夫卡的消息不再可供消费时，因此不会被摄入德鲁伊。如果设置为false，则将暂停对该特定分区的摄取，并且需要手动干预来纠正这种情况，请参阅`Reset Supervisor`下面的API。 | 不（默认==假）                                               |
| `workerThreads`                | 整数        | 管理程序将用于异步操作的线程数。                             | no（默认== min（10，taskCount））                            |
| `chatThreads`                  | 整数        | 将用于与索引任务通信的线程数。                               | no（默认== min（10，taskCount * replicas））                 |
| `chatRetries`                  | 整数        | 在考虑任务无响应之前，将重试HTTP请求索引任务的次数。         | 不（默认== 8）                                               |
| `httpTimeout`                  | ISO8601期间 | 等待索引任务的HTTP响应需要多长时间。                         | 不（默认== PT10S）                                           |
| `shutdownTimeout`              | ISO8601期间 | 等待主管在退出之前尝试正常关闭任务需要多长时间。             | 否（默认== PT80S）                                           |
| `offsetFetchPeriod`            | ISO8601期间 | 主管查询Kafka和索引任务以获取当前偏移量和计算延迟的频率。    | 否（默认== PT30S，分钟== PT5S）                              |
| `segmentWriteOutMediumFactory` | 串          | 分段写出介质，用于创建分段。有关说明和可用选项，请参阅[其他Peon配置：SegmentWriteOutMediumFactory](http://druid.io/docs/0.12.3/configuration/index.html#segmentwriteoutmediumfactory)。 | no（默认情况下未指定，使用的值`druid.peon.defaultSegmentWriteOutMediumFactory`） |

#### IndexSpec

| 领域                 | 类型 | 描述                                                         | 需要                 |
| -------------------- | ---- | ------------------------------------------------------------ | -------------------- |
| 位图                 | 宾语 | 位图索引的压缩格式。应该是一个JSON对象; 请参阅下面的选项。   | 不（默认为简明）     |
| dimensionCompression | 串   | 维列的压缩格式。选择`LZ4`，`LZF`或`uncompressed`。           | 不（默认== `LZ4`）   |
| metricCompression    | 串   | 度量标准列的压缩格式。选择`LZ4`，`LZF`，`uncompressed`，或`none`。 | 不（默认== `LZ4`）   |
| longEncoding         | 串   | 类型为long的度量和维度列的编码格式。选择`auto`或`longs`。`auto`根据列基数使用偏移量或查找表对值进行编码，并以可变大小存储它们。`longs`将值存储为每个8字节。 | 不（默认== `longs`） |

##### 位图类型

对于简明位图：

| 领域   | 类型 | 描述              | 需要 |
| ------ | ---- | ----------------- | ---- |
| `type` | 串   | 一定是`concise`。 | 是   |

对于咆哮的位图：

| 领域                         | 类型 | 描述                               | 需要                |
| ---------------------------- | ---- | ---------------------------------- | ------------------- |
| `type`                       | 串   | 一定是`roaring`。                  | 是                  |
| `compressRunOnSerialization` | 布尔 | 使用行程编码，估计其空间效率更高。 | 不（默认== `true`） |

### KafkaSupervisorIOConfig

| 领域                          | 类型        | 描述                                                         | 需要               |
| ----------------------------- | ----------- | ------------------------------------------------------------ | ------------------ |
| `topic`                       | 串          | 卡夫卡主题可供阅读。这必须是特定主题，因为不支持主题模式。   | 是                 |
| `consumerProperties`          | 地图        | 要传递给Kafka使用者的属性映射。这必须包含一个属性`bootstrap.servers`，其中包含以下形式的Kafka代理列表：`<BROKER_1>:<PORT_1>,<BROKER_2>:<PORT_2>,...`。 | 是                 |
| `replicas`                    | 整数        | 副本集的数量，其中1表示一组任务（无复制）。副本任务将始终分配给不同的工作程序，以提供针对节点故障的弹性。 | 不（默认== 1）     |
| `taskCount`                   | 整数        | *副本集中*的最大*读取*任务数。这意味着最大的阅读任务数量和任务总数（*阅读* + *发布*）将高于此。有关详细信息，请参阅下面的“容量规划”。阅读任务的数量将少于if 。`taskCount * replicas``taskCount``taskCount > {numKafkaPartitions}` | 不（默认== 1）     |
| `taskDuration`                | ISO8601期间 | 任务停止阅读并开始发布其细分之前的时间长度。请注意，当索引任务完成时，段仅被推送到深层存储并由历史节点加载。 | 不（默认== PT1H）  |
| `startDelay`                  | ISO8601期间 | 在主管开始管理任务之前等待的时间段。                         | 不（默认== PT5S）  |
| `period`                      | ISO8601期间 | 主管多久会执行一次管理逻辑。请注意，管理程序也将运行以响应某些事件（例如任务成功，失败并到达其tasksDuration），因此该值指定迭代之间的最长时间。 | 否（默认== PT30S） |
| `useEarliestOffset`           | 布尔        | 如果主管第一次管理dataSource，它将从Kafka获得一组起始偏移量。此标志确定它是否检索Kafka中的最早或最新偏移。在正常情况下，后续任务将从先前段结束的位置开始，因此该标志仅在首次运行时使用。 | 不（默认==假）     |
| `completionTimeout`           | ISO8601期间 | 在将发布任务声明为失败并终止之前等待的时间长度。如果设置得太低，您的任务可能永远不会发布。任务的发布时间大致在`taskDuration`过去后开始。 | 不（默认== PT30M） |
| `lateMessageRejectionPeriod`  | ISO8601期间 | 配置任务以拒绝时间戳早于创建任务之前的时间段的消息;例如，如果将此设置为`PT1H`并且主管在*2016-01-01T12：00Z*创建任务，则将删除时间戳早于*2016-01-01T11：00Z的*消息。如果您的数据流具有延迟消息并且您有多个需要在相同段上运行的管道（例如，实时和夜间批量提取管道），这可能有助于防止并发问题。 | 不（默认==无）     |
| `earlyMessageRejectionPeriod` | ISO8601期间 | 配置任务以在任务到达taskDuration后拒绝时间戳晚于此时间段的消息; 例如，如果设置为`PT1H`，则taskDuration设置为`PT1H`，并且主管在*2016-01-01T12：00Z*创建任务，将删除时间戳晚于*2016-01-01T14：00Z的*消息。 | 不（默认==无）     |
| `skipOffsetGaps`              | 布尔        | 是否允许Kafka流中缺失偏移的间隙。这与诸如MapR Streams之类的实现兼容是必需的，这些实现不保证连续的偏移。如果为false，则如果偏移不连续，则抛出异常。 | 不（默认==假）     |

## 主管API

霸主可以使用以下端点：

#### 创建主管

```text
POST /druid/indexer/v1/supervisor
```

`Content-Type: application/json`在请求正文中使用并提供主管规范。

当已存在同一dataSource的主管时调用此端点将导致：

- 正在运行的主管用信号通知其托管任务停止阅读并开始发布。
- 正在运行的主管退出。
- 使用请求正文中提供的配置创建的新主管。该主管将保留现有的发布任务，并将在发布任务结束的偏移处开始创建新任务。

因此，只需使用此端点提交新模式即可实现无缝模式迁移。

#### 关机主管

```text
POST /druid/indexer/v1/supervisor/<supervisorId>/shutdown
```

请注意，这将导致此主管管理的所有索引任务立即停止并开始发布其段。

#### 获取主管ID

```text
GET /druid/indexer/v1/supervisor
```

返回当前活动的主管的列表。

#### 获得Supervisor Spec

```text
GET /druid/indexer/v1/supervisor/<supervisorId>
```

使用提供的ID返回主管的当前规范。

#### 获取主管状态报告

```text
GET /druid/indexer/v1/supervisor/<supervisorId>/status
```

返回给定主管管理的任务的当前状态的快照报告。这包括Kafka报告的最新偏移量，每个分区的消费者滞后以及所有分区的总滞后。如果主管没有从Kafka收到最近的最新偏移响应，则每个分区的消费者滞后可以报告为负值。总滞后值始终> = 0。

#### 获取所有主管历史记录

```text
GET /druid/indexer/v1/supervisor/history
```

返回所有主管（当前和过去）的规范的审计历史记录。

#### 获得主管历史

```text
GET /druid/indexer/v1/supervisor/<supervisorId>/history
```

使用提供的ID返回主管规范的审核历史记录。

#### 重置主管

```text
POST /druid/indexer/v1/supervisor/<supervisorId>/reset
```

索引服务会跟踪最新的持久性Kafka偏移量，以便在任务中提供一次完整的摄取保证。后续任务必须从上一个任务完成的位置开始读取，以便接受生成的段。如果Kafka中不再提供预期起始偏移处的消息（通常是因为消息保留期已过或主题已被删除并重新创建），则主管将拒绝启动并且正在进行的任务将失败。

此端点可用于清除存储的偏移量，这将导致管理员从Kafka中的最早或最近偏移量开始读取（取决于值`useEarliestOffset`）。必须运行主管才能使此端点可用。清除存储的偏移后，主管将自动终止并重新创建任何活动任务，以便任务开始从有效偏移读取。

请注意，由于存储的偏移量是保证精确一次摄取所必需的，因此使用此端点重置它们可能会导致某些Kafka消息被跳过或被读取两次。

## 容量规划

Kafka索引任务在中层管理器上运行，因此受到中间管理器集群中可用资源的限制。特别是，您应确保具有足够的工作容量（使用`druid.worker.capacity`属性配置 ）来处理管理程序规范中的配置。请注意，工作者容量在所有类型的索引任务中共享，因此您应该规划工作容量以处理总索引负载（例如批处理，实时任务，合并任务等）。如果您的工作人员用完了容量，Kafka索引任务将排队等待下一个可用的工作人员。这可能导致查询返回部分结果但不会导致数据丢失（假设任务在Kafka清除这些偏移之前运行）。

正在运行的任务通常处于以下两种状态之一：*阅读*或*发布*。任务将保持读取状态 `taskDuration`，此时它将转换为发布状态。只要生成段，将段推送到深层存储，并由历史节点加载和提供（或直到`completionTimeout`过去），任务将保持在发布状态。

读取任务的数量由`replicas`和控制`taskCount`。通常，会有`replicas * taskCount` 读取任务，例外情况是taskCount> {numKafkaPartitions}，在这种情况下将使用{numKafkaPartitions}任务。当`taskDuration`流逝，这些任务将转变为公布情况和`replicas * taskCount` 新的阅读任务将被创建。因此，为了允许读取任务和发布任务同时运行，应该具有以下最小容量：

```text
workerCapacity = 2 * replicas * taskCount
```

此值适用于理想情况，其中最多有一组任务发布而另一组正在读取。在某些情况下，可以同时发布多组任务。如果发布时间（生成段，推送到深层存储，加载历史）>，则会发生这种情况`taskDuration`。这是一个有效的方案（正确性），但需要额外的工作人员支持。一般来说，最好 `taskDuration`足够大，以使前一组任务在当前集合开始之前完成发布。

## 主管坚持

当通过`POST /druid/indexer/v1/supervisor`端点提交管理程序规范时，它将保留在已配置的元数据数据库中。每个dataSource只能有一个supervisor，并且为同一个dataSource提交第二个规范将覆盖前一个。

当一个霸主获得领导时，无论是通过启动还是由于另一个霸主失败，它将为元数据库中的每个主管规范产生一个主管。然后，主管将发现正在运行的Kafka索引任务，并且如果它们与主管的配置兼容，将尝试采用它们。如果它们不兼容，因为它们具有不同的摄取规范或分区分配，则任务将被终止，并且主管将创建一组新任务。通过这种方式，主管可以持续进行霸主重启和故障转移。

主管通过`POST /druid/indexer/v1/supervisor/<supervisorId>/shutdown`端点停止。这会在数据库中放置一个逻辑删除标记（以防止主管在重新启动时重新加载），然后正常关闭当前运行的主管。当主管以这种方式关闭时，它将指示其托管任务停止阅读并立即开始发布其分段。在所有任务都已发出信号停止但在任务完成发布其分段之前，对关闭端点的调用将返回。

## 架构/配置更改

通过`POST /druid/indexer/v1/supervisor`用于最初创建主管的相同端点提交新的主管规范来处理模式和配置更改 。霸主将启动现有主管的正常关闭，这将导致该主管管理的任务停止阅读并开始发布其分段。然后将启动一个新的管理程序，它将创建一组新的任务，这些任务将从之前现在发布的任务停止的偏移开始读取，但使用更新的模式。通过这种方式，可以应用配置更改，而无需任何摄取暂停。

## 部署说明

### 论细分主体

每个Kafka索引任务都会将分配给它的Kafka分区所消耗的事件放在每个段粒度间隔的单个段中，直到达到maxRowsPerSegment限制，此时将为其他事件创建此段粒度的新分区。Kafka Indexing Task还执行增量切换，这意味着在任务持续时间结束之前，任务创建的所有段都不会被保留。一旦达到maxRowsPerSegment限制，该任务在该时间点保留的所有段将被切换，并且将为其他事件创建新的段组。这意味着任务可以运行更长的持续时间，而不会在Middle Manager节点上本地累积旧段，并鼓励它这样做。

Kafka Indexing Service仍可能会产生一些小部分。假设任务持续时间为4小时，分段粒度设置为HOUR，主管在9:10开始，然后在13:10开始4小时后，将启动新的任务集，并在13:00 - 14之间启动事件：00可以分为前一组和新任务组。如果您发现它成为一个问题，那么可以安排重新编制索引任务，将段合并为理想大小的新段（每段约500-700 MB）。还有正在进行的工作，以支持分片段的自动分段压缩以及不需要Hadoop的压缩（参见[此处](https://github.com/druid-io/druid/pull/5102)）。