# 教程：从Kafka加载流数据

## 入门

本教程演示了如何使用Druid Kafka索引服务从Kafka流加载数据。

对于本教程，我们假设您已经按照[单机快速入门](http://druid.io/docs/0.12.3/tutorials/index.html)中的描述下载了Druid，并让它在本地计算机上运行。您还不需要加载任何数据。

## 下载并启动Kafka

[Apache Kafka](http://kafka.apache.org/)是一种高吞吐量的消息总线，可以很好地与Druid配合使用。在本教程中，我们将使用Kafka 0.10.2.0。要下载Kafka，请在终端中发出以下命令：

```bash
curl -O https://archive.apache.org/dist/kafka/0.10.2.0/kafka_2.11-0.10.2.0.tgz
tar -xzf kafka_2.11-0.10.2.0.tgz
cd kafka_2.11-0.10.2.0
```

通过在新终端中运行以下命令来启动Kafka代理：

```bash
./bin/kafka-server-start.sh config/server.properties
```

运行此命令以创建名为*wikipedia*的Kafka主题，我们将向其发送数据：

```bash
./bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic wikipedia
```

## 启用德鲁伊卡夫卡摄取

我们将使用Druid的Kafka索引服务从我们新创建的*维基百科*主题中提取消息。要启动该服务，我们需要通过从Druid包根运行以下命令向Druid霸主提交一个主管规范：

```bash
curl -XPOST -H'Content-Type: application/json' -d @examples/wikipedia-kafka-supervisor.json http://localhost:8090/druid/indexer/v1/supervisor
```

如果主管成功创建，您将收到包含主管ID的回复; 在我们的例子中我们应该看到`{"id":"wikipedia-kafka"}`。

有关这里发生了什么的更多详细信息，请查看 [Druid Kafka索引服务文档](http://druid.io/docs/0.12.3/development/extensions-core/kafka-ingestion.html)。

## 加载数据

让我们为我们的主题启动一个控制台生产者并发送一些数据！

在Kafka目录中，运行以下命令，其中{PATH_TO_DRUID}被Druid目录的路径替换：

```bash
export KAFKA_OPTS="-Dfile.encoding=UTF-8"
./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic wikipedia < {PATH_TO_DRUID}/quickstart/wikiticker-2015-09-12-sampled.json
```

上一个命令将样本事件发布到*维基百科* Kafka主题，然后由Kafka索引服务将其提取到Druid中。你现在准备运行一些查询了！

## 查询您的数据

将数据发送到Kafka流后，它立即可用于查询。

请按照[查询教程](http://druid.io/docs/0.12.3/tutorials/tutorial-query.html)对新加载的数据运行一些示例查询。

## 清理

如果您希望浏览任何其他摄取教程，则需要重置群集并按照这些[重置说明进行操作](http://druid.io/docs/0.12.3/tutorials/index.html#resetting-cluster-state)，因为其他教程将写入相同的“维基百科”数据源。

## 进一步阅读

有关从Kafka流加载数据的更多信息，请参阅[Druid Kafka索引服务文档](http://druid.io/docs/0.12.3/development/extensions-core/kafka-ingestion.html)。