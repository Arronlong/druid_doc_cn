# 使用不同版本的Hadoop

德鲁伊可以通过两种方式与Hadoop互动：

1. [使用](http://druid.io/docs/0.12.3/development/extensions-core/hdfs.html) druid-hdfs-storage扩展[将HDFS用于深度存储](http://druid.io/docs/0.12.3/development/extensions-core/hdfs.html)。
2. 使用Map / Reduce作业[从Hadoop批量加载数据](http://druid.io/docs/0.12.3/ingestion/hadoop.html)。

这些不一定联系在一起; 您可以将Hadoop作业的数据加载到非HDFS深存储（如S3）中，即使您从流中加载数据而不是使用Hadoop作业，也可以使用HDFS进行深度存储。

为了获得最佳效果，请在配置Druid以与您最喜爱的Hadoop发行版进行交互时使用这些提示。

## 提示＃1：将Hadoop XML放在Druid类路径上

将Hadoop配置XML（core-site.xml，hdfs-site.xml，yarn-site.xml，mapred-site.xml）放在Druid节点的类路径上。您可以通过将它们复制到`conf/druid/_common/core-site.xml`， `conf/druid/_common/hdfs-site.xml`等等来完成此操作。这允许Druid找到您的Hadoop集群并正确提交作业。

## 提示＃2：Hadoop上的类加载器修改（仅限Map / Reduce作业）

Druid使用了许多可能存在于Hadoop集群中的库，如果这些库发生冲突，Map / Reduce作业可能会失败。使用Hadoop作业属性启用类加载器隔离可以避免此问题`mapreduce.job.classloader = true`。这指示Hadoop为Druid依赖项和Hadoop自己的依赖项使用单独的类加载器。

如果您的Hadoop版本不支持此功能，您还可以尝试设置该属性 `mapreduce.job.user.classpath.first = true`。这指示Hadoop更喜欢在发生冲突时加载德鲁伊版本的库。

通常，您应该只设置其中一个参数，而不是两者。

可以使用以下任一方法设置这些属性：

- 使用任务定义，例如添加`"mapreduce.job.classloader": "true"`到`jobProperties`的`tuningConfig`你的索引任务（见[的Hadoop批处理摄入文档](http://druid.io/docs/0.12.3/ingestion/hadoop.html)）。
- 使用系统属性，例如在middleManager集上`druid.indexer.runner.javaOpts=... -Dhadoop.mapreduce.job.classloader=true`。

### 覆盖特定的类

何时`mapreduce.job.classloader = true`，还可以专门定义应从hadoop系统类路径加载哪些类，哪些类应从作业提供的JAR加载。

这是通过`mapreduce.job.classloader.system.classes`在`jobProperties`of 属性中定义类包含/排除模式来控制的`tuningConfig`。

例如，某些社区成员报告了Validator类的版本不兼容错误：

```text
Error: java.lang.ClassNotFoundException: javax.validation.Validator
```

以下内容`jobProperties`排除了`javax.validation.`从系统类路径加载的类，同时包括来自的类`java.,javax.,org.apache.commons.logging.,org.apache.log4j.,org.apache.hadoop.`。

```text
"jobProperties": {
  "mapreduce.job.classloader": "true",
  "mapreduce.job.classloader.system.classes": "-javax.validation.,java.,javax.,org.apache.commons.logging.,org.apache.log4j.,org.apache.hadoop."
}
```

[mapred-default.xml](https://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml)文档包含有关此属性的更多信息。

## 提示＃3：使用特定版本的Hadoop库

Druid从两个不同的位置加载Hadoop客户端库。每组库都加载在一个独立的类加载器中。

1. HDFS深存储使用jar `extensions/druid-hdfs-storage/`来读取和写入HDFS上的德鲁伊数据。
2. 批量提取使用jar `hadoop-dependencies/`来提交Map / Reduce作业（可通过`druid.extensions.hadoopDependenciesDir`运行时属性自定义的位置 ;请参阅[配置](http://druid.io/docs/0.12.3/configuration/index.html#extensions)）。

`hadoop-client:2.3.0`是两个目的与Druid捆绑在一起的Hadoop客户端的默认版本。这适用于许多Hadoop发行版（该版本不一定需要匹配），但如果遇到问题，您可以使用与您的发行版完全匹配的德鲁伊加载库。要执行此操作，请从Hadoop集群复制jar，或使用该`pull-deps`工具从Maven存储库下载jar。

### 首选：使用德鲁伊的标准机制加载

If you have issues with HDFS deep storage, you can switch your Hadoop client libraries by recompiling the druid-hdfs-storage extension using an alternate version of the Hadoop client libraries. You can do this by editing the main Druid pom.xml and rebuilding the distribution by running `mvn package`.

If you have issues with Map/Reduce jobs, you can switch your Hadoop client libraries without rebuilding Druid. You can do this by adding a new set of libraries to the `hadoop-dependencies/` directory (or another directory specified by druid.extensions.hadoopDependenciesDir) and then using `hadoopDependencyCoordinates` in the [Hadoop Index Task](http://druid.io/docs/0.12.3/ingestion/hadoop.html) to specify the Hadoop dependencies you want Druid to load.

Example:

假设您指定了`druid.extensions.hadoopDependenciesDir=/usr/local/druid_tarball/hadoop-dependencies`，并且已经下载了`hadoop-client`2.3.0和2.4.0，可以从Hadoop集群中复制它们，也可以使用`pull-deps`从Maven存储库下载jar。然后在下面`hadoop-dependencies`，你的罐子应该是这样的：

```text
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

如您所见，在下面`hadoop-client`，有两个子目录，每个子目录表示一个版本`hadoop-client`。

接下来，`hadoopDependencyCoordinates`在[Hadoop Index Task中使用](http://druid.io/docs/0.12.3/ingestion/hadoop.html)以指定您希望Druid加载的Hadoop依赖项。

例如，在您的Hadoop索引任务规范文件中，您可以编写：

```
"hadoopDependencyCoordinates": ["org.apache.hadoop:hadoop-client:2.4.0"]
```

This instructs Druid to load hadoop-client 2.4.0 when processing the task. What happens behind the scene is that Druid first looks for a folder called `hadoop-client` underneath `druid.extensions.hadoopDependenciesDir`, then looks for a folder called `2.4.0` underneath `hadoop-client`, and upon successfully locating these folders, hadoop-client 2.4.0 is loaded.

### Alternative: Append your Hadoop jars to the Druid classpath

You can also load Hadoop client libraries in Druid's main classloader, rather than an isolated classloader. This mechanism is relatively easy to reason about, but it also means that you have to ensure that all dependency jars on the classpath are compatible. That is, Druid makes no provisions while using this method to maintain class loader isolation so you must make sure that the jars on your classpath are mutually compatible.

1. 设置`druid.indexer.task.defaultHadoopCoordinates=[]`。通过将其设置为空列表，Druid将不会加载除类路径中指定的依赖项之外的任何其他Hadoop依赖项。
2. 将你的Hadoop罐子贴在德鲁伊的类路径上。德鲁伊会将它们加载到系统中。

## 有关特定Hadoop发行版的说明

如果上述提示无法解决您在HDFS深层存储或Hadoop批量索引方面遇到的任何问题，那么您可以幸运地获得德鲁伊社区提供的以下建议之一。

### CDH

社区成员报告了在运行Mapreduce工作时，CDH中使用的Jackson版本与德鲁伊之间的依赖冲突：

```text
java.lang.VerifyError: class com.fasterxml.jackson.datatype.guava.deser.HostAndPortDeserializer overrides final method deserialize.(Lcom/fasterxml/jackson/core/JsonParser;Lcom/fasterxml/jackson/databind/DeserializationContext;)Ljava/lang/Object;
```

**首选解决方法**

首先，尝试上面“Hadoop上的类加载器修改”下的提示。据报道，更新版本的CDH可与classloader隔离选项（`mapreduce.job.classloader = true`）一起使用。

**替代解决方法 - 1**

您可以尝试编辑Druid的pom.xml依赖项，以匹配Hadoop版本中的Jackson版本并重新编译Druid。

有关建立德鲁伊的更多信息，请参阅[建筑德鲁伊](http://druid.io/docs/0.12.3/development/build.html)。

**替代解决方法 - 2**

另一个解决方案是使用[sbt](http://www.scala-sbt.org/)构建一个自定义胖jar的jar ，它手动排除所有冲突的Jackson依赖项，然后将这个胖jar放在启动霸主索引服务的命令的类路径中。为此，请按照以下步骤操作。

（1）下载并安装sbt。

（2）创建一个名为'druid_build'的新目录。

（3）将Cd转换为'druid_build'并使用[此处](http://druid.io/docs/0.12.3/operations/use_sbt_to_build_fat_jar.html)的内容创建build.sbt文件。

您始终可以添加更多建筑物目标或删除不需要的目标。

（4）在同一目录中创建一个名为“project”的新目录。

（5）将德鲁伊源代码放入'druid_build / project'。

（6）创建一个文件'druid_build / project / assembly.sbt'，内容如下。 `addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.13.0")`

（7）在'druid_build'目录中，运行'sbt assembly'。

（8）在'druid_build / target / scala-2.10'文件夹中，你会发现你刚刚建立的胖罐。

（9）确保已完全移除您上传的罐子。HDFS目录默认为'/ tmp / druid-indexing / classpath'。

（10）启动索引服务时，在类路径中包含fat jar。确保你已经从类路径中删除了'lib / *'，因为现在fat jar包含了你所需要的一切。

**替代解决方法 - 3**

如果sbt不是您的选择，您也可以使用`maven-shade-plugin`制作胖罐：重新定位所有jackson包也将解决它。这样，德鲁伊不会受嵌入hadoop的jackson库的影响。请按照以下步骤操作：

（1）添加你需要的所有扩展`services/pom.xml`喜欢

```xml
 <dependency>
      <groupId>io.druid.extensions</groupId>
      <artifactId>druid-avro-extensions</artifactId>
      <version>${project.parent.version}</version>
  </dependency>

  <dependency>
      <groupId>io.druid.extensions.contrib</groupId>
      <artifactId>druid-parquet-extensions</artifactId>
      <version>${project.parent.version}</version>
  </dependency>

  <dependency>
      <groupId>io.druid.extensions</groupId>
      <artifactId>druid-hdfs-storage</artifactId>
      <version>${project.parent.version}</version>
  </dependency>

  <dependency>
      <groupId>io.druid.extensions</groupId>
      <artifactId>mysql-metadata-storage</artifactId>
      <version>${project.parent.version}</version>
  </dependency>
```

（2）Shade jackson包装并组装一个胖罐。

```xml
<plugin>
     <groupId>org.apache.maven.plugins</groupId>
     <artifactId>maven-shade-plugin</artifactId>
     <executions>
         <execution>
             <phase>package</phase>
             <goals>
                 <goal>shade</goal>
             </goals>
             <configuration>
                 <outputFile>
                     ${project.build.directory}/${project.artifactId}-${project.version}-selfcontained.jar
                 </outputFile>
                 <relocations>
                     <relocation>
                         <pattern>com.fasterxml.jackson</pattern>
                         <shadedPattern>shade.com.fasterxml.jackson</shadedPattern>
                     </relocation>
                 </relocations>
                 <artifactSet>
                     <includes>
                         <include>*:*</include>
                     </includes>
                 </artifactSet>
                 <filters>
                     <filter>
                         <artifact>*:*</artifact>
                         <excludes>
                             <exclude>META-INF/*.SF</exclude>
                             <exclude>META-INF/*.DSA</exclude>
                             <exclude>META-INF/*.RSA</exclude>
                         </excludes>
                     </filter>
                 </filters>
                 <transformers>
                     <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                 </transformers>
             </configuration>
         </execution>
     </executions>
 </plugin>
```

在项目根目录`services/target/xxxxx-selfcontained.jar`之后复制出来`mvn install`以供进一步使用。

（3）运行hadoop索引器（现在不可能发布索引任务）如下所示。`lib`不再需要了。由于hadoop索引器是一个独立的工具，因此您无需替换正在运行的服务的jar：

```bash
java -Xmx32m \
  -Dfile.encoding=UTF-8 -Duser.timezone=UTC \
  -classpath config/hadoop:config/overlord:config/_common:$SELF_CONTAINED_JAR:$HADOOP_DISTRIBUTION/etc/hadoop \
  -Djava.security.krb5.conf=$KRB5 \
  io.druid.cli.Main index hadoop \
  $config_path
```