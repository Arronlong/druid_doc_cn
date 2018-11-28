# Druid快速入门

在本快速入门中，我们将下载Druid，在一台机器上进行设置，加载一些数据并查询数据。

## 先决条件

你会需要：

- Java 8或更高版本
- Linux，Mac OS X或其他类Unix操作系统（不支持Windows）
- 8G的RAM
- 2个vCPU

在Mac OS X上，您可以使用[Oracle的JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)来安装Java。

在Linux上，您的OS包管理器应该能够为Java提供帮助。如果基于Ubuntu的操作系统没有最新版本的Java，则WebUpd8会[为这些操作系统](http://www.webupd8.org/2012/09/install-oracle-java-8-in-ubuntu-via-ppa.html)提供[软件包](http://www.webupd8.org/2012/09/install-oracle-java-8-in-ubuntu-via-ppa.html)。

## 入门

要安装Druid，请在终端中发出以下命令：

```bash
curl -O http://static.druid.io/artifacts/releases/druid-0.12.3-bin.tar.gz
tar -xzf druid-0.12.3-bin.tar.gz
cd druid-0.12.3
```

在包中，您应该找到：

- `LICENSE` - 许可证文件。
- `bin/` - 对此快速入门有用的脚本。
- `conf/*` - 群集设置的模板配置。
- `conf-quickstart/*` - 此快速入门的配置。
- `extensions/*` - 所有Druid扩展。
- `hadoop-dependencies/*` - DruidHadoop依赖。
- `lib/*` - 所有包含核心Druid的软件包。
- `quickstart/*` - 对此快速入门有用的文件。

## 下载教程示例文件

在继续之前，请下载[教程示例包](http://druid.io/docs/0.12.3/tutorials/tutorial-examples.tar.gz)。

此tarball包含将在教程中使用的示例数据和提取规范。

```bash
curl -O http://druid.io/docs/0.12.3/tutorials/tutorial-examples.tar.gz
tar zxvf tutorial-examples.tar.gz
```

## 启动Zookeeper

Druid目前依赖[Apache ZooKeeper](http://zookeeper.apache.org/)进行分布式协调。您需要下载并运行Zookeeper。

```bash
curl http://www.gtlib.gatech.edu/pub/apache/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz -o zookeeper-3.4.10.tar.gz
tar -xzf zookeeper-3.4.10.tar.gz
cd zookeeper-3.4.10
cp conf/zoo_sample.cfg conf/zoo.cfg
./bin/zkServer.sh start
```

## 启动Druid服务

在Zookeeper运行后，返回druid-0.12.3目录。在该目录中，发出命令：

```bash
bin/init
```

这将为您设置一些目录。接下来，您可以在不同的终端窗口中启动Druid进程。本教程在同一系统上运行每个Druid进程。在大型分布式生产集群中，许多这些Druid进程仍然可以共同位于一起。

```bash
java `cat examples/conf/druid/coordinator/jvm.config | xargs` -cp "examples/conf/druid/_common:examples/conf/druid/_common/hadoop-xml:examples/conf/druid/coordinator:lib/*" io.druid.cli.Main server coordinator
java `cat examples/conf/druid/overlord/jvm.config | xargs` -cp "examples/conf/druid/_common:examples/conf/druid/_common/hadoop-xml:examples/conf/druid/overlord:lib/*" io.druid.cli.Main server overlord
java `cat examples/conf/druid/historical/jvm.config | xargs` -cp "examples/conf/druid/_common:examples/conf/druid/_common/hadoop-xml:examples/conf/druid/historical:lib/*" io.druid.cli.Main server historical
java `cat examples/conf/druid/middleManager/jvm.config | xargs` -cp "examples/conf/druid/_common:examples/conf/druid/_common/hadoop-xml:examples/conf/druid/middleManager:lib/*" io.druid.cli.Main server middleManager
java `cat examples/conf/druid/broker/jvm.config | xargs` -cp "examples/conf/druid/_common:examples/conf/druid/_common/hadoop-xml:examples/conf/druid/broker:lib/*" io.druid.cli.Main server broker
```

每个服务启动后，您就可以开始加载数据了。

### 重置群集状态

所有持久状态（如集群元数据存储和服务段）都将保存在`var`druid-0.12.3软件包根目录下的目录中。

稍后，如果您想停止服务，请按CTRL-C退出正在运行的java进程。如果要在停止服务后进行干净启动，请删除`log`和`var`目录并`init`再次运行脚本，然后关闭Zookeper并删除Zookeeper dataDir `/tmp/zookeeper`。

来自druid-0.12.3目录：

```bash
rm -rf log
rm -rf var
bin/init
```

如果您运行了[教程：从Kafka加载流数据](http://druid.io/docs/0.12.3/tutorials/tutorial-kafka.html)，您应该在关闭Zookeeper之前关闭Kafka，然后删除位于的Kafka日志目录`/tmp/kafka-logs`。

Ctrl-C关闭Kafka代理，然后删除日志目录：

```bash
rm -rf /tmp/kafka-logs
```

现在停止Zookeeper并清除其状态。从zookeeper-3.4.10目录：

```bash
./bin/zkServer.sh stop
rm -rf /tmp/zookeeper
```

清除Druid和Zookeeper状态后，重新启动Zookeeper，然后重新启动Druid服务。

## 加载数据中

### 教程数据集

对于以下数据加载教程，我们已经包含了一个示例数据文件，其中包含2015-09-12发生的Wikipedia页面编辑事件。

此示例数据位于`quickstart/wikiticker-2015-09-12-sampled.json.gz`Druid包根目录中。页面编辑事件作为JSON对象存储在文本文件中。

示例数据包含以下列，示例事件如下所示：

- 添加
- 渠道
- 城市名称
- 评论
- countryIsoCode
- 国家的名字
- 删除
- 三角洲
- isAnonymous
- isMinor
- 是新的
- isRobot
- isUnpatrolled
- metroCode
- 命名空间
- 页
- regionIsoCode
- regionName
- 用户

```json
{
  "timestamp":"2015-09-12T20:03:45.018Z",
  "channel":"#en.wikipedia",
  "namespace":"Main"
  "page":"Spider-Man's powers and equipment",
  "user":"foobar",
  "comment":"/* Artificial web-shooters */",
  "cityName":"New York",
  "regionName":"New York",
  "regionIsoCode":"NY",
  "countryName":"United States",
  "countryIsoCode":"US",
  "isAnonymous":false,
  "isNew":false,
  "isMinor":false,
  "isRobot":false,
  "isUnpatrolled":false,
  "added":99,
  "delta":99,
  "deleted":0,
}
```

以下教程演示了将数据加载到Druid中的各种方法，包括批处理和流式使用案例。

### [教程：加载文件](http://druid.io/docs/0.12.3/tutorials/tutorial-batch.html)

本教程演示了如何使用Druid的本机批处理提取来执行批处理文件加载。

### [教程：从Kafka加载流数据](http://druid.io/docs/0.12.3/tutorials/tutorial-kafka.html)

本教程演示了如何从Kafka主题加载流数据。

### [教程：使用Hadoop加载文件](http://druid.io/docs/0.12.3/tutorials/tutorial-batch-hadoop.html)

本教程演示了如何使用远程Hadoop集群执行批处理文件加载。

### [教程：使用Tranquility加载数据](http://druid.io/docs/0.12.3/tutorials/tutorial-tranquility.html)

本教程演示了如何使用Tranquility服务将事件推送到Druid来加载流数据。

### [教程：编写自己的摄取规范](http://druid.io/docs/0.12.3/tutorials/tutorial-ingestion-spec.html)

本教程演示了如何编写新的提取规范并使用它来加载数据。