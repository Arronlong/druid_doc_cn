# 元数据存储

元数据存储是德鲁伊的外部依赖。Druid使用它来存储有关系统的各种元数据，但不存储实际数据。有许多表用于下面描述的各种目的。

Derby是Druid的默认元数据存储，但是它不适合生产。 [MySQL](http://druid.io/docs/0.12.3/development/extensions-core/mysql.html)和[PostgreSQL](http://druid.io/docs/0.12.3/development/extensions-core/postgresql.html)是更适合生产的元数据存储。

Derby不适合作为元数据存储的生产用途。请改用MySQL或PostgreSQL。

## 使用德比

将以下内容添加到您的德鲁伊配置中。

```properties
druid.metadata.storage.type=derby
druid.metadata.storage.connector.connectURI=jdbc:derby://localhost:1527//opt/var/druid_state/derby;create=true
```

## MySQL的

请参阅[mysql-metadata-storage扩展文档](http://druid.io/docs/0.12.3/development/extensions-core/mysql.html)。

## PostgreSQL的

请参阅[postgresql-metadata-storage](http://druid.io/docs/0.12.3/development/extensions-core/postgresql.html)。

## 元数据存储表

### 细分表

这取决于`druid.metadata.storage.tables.segments`酒店。

此表存储有关系统中可用的段的元数据。[协调](http://druid.io/docs/0.12.3/design/coordinator.html)器轮询该表以确定应该可用于在系统中查询的一组段。该表有两个主要功能列，其他列用于索引目的。

该`used`列是一个布尔“墓碑”。A 1表示该段应该由集群“使用”（即它应该被加载并可用于请求）。0表示不应将段主动加载到群集中。我们这样做是为了从群集中删除段而不实际删除其元数据（如果这是一个问题，则允许更简单的回滚）。

该`payload`列存储一个JSON blob，其中包含该段的所有元数据（存储在此有效负载中的一些数据是冗余的，表中的某些列是有意的）。这看起来像

```json
{
 "dataSource":"wikipedia",
 "interval":"2012-05-23T00:00:00.000Z/2012-05-24T00:00:00.000Z",
 "version":"2012-05-24T00:10:00.046Z",
 "loadSpec":{
    "type":"s3_zip",
    "bucket":"bucket_for_segment",
    "key":"path/to/segment/on/s3"
 },
 "dimensions":"comma-delimited-list-of-dimension-names",
 "metrics":"comma-delimited-list-of-metric-names",
 "shardSpec":{"type":"none"},
 "binaryVersion":9,
 "size":size_of_segment,
 "identifier":"wikipedia_2012-05-23T00:00:00.000Z_2012-05-24T00:00:00.000Z_2012-05-23T00:10:00.046Z"
}
```

请注意，此blob的格式可以并且将随时更改。

### 规则表

规则表用于存储关于段应该着陆的各种规则。[协调器](http://druid.io/docs/0.12.3/design/coordinator.html) 在制定有关群集的分段（重新）分配决策时使用这些规则。

### 配置表

配置表用于存储运行时配置对象。我们还没有很多这些，我们不确定是否会继续使用这种机制，但它是在运行时跨集群更改某些配置参数的方法的开始。

### 任务相关表

[索引服务](http://druid.io/docs/0.12.3/design/indexing-service.html)在其工作过程中还创建并使用了许多表。

### 审计表

Audit表用于存储配置更改的审计历史记录，例如[协调器](http://druid.io/docs/0.12.3/design/coordinator.html)完成的规则更改以及其他配置更改。

## 访问者：

元数据存储仅通过以下方式访问：

1. 索引服务节点（如果有）
2. 实时节点（如果有）
3. 协调员节点

因此，您只需为这些计算机授予访问元数据存储的权限（例如，在AWS安全组中）。