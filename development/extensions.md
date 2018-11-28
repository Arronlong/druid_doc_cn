# 德鲁伊扩展

Druid实现了一个扩展系统，允许在运行时添加功能。扩展通常用于添加对深存储（如HDFS和S3），元数据存储（如MySQL和PostgreSQL），新聚合器，新输入格式等的支持。

生产集群通常至少使用两个扩展; 一个用于深度存储，一个用于元数据存储。许多群集也将使用其他扩展。

## 包括扩展

请看[这里](http://druid.io/docs/0.12.3/operations/including-extensions.html)。

## 核心扩展

核心扩展由Druid提交者维护。

| 名称                           | 描述                                                         | 文件                                                         |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 德鲁伊的Avro的扩展             | 支持Apache Avro数据格式的数据。                              | [链接](http://druid.io/docs/0.12.3/development/extensions-core/avro.html) |
| 德鲁伊的基本安全               | 支持基本HTTP身份验证和基于角色的访问控制。                   | [链接](http://druid.io/docs/0.12.3/development/extensions-core/druid-basic-security.html) |
| 德鲁伊咖啡因缓存               | 由Caffeine支持的本地缓存实现。                               | [链接](http://druid.io/docs/0.12.3/development/extensions-core/caffeine-cache.html) |
| 德鲁伊datasketches             | 使用[DataSketches](http://datasketches.github.io/)支持近似计数和设置操作。 | [链接](http://druid.io/docs/0.12.3/development/extensions-core/datasketches-aggregators.html) |
| 德鲁伊HDFS存储                 | HDFS深度存储。                                               | [链接](http://druid.io/docs/0.12.3/development/extensions-core/hdfs.html) |
| 德鲁伊直方图                   | 近似直方图和分位数聚合器。                                   | [链接](http://druid.io/docs/0.12.3/development/extensions-core/approximate-histograms.html) |
| 德鲁伊卡夫卡八                 | Kafka为实时节点摄取firehose（高级消费者）。                  | [链接](http://druid.io/docs/0.12.3/development/extensions-core/kafka-eight-firehose.html) |
| 德鲁伊卡夫卡 - 提取 - 命名空间 | 基于Kafka的命名空间查找。需要命名空间查找扩展。              | [链接](http://druid.io/docs/0.12.3/development/extensions-core/kafka-extraction-namespace.html) |
| 德鲁伊卡夫卡索引服务           | 完全监督一次Kafka摄取索引服务。                              | [链接](http://druid.io/docs/0.12.3/development/extensions-core/kafka-ingestion.html) |
| 德鲁伊的Kerberos               | 德鲁伊节点的Kerberos身份验证。                               | [链接](http://druid.io/docs/0.12.3/development/extensions-core/druid-kerberos.html) |
| 德鲁伊查找缓存全局             | 用于[查找](http://druid.io/docs/0.12.3/querying/lookups.html)的模块，为[查找](http://druid.io/docs/0.12.3/querying/lookups.html)提供jvm-global eager缓存。它提供了用于获取查找数据的JDBC和URI实现。 | [链接](http://druid.io/docs/0.12.3/development/extensions-core/lookups-cached-global.html) |
| 德鲁伊查找缓存单               | 每个查找缓存模块，以支持需要从全局查找池中隔离查找的用例     | [链接](http://druid.io/docs/0.12.3/development/extensions-core/druid-lookups.html) |
| 德鲁伊的protobuf的扩展         | 支持Protobuf数据格式的数据。                                 | [链接](http://druid.io/docs/0.12.3/development/extensions-core/protobuf.html) |
| 德鲁伊-S3的扩展                | 与AWS S3中的数据连接，并将S3用作深度存储。                   | [链接](http://druid.io/docs/0.12.3/development/extensions-core/s3.html) |
| 德鲁伊统计                     | 统计相关模块包括方差和标准差。                               | [链接](http://druid.io/docs/0.12.3/development/extensions-core/stats.html) |
| MySQL的元数据的存储            | MySQL元数据存储。                                            | [链接](http://druid.io/docs/0.12.3/development/extensions-core/mysql.html) |
| PostgreSQL的元数据的存储       | PostgreSQL元数据存储。                                       | [链接](http://druid.io/docs/0.12.3/development/extensions-core/postgresql.html) |
| 简单客户端的SSLContext         | 内部HttpClient通过HTTPS与其他节点通信的简单SSLContext提供程序模块。 | [链接](http://druid.io/docs/0.12.3/development/extensions-core/simple-client-sslcontext.html) |

# 社区扩展

虽然我们接受使用这些扩展的社区成员的补丁，但是社区扩展不是由德鲁伊提交者维护的。它们可能没有像核心扩展那样经过广泛测试。

许多社区成员为Druid贡献了自己的扩展，这些扩展没有使用默认的Druid tarball打包。如果您想为社区扩展进行维护，请发布在[德鲁伊开发小组上](https://groups.google.com/forum/#!forum/druid-development)以告知我们！

所有这些社区扩展都可以使用带有坐标io.druid.extensions.contrib：EXTENSION_NAME：LATEST_DRUID_STABLE_VERSION的*pull-deps*下载。

| 名称                         | 描述                                                         | 文件                                                         |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ambari度量极 - 发射极        | Ambari指标发射器                                             | [链接](http://druid.io/docs/0.12.3/development/extensions-contrib/ambari-metrics-emitter.html) |
| 德鲁伊蔚蓝的扩展             | Microsoft Azure深层存储。                                    | [链接](http://druid.io/docs/0.12.3/development/extensions-contrib/azure.html) |
| 德鲁伊卡桑德拉存储           | Apache Cassandra深度存储。                                   | [链接](http://druid.io/docs/0.12.3/development/extensions-contrib/cassandra.html) |
| 德鲁伊cloudfiles的扩展       | Rackspace Cloudfiles深度存储和firehose。                     | [链接](http://druid.io/docs/0.12.3/development/extensions-contrib/cloudfiles.html) |
| 德鲁伊distinctcount          | DistinctCount聚合器                                          | [链接](http://druid.io/docs/0.12.3/development/extensions-contrib/distinctcount.html) |
| 德鲁伊卡夫卡八simpleConsumer | 卡夫卡摄取firehose（低级消费者）。                           | [链接](http://druid.io/docs/0.12.3/development/extensions-contrib/kafka-simple.html) |
| 德鲁伊兽人的扩展             | 支持Apache Orc数据格式的数据。                               | [链接](http://druid.io/docs/0.12.3/development/extensions-contrib/orc.html) |
| 德鲁伊实木复合地板的扩展     | 支持Apache Parquet数据格式的数据。需要加载druid-avro-extensions。 | [链接](http://druid.io/docs/0.12.3/development/extensions-contrib/parquet.html) |
| 德鲁伊的RabbitMQ             | RabbitMQ firehose。                                          | [链接](http://druid.io/docs/0.12.3/development/extensions-contrib/rabbitmq.html) |
| 德鲁伊Redis的缓存            | 基于Redis的Druid缓存实现。                                   | [链接](http://druid.io/docs/0.12.3/development/extensions-contrib/redis-cache.html) |
| 德鲁伊rocketmq               | RocketMQ firehose。                                          | [链接](http://druid.io/docs/0.12.3/development/extensions-contrib/rocketmq.html) |
| 德 - 时间 - 最小 - 最大      | 时间戳的最小/最大聚合器。                                    | [链接](http://druid.io/docs/0.12.3/development/extensions-contrib/time-min-max.html) |
| 德鲁伊谷歌的扩展             | Google云端存储深层存储。                                     | [链接](http://druid.io/docs/0.12.3/development/extensions-contrib/google.html) |
| sqlserver的元数据的存储      | Microsoft SqlServer深层存储。                                | [链接](http://druid.io/docs/0.12.3/development/extensions-contrib/sqlserver.html) |
| 石墨发射器                   | Graphite metrics发射器                                       | [链接](http://druid.io/docs/0.12.3/development/extensions-contrib/graphite.html) |
| statsd发射极                 | StatsD指标发射器                                             | [链接](http://druid.io/docs/0.12.3/development/extensions-contrib/statsd.html) |
| 卡夫卡发射                   | 卡夫卡指标发射器                                             | [链接](http://druid.io/docs/0.12.3/development/extensions-contrib/kafka-emitter.html) |
| 德鲁伊节俭的扩展             | 支持节俭摄取                                                 | [链接](http://druid.io/docs/0.12.3/development/extensions-contrib/thrift.html) |

## 促进社区扩展到核心扩展

如果您希望将扩展程序升级为核心，请[告诉我们](https://groups.google.com/forum/#!forum/druid-development)。如果我们看到社区积极支持的社区扩展，我们可以根据社区反馈将其推广到核心。

# 创建自己的扩展

有关如何创建自己的扩展程序的信息，请参阅[此处](http://druid.io/docs/0.12.3/development/modules.html)。