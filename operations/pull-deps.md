# pull-deps工具

`pull-deps` 是一个工具，可以将依赖关系下拉到本地存储库，并根据需要将依赖关系放在扩展目录中。

`pull-deps` 有几个命令行选项，它们如下：

`-c`或`--coordinate`（可以指定乘法次数）

下拉的扩展坐标，后跟maven坐标，例如io.druid.extensions：mysql-metadata-storage

`-h`或`--hadoop-coordinate`（可以指定乘法次数）

Hadoop依赖关系下拉，然后是maven坐标，例如org.apache.hadoop：hadoop-client：2.4.0

```
--no-default-hadoop
```

不要下拉默认的hadoop坐标，即org.apache.hadoop：hadoop-client：2.3.0。如果`-h`提供了选项，则不会下载默认的hadoop坐标。

```
--clean
```

在下拉依赖项之前删除现有扩展和hadoop依赖项目录。

```
-l` 要么 `--localRepository
```

Maven将用于放置下载文件的本地存储库。然后pull-deps会根据需要将这些文件放到extensions目录中。

```
-r` 要么 `--remoteRepository
```

添加远程存储库。除非提供了--no-default-remote-repositories，否则这些将在https://repo1.maven.org/maven2/和https://metamx.artifactoryonline.com/metamx/pub-libs-releases-local之后使用。

```
--no-default-remote-repositories
```

不要使用默认的远程存储库，只使用通过--remoteRepository直接提供的存储库。

```
-d` 要么 `--defaultVersion
```

用于没有版本信息的扩展坐标的版本。例如，如果扩展坐标为`io.druid.extensions:mysql-metadata-storage`，且默认版本为`0.12.3`，则此坐标将被视为`io.druid.extensions:mysql-metadata-storage:0.12.3`

要跑`pull-deps`，你应该

1）指定`druid.extensions.directory`和`druid.extensions.hadoopDependenciesDir`，这两个属性告诉`pull-deps`放置扩展的位置。如果未指定它们，将使用默认值，请参阅[配置](http://druid.io/docs/0.12.3/configuration/index.html)。

2）告诉`pull-deps`使用`-c`或`-h`选项下载什么，然后是maven坐标。

例：

假设您想要下载`druid-rabbitmq`，`mysql-metadata-storage`并`hadoop-client`与特定的版本（2.3.0都和2.4.0），你可以运行`pull-deps`与命令`-c io.druid.extensions:druid-examples:0.12.3`，`-c io.druid.extensions:mysql-metadata-storage:0.12.3`，`-h org.apache.hadoop:hadoop-client:2.3.0`和`-h org.apache.hadoop:hadoop-client:2.4.0`，一个示例命令是：

```text
java -classpath "/my/druid/lib/*" io.druid.cli.Main tools pull-deps --clean -c io.druid.extensions:mysql-metadata-storage:0.12.3 -c io.druid.extensions.contrib:druid-rabbitmq:0.12.3 -h org.apache.hadoop:hadoop-client:2.3.0 -h org.apache.hadoop:hadoop-client:2.4.0
```

由于`--clean`提供，此命令会先删除在指定的目录`druid.extensions.directory`和`druid.extensions.hadoopDependenciesDir`，然后重新创建它们，并开始下载扩展那里。完成下载后，如果您转到指定的扩展目录，您将看到

```text
tree extensions
extensions
├── druid-examples
│   ├── commons-beanutils-1.8.3.jar
│   ├── commons-digester-1.8.jar
│   ├── commons-logging-1.1.1.jar
│   ├── commons-validator-1.4.0.jar
│   ├── druid-examples-0.12.3.jar
│   ├── twitter4j-async-3.0.3.jar
│   ├── twitter4j-core-3.0.3.jar
│   └── twitter4j-stream-3.0.3.jar
└── mysql-metadata-storage
    ├── jdbi-2.32.jar
    ├── mysql-connector-java-5.1.34.jar
    └── mysql-metadata-storage-0.12.3.jar
tree hadoop-dependencies
hadoop-dependencies/
└── hadoop-client
    ├── 2.3.0
    │   ├── activation-1.1.jar
    │   ├── avro-1.7.4.jar
    │   ├── commons-beanutils-1.7.0.jar
    │   ├── commons-beanutils-core-1.8.0.jar
    │   ├── commons-cli-1.2.jar
    │   ├── commons-codec-1.4.jar
    ..... lots of jars
    └── 2.4.0
        ├── activation-1.1.jar
        ├── avro-1.7.4.jar
        ├── commons-beanutils-1.7.0.jar
        ├── commons-beanutils-core-1.8.0.jar
        ├── commons-cli-1.2.jar
        ├── commons-codec-1.4.jar
    ..... lots of jars
```

请注意，如果指定`--defaultVersion`，则不必将版本信息放在坐标中。例如，如果您想要两者`druid-rabbitmq`并`mysql-metadata-storage`使用版本`0.12.3`，则可以将上面的命令更改为

```text
java -classpath "/my/druid/lib/*" io.druid.cli.Main tools pull-deps --defaultVersion 0.12.3 --clean -c io.druid.extensions:mysql-metadata-storage -c io.druid.extensions.contrib:druid-rabbitmq -h org.apache.hadoop:hadoop-client:2.3.0 -h org.apache.hadoop:hadoop-client:2.4.0
```

请注意，要使用pull-deps工具，您必须知道Maven groupId，artifactId和扩展的版本。对于[此处](http://druid.io/docs/0.12.3/development/extensions.html)列出的Druid社区扩展，groupId是“io.druid.extensions.contrib”，artifactId是扩展名。