# 聚类

Druid旨在部署为可扩展的容错集群。

在本文档中，我们将设置一个简单的集群，并讨论如何进一步配置以满足您的需求。这个简单的集群将为Historicals和MiddleManagers提供可扩展的容错服务器，以及一个用于托管Coordinator和Overlord进程的单一协调服务器。在生产中，我们建议在容错配置中部署协调器和宿主。

## 选择硬件

Coordinator和Overlord进程可以共存于一台服务器上，该服务器负责处理集群的元数据和协调需求。相当于AWS [m3.xlarge](https://aws.amazon.com/ec2/instance-types/#M3)对于大多数集群[来说](https://aws.amazon.com/ec2/instance-types/#M3)已经足够了。这个硬件提供：

- 4个vCPU
- 15 GB RAM
- 80 GB SSD存储

可以在单个服务器上共存历史记录和MiddleManagers，以处理群集中的实际数据。这些服务器从CPU，RAM和SSD中受益匪浅。相当于AWS [r3.2xlarge](https://aws.amazon.com/ec2/instance-types/#r3)是一个很好的起点。这个硬件提供：

- 8个vCPU
- 61 GB RAM
- 160 GB SSD存储

德鲁伊经纪人接受查询并将其分配给群集的其余部分。它们还可以选择维护内存中的查询缓存。这些服务器从CPU和RAM中受益匪浅，也可以部署在AWS [r3.2xlarge上](https://aws.amazon.com/ec2/instance-types/#r3)。这个硬件提供：

- 8个vCPU
- 61 GB RAM
- 160 GB SSD存储

您可以考虑在运行Broker的同一服务器上共同定位任何开源UI或查询库。

非常大的集群应考虑选择更大的服务器。

## 选择OS

我们建议运行您喜欢的Linux发行版。您还需要：

- Java 8

您的OS包管理器应该能够为两个Java提供帮助。如果您的基于Ubuntu的操作系统没有最新版本的Java，则WebUpd8会[为这些操作系统](http://www.webupd8.org/2012/09/install-oracle-java-8-in-ubuntu-via-ppa.html)提供[软件包](http://www.webupd8.org/2012/09/install-oracle-java-8-in-ubuntu-via-ppa.html)。

## 下载发行版

首先，下载并解压缩发布存档。最好先在一台机器上执行此操作，因为您将编辑配置，然后将修改后的分发复制到所有服务器。

```bash
curl -O http://static.druid.io/artifacts/releases/druid-0.12.3-bin.tar.gz
tar -xzf druid-0.12.3-bin.tar.gz
cd druid-0.12.3
```

在这个包中，你会发现：

- `LICENSE` - 许可证文件。
- `bin/`- 与[单机快速入门](http://druid.io/docs/0.12.3/tutorials/quickstart.html)相关的脚本。
- `conf/*` - 群集设置的模板配置。
- `conf-quickstart/*`- [单机快速入门的](http://druid.io/docs/0.12.3/tutorials/quickstart.html)配置。
- `extensions/*` - 所有德鲁伊扩展。
- `hadoop-dependencies/*` - 德鲁伊Hadoop依赖。
- `lib/*` - 所有包含核心德鲁伊的软件包。
- `quickstart/*`- 与[单机快速入门](http://druid.io/docs/0.12.3/tutorials/quickstart.html)相关的文件。

我们将编辑文件`conf/`以便运行。

## 配置深层存储

Druid依赖于分布式文件系统或大型对象（blob）存储来进行数据存储。最常用的深度存储实现是S3（适用于AWS上的那些）和HDFS（如果您已经有Hadoop部署，则很流行）。

### S3

在`conf/druid/_common/common.runtime.properties`，

- 设置`druid.extensions.loadList=["druid-s3-extensions"]`。
- 在“Deep Storage”和“索引服务日志”下注释掉本地存储的配置。
- 取消注释并在“Deep Storage”和“索引服务日志”的“For S3”部分中配置适当的值。

在此之后，您应该进行以下更改：

```text
druid.extensions.loadList=["druid-s3-extensions"]

#druid.storage.type=local
#druid.storage.storageDirectory=var/druid/segments

druid.storage.type=s3
druid.storage.bucket=your-bucket
druid.storage.baseKey=druid/segments
druid.s3.accessKey=...
druid.s3.secretKey=...

#druid.indexer.logs.type=file
#druid.indexer.logs.directory=var/druid/indexing-logs

druid.indexer.logs.type=s3
druid.indexer.logs.s3Bucket=your-bucket
druid.indexer.logs.s3Prefix=druid/indexing-logs
```

### HDFS

在`conf/druid/_common/common.runtime.properties`，

- 设置`druid.extensions.loadList=["druid-hdfs-storage"]`。
- 在“Deep Storage”和“索引服务日志”下注释掉本地存储的配置。
- 取消注释并在“Deep Storage”和“索引服务日志”的“For HDFS”部分中配置适当的值。

在此之后，您应该进行以下更改：

```text
druid.extensions.loadList=["druid-hdfs-storage"]

#druid.storage.type=local
#druid.storage.storageDirectory=var/druid/segments

druid.storage.type=hdfs
druid.storage.storageDirectory=/druid/segments

#druid.indexer.logs.type=file
#druid.indexer.logs.directory=var/druid/indexing-logs

druid.indexer.logs.type=hdfs
druid.indexer.logs.directory=/druid/indexing-logs
```

也，

- 将Hadoop配置XML（core-site.xml，hdfs-site.xml，yarn-site.xml，mapred-site.xml）放在Druid节点的类路径上。您可以通过复制它们来完成此操作 `conf/druid/_common/`。

## 配置Tranquility Server（可选）

可以通过由Tranquility Server提供支持的简单HTTP API将数据流发送到Druid。如果您将使用此功能，那么此时您应该[配置Tranquility Server](http://druid.io/docs/0.12.3/ingestion/stream-ingestion.html#server)。

## 配置Tranquility Kafka（可选）

德鲁伊可以通过Tranquility Kafka消耗来自Kafka的溪流。如果您将使用此功能，那么此时您应该 [配置Tranquility Kafka](http://druid.io/docs/0.12.3/ingestion/stream-ingestion.html#kafka)。

## 配置连接到Hadoop（可选）

如果您要从Hadoop集群加载数据，那么此时您应该配置德鲁伊以了解您的集群：

- 更新`druid.indexer.task.hadoopWorkingPath`中`conf/middleManager/runtime.properties`为您想使用的索引过程所需的临时文件的HDFS路径。 `druid.indexer.task.hadoopWorkingPath=/tmp/druid-indexing`是一种常见的选择。
- 将Hadoop配置XML（core-site.xml，hdfs-site.xml，yarn-site.xml，mapred-site.xml）放在Druid节点的类路径上。您可以通过将它们复制到`conf/druid/_common/core-site.xml`，`conf/druid/_common/hdfs-site.xml`等等来完成此 操作。

请注意，您无需使用HDFS深存储即可从Hadoop加载数据。例如，如果您的群集在Amazon Web Services上运行，我们建议您使用S3进行深度存储，即使您使用Hadoop或Elastic MapReduce加载数据也是如此。

有关详细信息，请参阅[批量提取](http://druid.io/docs/0.12.3/ingestion/batch-ingestion.html)。

## 配置德鲁伊协调的地址

在这个简单的集群中，您将在同一服务器上部署单个Druid协调器，单个Druid Overlord，单个ZooKeeper实例和嵌入式Derby元数据存储。

在`conf/druid/_common/common.runtime.properties`，将“zk.service.host”替换为运行ZK实例的计算机的地址：

- `druid.zk.service.host`

在`conf/druid/_common/common.runtime.properties`，将“metadata.storage。*”替换为您将用作元数据存储的计算机的地址：

- `druid.metadata.storage.connector.connectURI`
- `druid.metadata.storage.connector.host`

在制作中，我们建议运行2个服务器，每个服务器运行一个德鲁伊协调员和一个德鲁伊霸主。我们还建议在自己的专用硬件上运行ZooKeeper集群，以及在自己的专用硬件上运行 MySQL或PostgreSQL等复制的[元数据存储](http://druid.io/docs/latest/dependencies/metadata-storage.html)。

## 调整提供查询的德鲁伊流程

Druid Historicals和MiddleManagers可以位于同一硬件上。德鲁伊的两个流程都可以从调整到运行的硬件中受益匪浅。如果您正在运行Tranquility Server或Kafka，您还可以将Tranquility与这两个德鲁伊进程共存。如果您使用的是[r3.2xlarge](https://aws.amazon.com/ec2/instance-types/#r3) EC2实例或类似硬件，则分发中的配置是一个合理的起点。

如果您使用的是不同的硬件，我们建议您调整特定硬件的配置。最常调整的配置是：

- `-Xmx` 和 `-Xms`
- `druid.server.http.numThreads`
- `druid.processing.buffer.sizeBytes`
- `druid.processing.numThreads`
- `druid.query.groupBy.maxIntermediateRows`
- `druid.query.groupBy.maxResults`
- `druid.server.maxSize`和`druid.segmentCache.locations`历史节点
- `druid.worker.capacity` 在MiddleManagers上

保持-XX：MaxDirectMemory> = numThreads * sizeBytes，否则德鲁伊将无法启动..

有关所有可能配置选项的完整说明，请参阅Druid [配置文档](http://druid.io/docs/0.12.3/configuration/index.html)。

## 调整德鲁伊经纪人

德鲁伊经纪人也可以通过调整他们运行的硬件而受益匪浅。如果您使用的是[r3.2xlarge](https://aws.amazon.com/ec2/instance-types/#r3) EC2实例或类似硬件，则分发中的配置是一个合理的起点。

如果您使用的是不同的硬件，我们建议您调整特定硬件的配置。最常调整的配置是：

- `-Xmx` 和 `-Xms`
- `druid.server.http.numThreads`
- `druid.cache.sizeInBytes`
- `druid.processing.buffer.sizeBytes`
- `druid.processing.numThreads`
- `druid.query.groupBy.maxIntermediateRows`
- `druid.query.groupBy.maxResults`

保持-XX：MaxDirectMemory> = numThreads * sizeBytes，否则德鲁伊将无法启动。

有关所有可能配置选项的完整说明，请参阅Druid [配置文档](http://druid.io/docs/0.12.3/configuration/index.html)。

## 打开端口（如果使用防火墙）

如果您正在使用防火墙或仅允许特定端口上的流量的其他系统，请允许以下内容的入站连接：

- 1527（协调器上的Derby;如果您使用的是MySQL或PostgreSQL等单独的元数据存储，则不需要）
- 2181（ZooKeeper;如果您使用单独的ZooKeeper集群则不需要）
- 8081（协调员）
- 8082（经纪人）
- 8083（历史）
- 8084（独立实时，如果使用）
- 8088（路由器，如果使用）
- 8090（霸王）
- 8091,8100-8199（德鲁伊中级经理;如果你有一个非常高的话，你可能需要高于8199号港口`druid.worker.capacity`）
- 8200（Tranquility Server，如果使用的话）

在生产中，我们建议在自己的专用硬件上而不是在Coordinator服务器上部署ZooKeeper和元数据存储。

## 启动协调器，Overlord，Zookeeper和元数据存储

将Druid发行版和您编辑的配置复制到协调服务器。如果您一直在编辑本地计算机上的配置，则可以使用*rsync*复制它们：

```bash
rsync -az druid-0.12.3/ COORDINATION_SERVER:druid-0.12.3/
```

登录到协调服务器并安装Zookeeper：

```bash
curl http://www.gtlib.gatech.edu/pub/apache/zookeeper/zookeeper-3.4.11/zookeeper-3.4.11.tar.gz -o zookeeper-3.4.11.tar.gz
tar -xzf zookeeper-3.4.11.tar.gz
cd zookeeper-3.4.11
cp conf/zoo_sample.cfg conf/zoo.cfg
./bin/zkServer.sh start
```

在生产中，我们还建议在自己的专用硬件上运行ZooKeeper集群。

在协调服务器上，*cd*进入分发并启动协调服务（您应该在不同的窗口中执行此操作或将日志传递到文件）：

```bash
java `cat conf/druid/coordinator/jvm.config | xargs` -cp conf/druid/_common:conf/druid/coordinator:lib/* io.druid.cli.Main server coordinator
java `cat conf/druid/overlord/jvm.config | xargs` -cp conf/druid/_common:conf/druid/overlord:lib/* io.druid.cli.Main server overlord
```

您应该看到为每个启动的服务打印出一条日志消息。您可以通过`var/log/druid`使用其他终端查看目录来查看任何服务的详细日志。

## 启动历史和中间管理员

将Druid发行版和您编辑的配置复制到为Druid Historicals和MiddleManagers预留的服务器中。

在每个上，*cd*进入发行版并运行此命令以启动数据服务器：

```bash
java `cat conf/druid/historical/jvm.config | xargs` -cp conf/druid/_common:conf/druid/historical:lib/* io.druid.cli.Main server historical
java `cat conf/druid/middleManager/jvm.config | xargs` -cp conf/druid/_common:conf/druid/middleManager:lib/* io.druid.cli.Main server middleManager
```

您可以根据需要添加更多具有Druid Historicals和MiddleManagers的服务器。

对于具有复杂资源分配需求的群集，您可以拆分Historicals和MiddleManagers并单独扩展组件。这也允许您利用Druid内置的MiddleManager自动缩放功能。

如果您使用Kafka或HTTP进行基于推送的流提取，您还可以在包含MiddleManagers和Historicals的相同硬件上启动Tranquility Server。对于大规模生产，MiddleManagers和Tranquility Server仍然可以共存。如果您使用流处理器运行Tranquility（非服务器），则可以将Tranquility与流处理器共同定位，而不需要Tranquility Server。

```bash
curl -O http://static.druid.io/tranquility/releases/tranquility-distribution-0.8.0.tgz
tar -xzf tranquility-distribution-0.8.0.tgz
cd tranquility-distribution-0.8.0
bin/tranquility <server or kafka> -configFile <path_to_druid_distro>/conf/tranquility/<server or kafka>.json
```

## 启动德鲁伊经纪人

将德鲁伊发行版和您编辑的配置复制到为德鲁伊经纪人预留的服务器中。

在每个上，*cd*进入发行版并运行此命令以启动Broker（您可能希望将输出通过管道传输到日志文件）：

```bash
java `cat conf/druid/broker/jvm.config | xargs` -cp conf/druid/_common:conf/druid/broker:lib/* io.druid.cli.Main server broker
```

您可以根据查询负载根据需要添加更多代理。

## 加载数据中

恭喜你，你现在有一个德鲁伊集群！下一步是了解根据您的用例将数据加载到Druid中的推荐方法。详细了解如何[加载数据](http://druid.io/docs/0.12.3/ingestion/index.html)。