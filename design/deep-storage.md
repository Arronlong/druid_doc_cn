# 深度存储

深度存储是存储段的位置。它是德鲁伊不提供的存储机制。这种深度存储基础架构定义了数据的持久性级别，只要德鲁伊节点可以看到这个存储基础架构并获得存储在其上的段，无论丢失多少个德鲁伊节点，都不会丢失数据。如果片段从此存储层中消失，那么您将丢失这些片段所代表的任何数据。

## 本地山

本地安装也可用于存储段。这允许您仅使用本地文件系统或可以在本地挂载的任何其他东西，如NFS，Ceph等。这是默认的深层存储实现。

要使用本地安装进行深度存储，您需要在常用配置中设置以下配置。

| 属性                             | 可能的值 | 描述           | 默认       |
| -------------------------------- | -------- | -------------- | ---------- |
| `druid.storage.type`             | 本地     |                | 必须设置。 |
| `druid.storage.storageDirectory` |          | 存储段的目录。 | 必须设置。 |

请注意，您通常应设置`druid.storage.storageDirectory`为与`druid.segmentCache.locations`and 不同的内容`druid.segmentCache.infoDir`。

如果您在本地模式下使用Hadoop索引器，那么只需将它作为输出目录给它一个本地文件即可。

## S3兼容

请参阅[druid-s3-extensions扩展文档](http://druid.io/docs/0.12.3/development/extensions-core/s3.html)。

## HDFS

请参阅[druid-hdfs-storage扩展文档](http://druid.io/docs/0.12.3/development/extensions-core/hdfs.html)。

## 额外的深店

如需更多深度商店，请参阅我们的[扩展列表](http://druid.io/docs/0.12.3/development/extensions.html)。