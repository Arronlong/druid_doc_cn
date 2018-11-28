# 教程：使用Hadoop加载批处理数据

本教程将向您展示如何使用远程Hadoop集群将数据文件加载到Druid中。

对于本教程，我们假设您已经使用Druid的本机批量摄取系统完成了上[一批的摄取教程](http://druid.io/docs/0.12.3/tutorials/tutorial-batch.html)。

## 安装Docker

本教程要求在教程计算机上安装[Docker](https://docs.docker.com/install/)。

Docker安装完成后，请继续执行本教程中的后续步骤。

## 构建Hadoop docker镜像

在本教程中，我们为Hadoop 2.7.3集群提供了一个Dockerfile，我们将用它来运行批量索引任务。

此Dockerfile和相关文件位于`examples/hadoop/docker`。

从druid-0.12.3软件包根目录开始，运行以下命令以构建名为“druid-hadoop-demo”的Docker映像，其版本标签为“2.7.3”：

```bash
cd examples/hadoop/docker
docker build -t druid-hadoop-demo:2.7.3 .
```

这将开始构建Hadoop映像。完成映像构建后，您应该会看到消息`Successfully tagged druid-hadoop-demo:2.7.3`打印到控制台。

## 设置Hadoop docker集群

### 创建临时共享目录

我们需要主机和Hadoop容器之间的共享文件夹来传输一些文件。

让我们创建一些文件夹`/tmp`，稍后我们将在启动Hadoop容器时使用它们：

```bash
mkdir -p /tmp/shared
mkdir -p /tmp/shared/hadoop-xml
```

### 配置/ etc / hosts

在主机上，将以下条目添加到`/etc/hosts`：

```text
127.0.0.1 druid-hadoop-demo
```

### 启动Hadoop容器

一旦`/tmp/shared`创建文件夹和`etc/hosts`项已添加，运行下面的命令来启动Hadoop的容器。

```bash
docker run -it  -h druid-hadoop-demo --name druid-hadoop-demo -p 50010:50010 -p 50020:50020 -p 50075:50075 -p 50090:50090 -p 8020:8020 -p 10020:10020 -p 19888:19888 -p 8030:8030 -p 8031:8031 -p 8032:8032 -p 8033:8033 -p 8040:8040 -p 8042:8042 -p 8088:8088 -p 8443:8443 -p 2049:2049 -p 9000:9000 -p 49707:49707 -p 2122:2122  -p 34455:34455 -v /tmp/shared:/shared druid-hadoop-demo:2.7.3 /etc/bootstrap.sh -bash
```

启动容器后，终端将连接到容器内运行的bash shell：

```bash
Starting sshd:                                             [  OK  ]
18/07/26 17:27:15 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Starting namenodes on [druid-hadoop-demo]
druid-hadoop-demo: starting namenode, logging to /usr/local/hadoop/logs/hadoop-root-namenode-druid-hadoop-demo.out
localhost: starting datanode, logging to /usr/local/hadoop/logs/hadoop-root-datanode-druid-hadoop-demo.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /usr/local/hadoop/logs/hadoop-root-secondarynamenode-druid-hadoop-demo.out
18/07/26 17:27:31 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
starting yarn daemons
starting resourcemanager, logging to /usr/local/hadoop/logs/yarn--resourcemanager-druid-hadoop-demo.out
localhost: starting nodemanager, logging to /usr/local/hadoop/logs/yarn-root-nodemanager-druid-hadoop-demo.out
starting historyserver, logging to /usr/local/hadoop/logs/mapred--historyserver-druid-hadoop-demo.out
bash-4.1#  
```

该`Unable to load native-hadoop library for your platform... using builtin-java classes where applicable`警告消息可以忽略。

#### 访问Hadoop容器shell

要打开另一个shell到Hadoop容器，请运行以下命令：

```text
docker exec -it druid-hadoop-demo bash
```

### 将输入数据复制到Hadoop容器

从主机上的druid-0.12.3软件包根目录中，将`quickstart/wikiticker-2015-09-12-sampled.json.gz`示例数据复制到共享文件夹：

```bash
cp quickstart/wikiticker-2015-09-12-sampled.json.gz /tmp/shared/wikiticker-2015-09-12-sampled.json.gz
```

### 设置HDFS目录

在Hadoop容器的shell中，运行以下命令来设置本教程所需的HDFS目录，并将输入数据复制到HDFS。

```bash
cd /usr/local/hadoop/bin
./hadoop fs -mkdir /druid
./hadoop fs -mkdir /druid/segments
./hadoop fs -mkdir /quickstart
./hadoop fs -chmod 777 /druid
./hadoop fs -chmod 777 /druid/segments
./hadoop fs -chmod 777 /quickstart
./hadoop fs -chmod -R 777 /tmp
./hadoop fs -chmod -R 777 /user
./hadoop fs -put /shared/wikiticker-2015-09-12-sampled.json.gz /quickstart/wikiticker-2015-09-12-sampled.json.gz
```

如果遇到namenode错误，例如`mkdir: Cannot create directory /druid. Name node is in safe mode.`运行此命令时，Hadoop容器未完成初始化。发生这种情况时，请等待几分钟后重试命令。

## 配置德鲁伊使用Hadoop

为Hadoop批量索引配置Druid集群需要一些额外的步骤。

### 将Hadoop配置复制到Druid类路径

从Hadoop容器的shell中，运行以下命令将Hadoop .xml配置文件复制到共享文件夹：

```bash
cp /usr/local/hadoop/etc/hadoop/*.xml /shared/hadoop-xml
```

在主机上运行以下命令，其中{PATH_TO_DRUID}被Druid包的路径替换。

```bash
cp /tmp/shared/hadoop-xml/*.xml {PATH_TO_DRUID}/examples/conf/druid/_common/hadoop-xml/
```

### 更新德鲁伊细分和日志存储

在您喜欢的文本编辑器中，打开`examples/conf/druid/_common/common.runtime.properties`并进行以下编辑：

#### 禁用本地深度存储并启用HDFS深度存储

```text
#
# Deep storage
#

# For local disk (only viable in a cluster if this is a network mount):
#druid.storage.type=local
#druid.storage.storageDirectory=var/druid/segments

# For HDFS:
druid.storage.type=hdfs
druid.storage.storageDirectory=/druid/segments
```

#### 禁用本地日志存储并启用HDFS日志存储

```text
#
# Indexing service logs
#

# For local disk (only viable in a cluster if this is a network mount):
#druid.indexer.logs.type=file
#druid.indexer.logs.directory=var/druid/indexing-logs

# For HDFS:
druid.indexer.logs.type=hdfs
druid.indexer.logs.directory=/druid/indexing-logs
```

### 重启德鲁伊集群

将Hadoop .xml文件复制到Druid集群并更新段/日志存储配置以使用HDFS后，需要重新启动Druid集群才能使新配置生效。

如果群集仍在运行，请按CTRL-C终止每个Druid服务，然后重新运行它们。

## 加载批次数据

我们从2015年9月12日开始包含维基百科编辑样本，以帮助您入门。

要将此数据加载到Druid中，您可以提交指向该文件的*摄取任务*。我们已经包含了一个加载`wikiticker-2015-09-12-sampled.json.gz`存档中包含的文件的任务。要提交此任务，请在druid-0.12.3目录的新终端窗口中将其发布到Druid：

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/wikipedia-index-hadoop.json http://localhost:8090/druid/indexer/v1/task
```

如果提交成功，将打印任务的ID：

```bash
{"task":"index_hadoop_wikipedia-hadoop_2018-06-09T21:30:32.802Z"}
```

要查看摄取任务的状态，请转到您的霸主控制台： [http：// localhost：8090 / console.html](http://localhost:8090/console.html)。您可以定期刷新控制台，任务成功后，您应该看到任务的“成功”状态。

摄取任务完成后，数据将由历史节点加载，并在一两分钟内可供查询。您可以通过检查是否存在带有蓝色圆圈的数据源“wikipedia”来监控协调器控制台中加载数据的进度：[http：// localhost：8081 /＃/](http://localhost:8081/#/)。

![协调员控制台](http://druid.io/docs/0.12.3/tutorials/img/tutorial-batch-01.png)

## 查询您的数据

任务完成后，您的数据应在一两分钟内完全可用。您可以在协调器控制台上以[http：// localhost：8081 /＃/](http://localhost:8081/#/)监视此过程。

请按照[查询教程](http://druid.io/docs/0.12.3/tutorials/tutorial-query.html)对新加载的数据运行一些示例查询。

## 清理

本教程仅用于与[查询教程](http://druid.io/docs/0.12.3/tutorials/tutorial-query.html)一起使用。

如果您希望浏览任何其他教程，则需要：*按照[重置说明](http://druid.io/docs/0.12.3/tutorials/index.html#resetting-cluster-state)关闭群集并重置群集状态。*将深层存储和任务存储配置还原为本地类型`examples/conf/druid/_common/common.runtime.properties` *重新启动群集

这是必要的，因为其他摄取教程将写入相同的“wikipedia”数据源，后来的教程希望群集使用本地深度存储。

示例恢复配置：

```text
#
# Deep storage
#

# For local disk (only viable in a cluster if this is a network mount):
druid.storage.type=local
druid.storage.storageDirectory=var/druid/segments

# For HDFS:
#druid.storage.type=hdfs
#druid.storage.storageDirectory=/druid/segments

#
# Indexing service logs
#

# For local disk (only viable in a cluster if this is a network mount):
druid.indexer.logs.type=file
druid.indexer.logs.directory=var/druid/indexing-logs

# For HDFS:
#druid.indexer.logs.type=hdfs
#druid.indexer.logs.directory=/druid/indexing-logs
```

## 进一步阅读

有关使用Hadoop加载批处理数据的更多信息，请参阅[Hadoop批处理提取文档](http://druid.io/docs/0.12.3/ingestion/hadoop.html)。