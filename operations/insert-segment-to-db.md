# insert-segment-to-db工具

`insert-segment-to-db`是一个可以将段插入Druid元数据存储的工具。在人们将段从一个地方手动迁移到另一个地方之后，它旨在用于更新元数据存储中的段表。它还可以用于将缺失的段插入到Druid中，甚至可以通过告诉它存储段的位置来恢复元数据存储。

**注意：**此工具只扫描深层存储目录，以重建用于定位和标识每个段的元数据条目。它对这些细分是否*应该实际*没有任何了解被写入元数据存储。在某些情况下，这可能导致不希望的或不一致的结果。需要注意的一些示例： - 将重新启用丢弃的数据源。 - 每个分段集的最新版本将由Druid加载，在某些情况下可能不是您真正想要的版本。一个例子是一个糟糕的压缩作业，它生成需要通过从元数据表中删除该版本来手动回滚的段。如果这些段也没有从深层存储中删除，它们将被导回到元数据表中并掩盖正确的版本。 - 某些索引器（如Kafka索引服务）可能会生成多个具有相同段ID但内容不同的段。首次编写元数据时 引用正确的段集，通常从深层存储中删除另一组。但是，未处理的异常可能导致具有相同段ID的多组段保留在深度存储中。由于此工具不知道哪一个是“正确的”使用者，因此只需选择最新的段集并忽略其他版本。如果选择了错误的段集，那么Kafka索引服务的一次性语义将不再成立，您可能会获得重复或丢弃的事件。由于此工具不知道哪一个是“正确的”使用者，因此只需选择最新的段集并忽略其他版本。如果选择了错误的段集，那么Kafka索引服务的一次性语义将不再成立，您可能会获得重复或丢弃的事件。由于此工具不知道哪一个是“正确的”使用者，因此只需选择最新的段集并忽略其他版本。如果选择了错误的段集，那么Kafka索引服务的一次性语义将不再成立，您可能会获得重复或丢弃的事件。

考虑到这些因素，建议通过直接导出原始元数据存储来完成数据迁移，因为这是最终的群集状态。当无法直接导出时，此工具应作为最后的手段。

**注意：**此工具希望用户让德鲁伊群集以“安全”模式运行，其中没有活动任务干扰正在插入的段。用户可以选择关闭群集，100％确保没有任何干扰。

为了使其工作，用户必须通过Java JVM参数或runtime.properties文件提供元数据存储凭证和深度存储类型。具体来说，这个工具需要知道：

```text
druid.metadata.storage.type
druid.metadata.storage.connector.connectURI
druid.metadata.storage.connector.user
druid.metadata.storage.connector.password
druid.storage.type
```

除上述属性外，还需要指定存储段的位置以及是否要更新descriptor.json（`partitionNum_descriptor.json`用于HDFS数据存储）。这两个可以通过命令行参数提供。

`--workingDir` （需要）

```text
The directory URI where segments are stored. This tool will recursively look for segments underneath this directory
and insert/update these segments in metdata storage.
Attention: workingDir must be a complete URI, which means it must be prefixed with scheme type. For example,
hdfs://hostname:port/segment_directory
```

`--updateDescriptor` （可选的）

```text
if set to true, this tool will update `loadSpec` field in `descriptor.json` (`partitionNum_descriptor.json` for HDFS data storage) if the path in `loadSpec` is different from
where `desciptor.json` (`partitionNum_descriptor.json` for HDFS data storage) was found. Default value is `true`.
```

注意：您还需要根据您使用的元数据和深层存储加载不同的Druid扩展。例如，如果您将`mysql`元数据存储和HDFS用作深层存储，则应加载`mysql-metadata-storage`和`druid-hdfs-storage` 扩展。

例：

假设您的元数据存储已经`mysql`并且您已将某些段迁移到HDFS中的目录，并且该目录如下所示，

```text
Directory path: /druid/storage/wikipedia

├── 2013-08-31T000000.000Z_2013-09-01T000000.000Z
│   └── 2015-10-21T22_07_57.074Z
│           ├── 0_descriptor.json
│           └── 0_index.zip
├── 2013-09-01T000000.000Z_2013-09-02T000000.000Z
│   └── 2015-10-21T22_07_57.074Z
│           ├── 0_descriptor.json
│           └── 0_index.zip
├── 2013-09-02T000000.000Z_2013-09-03T000000.000Z
│   └── 2015-10-21T22_07_57.074Z
│           ├── 0_descriptor.json
│           └── 0_index.zip
└── 2013-09-03T000000.000Z_2013-09-04T000000.000Z
    └── 2015-10-21T22_07_57.074Z
            ├── 0_descriptor.json
            └── 0_index.zip
```

要将所有这些段加载到`mysql`，可以触发下面的命令，

```text
java 
-Ddruid.metadata.storage.type=mysql 
-Ddruid.metadata.storage.connector.connectURI=jdbc\:mysql\://localhost\:3306/druid 
-Ddruid.metadata.storage.connector.user=druid 
-Ddruid.metadata.storage.connector.password=diurd 
-Ddruid.extensions.loadList=[\"mysql-metadata-storage\",\"druid-hdfs-storage\"] 
-Ddruid.storage.type=hdfs
-cp $DRUID_CLASSPATH 
io.druid.cli.Main tools insert-segment-to-db --workingDir hdfs://host:port//druid/storage/wikipedia --updateDescriptor true
```

在此示例中，`mysql`通过Java JVM参数提供深度存储类型，您可以选择将所有这些类型放在runtime.properites文件中，并将其包含在Druid类路径中。需要注意的是，我们也包括`mysql-metadata-storage` 和`druid-hdfs-storage`扩展名列表中。

运行此命令后，segments表中`mysql`应存储刚刚插入的每个段的新位置。请注意，对于存储在HDFS中的段，druid配置必须包含druid [Docs中](http://druid.io/docs/latest/tutorials/cluster.html)描述的core-site.xml ，因为此新位置存储了相对路径。

它也可以`s3`用作深度存储。要使用它，请指定`s3`为深度存储类型并将其[`druid-s3-extensions`](http://druid.io/docs/0.12.3/development/extensions-core/s3.html)作为扩展名加载 。

```text
java
-Ddruid.metadata.storage.type=mysql 
-Ddruid.metadata.storage.connector.connectURI=jdbc\:mysql\://localhost\:3306/druid 
-Ddruid.metadata.storage.connector.user=druid 
-Ddruid.metadata.storage.connector.password=diurd
-Ddruid.extensions.loadList=[\"mysql-metadata-storage\",\"druid-s3-extensions\"]
-Ddruid.storage.type=s3
-Ddruid.s3.accessKey=... 
-Ddruid.s3.secretKey=...
-Ddruid.storage.bucket=your-bucket
-Ddruid.storage.baseKey=druid/storage/wikipedia
-Ddruid.storage.maxListingLength=1000
-cp $DRUID_CLASSPATH
io.druid.cli.Main tools insert-segment-to-db --workingDir "druid/storage/wikipedia" --updateDescriptor true
```

请注意，您可以使用`druid.storage.baseKey`或提供段的位置`--workingDir`。如果两者都指定，则`--workingDir`获得更高的优先级。`druid.storage.maxListingLength`用于确定请求对象列表的部分列表的长度`s3`，默认为1000。