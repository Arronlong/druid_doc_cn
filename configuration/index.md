# 配置参考

此页面记录了每种德鲁伊服务类型的所有配置属性。

## 目录

- [推荐的配置文件组织](http://druid.io/docs/0.12.3/configuration/index.html#recommended-configuration-file-organization)
- 常见配置
  - [JVM配置最佳实践](http://druid.io/docs/0.12.3/configuration/index.html#jvm-configuration-best-practices)
  - [扩展](http://druid.io/docs/0.12.3/configuration/index.html#extensions)
  - [模块](http://druid.io/docs/0.12.3/configuration/index.html#modules)
  - [动物园管理员](http://druid.io/docs/0.12.3/configuration/index.html#zookeper)
  - [参展商](http://druid.io/docs/0.12.3/configuration/index.html#exhibitor)
  - [TLS](http://druid.io/docs/0.12.3/configuration/index.html#tls)
  - [身份验证和授权](http://druid.io/docs/0.12.3/configuration/index.html#authentication-and-authorization)
  - [启动日志记录](http://druid.io/docs/0.12.3/configuration/index.html#startup-logging)
  - [请求记录](http://druid.io/docs/0.12.3/configuration/index.html#request-logging)
  - [启用指标](http://druid.io/docs/0.12.3/configuration/index.html#enabling-metrics)
  - [发出指标](http://druid.io/docs/0.12.3/configuration/index.html#emitting-metrics)
  - [元数据存储](http://druid.io/docs/0.12.3/configuration/index.html#metadata-storage)
  - [深度存储](http://druid.io/docs/0.12.3/configuration/index.html#deep-storage)
  - [任务记录](http://druid.io/docs/0.12.3/configuration/index.html#task-logging)
  - [索引服务发现](http://druid.io/docs/0.12.3/configuration/index.html#indexing-service-discovery)
  - [协调员发现](http://druid.io/docs/0.12.3/configuration/index.html#coordinator-discovery)
  - [宣布细分](http://druid.io/docs/0.12.3/configuration/index.html#announcing-segments)
  - [JavaScript的](http://druid.io/docs/0.12.3/configuration/index.html#javascript)
  - [双柱存储](http://druid.io/docs/0.12.3/configuration/index.html#double-column-storage)
- 协调员
  - 静态配置
    - [节点配置](http://druid.io/docs/0.12.3/configuration/index.html#coordinator-node-config)
    - [协调员行动](http://druid.io/docs/0.12.3/configuration/index.html#coordinator-operation)
    - [细分管理](http://druid.io/docs/0.12.3/configuration/index.html#segment-management)
    - [元数据检索](http://druid.io/docs/0.12.3/configuration/index.html#metadata-retrieval)
  - 动态配置
    - [查找](http://druid.io/docs/0.12.3/configuration/index.html#lookups-dynamic-configuration)
    - [压实](http://druid.io/docs/0.12.3/configuration/index.html#compaction-dynamic-configuration)
- 霸王
  - [节点配置](http://druid.io/docs/0.12.3/configuration/index.html#overlord-node-config)
  - [静态配置](http://druid.io/docs/0.12.3/configuration/index.html#overlord-static-configuration)
  - 动态配置
    - [工人选择战略](http://druid.io/docs/0.12.3/configuration/index.html#worker-select-strategy)
    - [自动配置器](http://druid.io/docs/0.12.3/configuration/index.html#autoscaler)
- MiddleManager＆Peons
  - [节点配置](http://druid.io/docs/0.12.3/configuration/index.html#middlemanager-node-config)
  - [MiddleManger配置](http://druid.io/docs/0.12.3/configuration/index.html#middlemanager-configuration)
  - [Peon Processing](http://druid.io/docs/0.12.3/configuration/index.html#peon-processing)
  - [Peon查询配置](http://druid.io/docs/0.12.3/configuration/index.html#peon-query-configuration)
  - [高速缓存](http://druid.io/docs/0.12.3/configuration/index.html#peon-caching)
  - [额外的Peon配置](http://druid.io/docs/0.12.3/configuration/index.html#additional-peon-configuration)
- 经纪人
  - [节点配置](http://druid.io/docs/0.12.3/configuration/index.html#broker-node-configs)
  - [查询配置](http://druid.io/docs/0.12.3/configuration/index.html#broker-query-configuration)
  - [SQL](http://druid.io/docs/0.12.3/configuration/index.html#sql)
  - [高速缓存](http://druid.io/docs/0.12.3/configuration/index.html#broker-caching)
  - [段发现](http://druid.io/docs/0.12.3/configuration/index.html#segment-discovery)
- 历史的
  - [节点配置](http://druid.io/docs/0.12.3/configuration/index.html#historical-node-config)
  - [一般配置](http://druid.io/docs/0.12.3/configuration/index.html#historical-general-configuration)
  - [查询配置](http://druid.io/docs/0.12.3/configuration/index.html#historical-query-configs)
  - [高速缓存](http://druid.io/docs/0.12.3/configuration/index.html#historical-caching)
- [高速缓存](http://druid.io/docs/0.12.3/configuration/index.html#cache-configuration)
- [一般查询配置](http://druid.io/docs/0.12.3/configuration/index.html#general-query-configuration)
- [实时节点（已弃用）](http://druid.io/docs/0.12.3/configuration/index.html#realtime-nodes)

## 推荐的配置文件组织

可以`conf`在Druid包根目录中看到组织Druid配置文件的推荐方法，如下所示：

```text
$ ls -R conf
druid       tranquility

conf/druid:
_common       broker        coordinator   historical    middleManager overlord

conf/druid/_common:
common.runtime.properties log4j2.xml

conf/druid/broker:
jvm.config         runtime.properties

conf/druid/coordinator:
jvm.config         runtime.properties

conf/druid/historical:
jvm.config         runtime.properties

conf/druid/middleManager:
jvm.config         runtime.properties

conf/druid/overlord:
jvm.config         runtime.properties

conf/tranquility:
kafka.json  server.json
```

每个目录都有一个`runtime.properties`文件，其中包含与目录相对应的特定德鲁伊服务的配置属性（例如`historical`）。

这些`jvm.config`文件包含JVM标志，例如每个服务的堆大小调整属性。

所有服务共享的公共属性都放在`_common/common.runtime.properties`。

## 常见配置

这描述了所有德鲁伊节点共享的通用配置。可以在`common.runtime.properties`文件中定义这些配置。

### JVM配置最佳实践

我们在所有进程中设置了四个JVM参数：

1. `-Duser.timezone=UTC`这会将JVM的默认时区设置为UTC。我们总是设置它并且不测试其他默认时区，因此本地时区可能有用，但它们也可能发现奇怪而有趣的错误。要在非UTC时区中发出查询，请参阅[查询粒度](http://druid.io/docs/0.12.3/querying/granularities.html#period-granularities)
2. `-Dfile.encoding=UTF-8`这类似于时区，我们测试假设为UTF-8。本地编码可能有效，但它们也可能导致奇怪而有趣的错误。
3. `-Djava.io.tmpdir=<a path>`与文件系统交互的系统的各个部分通过临时文件来完成，并且这些文件可能会变得有些大。许多生产系统被设置为具有小（但快速）的`/tmp`目录，这对于德鲁伊来说可能是有问题的，因此我们建议将JVM的tmp目录指向具有更多肉的东西。
4. `-Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager` 这允许log4j2处理使用标准java日志记录的非log4j2组件（如jetty）的日志。

### 扩展

Druid的许多外部依赖项都可以作为模块插入。可以使用以下配置提供扩展：

| 属性                                              | 描述                                                         | 默认                                                 |
| ------------------------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------- |
| `druid.extensions.directory`                      | 用户可以在其中放置扩展相关文件的根扩展目录。Druid将加载存储在此目录下的扩展名。 | `extensions` （这是德鲁伊工作目录的相对路径）        |
| `druid.extensions.hadoopDependenciesDir`          | root hadoop依赖项目录，用户可以在其中放入与hadoop相关的依赖项文件。Druid将根据hadoop索引任务中指定的hadoop坐标加载依赖项。 | `hadoop-dependencies` （这是德鲁伊工作目录的相对路径 |
| `druid.extensions.loadList`                       | 由Druid从扩展目录加载的JSON扩展数组。如果未指定，则其值为，`null`并且德鲁伊将加载所有扩展名`druid.extensions.directory`。如果其值为空列表`[]`，则根本不会加载任何扩展名。还允许指定未存储在公共扩展目录中的其他自定义扩展的绝对路径。 | 空值                                                 |
| `druid.extensions.searchCurrentClassloader`       | 这是一个布尔标志，用于确定Druid是否会在主类加载器中搜索扩展名。它默认为true，但如果您有理由不在类路径上自动添加所有模块，则可以将其关闭。 | 真正                                                 |
| `druid.extensions.hadoopContainerDruidClasspath`  | Hadoop Indexing启动hadoop作业，此配置提供了显式设置hadoop作业的用户类路径的方法。默认情况下，这是由德鲁伊根据德鲁伊进程类路径和扩展集自动计算的。但是，有时您可能希望明确解决德鲁伊和hadoop之间的依赖冲突。 | 空值                                                 |
| `druid.extensions.addExtensionsToHadoopContainer` | 仅适用于`druid.extensions.hadoopContainerDruidClasspath`提供的情况。如果设置为true，则loadList中指定的扩展名将添加到hadoop容器类路径中。请注意，如果`druid.extensions.hadoopContainerDruidClasspath`未提供，则始终将扩展添加到hadoop容器类路径中。 | 假                                                   |

### 模块

| 属性                        | 描述                                                         | 默认 |
| --------------------------- | ------------------------------------------------------------ | ---- |
| `druid.modules.excludeList` | `"io.druid.somepackage.SomeModule"`不应加载的模块类的规范类名称（例如）的JSON数组，即使它们在指定的扩展中找到`druid.extensions.loadList`，也可以在指定要加载到特定Druid节点类型的核心模块列表中找到。当一些有用的扩展包含一些模块时很有用，不应该在某些Druid节点类型上加载，因为无法满足该模块的某些依赖性。 | []   |

### 动物园管理员

我们建议只设置基本ZK路径和ZK服务主机，但德鲁伊使用的所有ZK路径都可以覆盖到绝对路径。

| 属性                    | 描述                                                         | 默认     |
| ----------------------- | ------------------------------------------------------------ | -------- |
| `druid.zk.paths.base`   | 基础Zookeeper路径。                                          | `/druid` |
| `druid.zk.service.host` | ZooKeeper主机连接到。这是一个REQUIRED属性，因此必须提供主机地址。 | 没有     |

#### Zookeeper行为

| 属性                                | 描述                                                         | 默认    |
| ----------------------------------- | ------------------------------------------------------------ | ------- |
| `druid.zk.service.sessionTimeoutMs` | ZooKeeper会话超时，以毫秒为单位。                            | `30000` |
| `druid.zk.service.compress`         | 是否应压缩创建的Znodes的布尔标志。                           | `true`  |
| `druid.zk.service.acl`              | 是否为ZooKeeper启用ACL安全性的布尔标志。如果启用了ACL，则zNode创建者将拥有所有权限。 | `false` |

#### 路径配置

德鲁伊通过一组标准路径配置与ZK交互。我们建议只设置基本ZK路径，但德鲁伊使用的所有ZK路径都可以覆盖到绝对路径。

| 属性                                | 描述                                          | 默认                                    |
| ----------------------------------- | --------------------------------------------- | --------------------------------------- |
| `druid.zk.paths.base`               | 基础Zookeeper路径。                           | `/druid`                                |
| `druid.zk.paths.propertiesPath`     | Zookeeper属性路径。                           | `${druid.zk.paths.base}/properties`     |
| `druid.zk.paths.announcementsPath`  | 德鲁伊节点公告路径。                          | `${druid.zk.paths.base}/announcements`  |
| `druid.zk.paths.liveSegmentsPath`   | 德鲁伊节点宣布其片段的当前路径。              | `${druid.zk.paths.base}/segments`       |
| `druid.zk.paths.loadQueuePath`      | 此处的条目会导致历史节点加载和删除段。        | `${druid.zk.paths.base}/loadQueue`      |
| `druid.zk.paths.coordinatorPath`    | 协调员用于领导选举。                          | `${druid.zk.paths.base}/coordinator`    |
| `druid.zk.paths.servedSegmentsPath` | @Deprecated。德鲁伊节点宣布其细分的遗留路径。 | `${druid.zk.paths.base}/servedSegments` |

索引服务还使用自己的一组路径。这些配置可以包含在通用配置中。

| 属性                                       | 描述                       | 默认                                           |
| ------------------------------------------ | -------------------------- | ---------------------------------------------- |
| `druid.zk.paths.indexer.base`              | 基地动物园管理员路径       | `${druid.zk.paths.base}/indexer`               |
| `druid.zk.paths.indexer.announcementsPath` | 中层经理在这里宣布自己。   | `${druid.zk.paths.indexer.base}/announcements` |
| `druid.zk.paths.indexer.tasksPath`         | 用于将任务分配给中层经理。 | `${druid.zk.paths.indexer.base}/tasks`         |
| `druid.zk.paths.indexer.statusPath`        | 任务状态公告的父路径。     | `${druid.zk.paths.indexer.base}/status`        |

如果`druid.zk.paths.base`和`druid.zk.paths.indexer.base`都已设置，并且没有设置其他值`druid.zk.paths.*`或任何`druid.zk.paths.indexer.*`值，则将相对于它们各自的属性评估其他属性`base`。例如，如果`druid.zk.paths.base`设置为`/druid1`并`druid.zk.paths.indexer.base`设置为`/druid2`随后`druid.zk.paths.announcementsPath`将默认为`/druid1/announcements`，同时`druid.zk.paths.indexer.announcementsPath`将默认`/druid2/announcements`。

以下路径用于服务发现。它**不会**影响`druid.zk.paths.base`和**必须**另行规定。

| 属性                           | 描述                              | 默认               |
| ------------------------------ | --------------------------------- | ------------------ |
| `druid.discovery.curator.path` | 服务在此ZooKeeper路径下宣布自己。 | `/druid/discovery` |

### 参展商

[参展商](https://github.com/Netflix/exhibitor/wiki)是ZooKeeper的主管系统。参展商可以动态扩大/缩小ZooKeeper服务器集群。德鲁伊可以通过参展商更新自己的ZooKeeper服务器列表而无需重新启动。也就是说，它允许德鲁伊保持参展商监督的ZooKeeper服务器的连接。

| 属性                                  | 描述                                                         | 默认                         |
| ------------------------------------- | ------------------------------------------------------------ | ---------------------------- |
| `druid.exhibitor.service.hosts`       | 一个JSON数组，包含Exhibitor实例的主机名。如果您想使用参展商监督的集群，请指定此属性。 | 没有                         |
| `druid.exhibitor.service.port`        | 用于连接参展商的REST端口。                                   | `8080`                       |
| `druid.exhibitor.service.restUriPath` | 用于获取服务器集的REST调用的路径。                           | `/exhibitor/v1/cluster/list` |
| `druid.exhibitor.service.useSsl`      | 是否使用https协议的布尔标志。                                | `false`                      |
| `druid.exhibitor.service.pollingMs`   | 如何评估参展商的名单                                         | `10000`                      |

请注意，`druid.zk.service.host`如果无法联系参展商实例，则将其用作备份，因此仍应进行设置。

### TLS

#### 一般配置

| 属性                        | 描述                   | 默认    |
| --------------------------- | ---------------------- | ------- |
| `druid.enablePlaintextPort` | 启用/禁用HTTP连接器。  | `true`  |
| `druid.enableTlsPort`       | 启用/禁用HTTPS连接器。 | `false` |

虽然不推荐，但可以同时启用HTTP和HTTPS连接器，并且可以使用每个节点上的`druid.plaintextPort` 和`druid.tlsPort`属性配置相应的端口。请参阅`Configuration`各个节点的部分以检查这些端口的有效值和默认值。

#### Jetty服务器TLS配置

Druid使用Jetty作为嵌入式Web服务器。要熟悉一般的TLS / SSL以及证书等相关概念，阅读本[Jetty文档](http://www.eclipse.org/jetty/documentation/9.3.x/configuring-ssl.html)可能会有所帮助。要获得有关Java中TLS / SSL支持的更多深入知识，请参阅本[指南](http://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html)。[此处](http://www.eclipse.org/jetty/documentation/9.3.x/configuring-ssl.html#configuring-sslcontextfactory)的文档 有助于理解下面列出的TLS / SSL配置。本[文档](http://docs.oracle.com/javase/8/docs/technotes/guides/security/StandardNames.html)列出了Java实现提供的下述配置的所有可能值。

| 属性                                  | 描述                                                         | 默认 | 需要 |
| ------------------------------------- | ------------------------------------------------------------ | ---- | ---- |
| `druid.server.https.keyStorePath`     | TLS / SSL密钥库的文件路径或URL。                             | 没有 | 是   |
| `druid.server.https.keyStoreType`     | 密钥库的类型。                                               | 没有 | 是   |
| `druid.server.https.certAlias`        | 连接器的TLS / SSL证书别名。                                  | 没有 | 是   |
| `druid.server.https.keyStorePassword` | 密钥库的[密码提供程序](http://druid.io/docs/0.12.3/operations/password-provider.html)或字符串密码。 | 没有 | 是   |

下表包含非强制性高级配置选项，请谨慎使用。

| 属性                                            | 描述                                                         | 默认                                                    | 需要 |
| ----------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------- | ---- |
| `druid.server.https.keyManagerFactoryAlgorithm` | 用于创建KeyManager的算法，[这里有](https://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/JSSERefGuide.html#KeyManager)更多细节。 | `javax.net.ssl.KeyManagerFactory.getDefaultAlgorithm()` | 没有 |
| `druid.server.https.keyManagerPassword`         | 密钥管理器的[密码提供程序](http://druid.io/docs/0.12.3/operations/password-provider.html)或字符串密码。 | 没有                                                    | 没有 |
| `druid.server.https.includeCipherSuites`        | 要包含的密码套件名称列表。您可以使用确切的密码套件名称或正则表达式。 | Jetty的默认包括密码列表                                 | 没有 |
| `druid.server.https.excludeCipherSuites`        | 要排除的密码套件名称列表。您可以使用确切的密码套件名称或正则表达式。 | Jetty的默认排除密码列表                                 | 没有 |
| `druid.server.https.includeProtocols`           | 要包含的确切协议名称列表。                                   | Jetty的默认包括协议列表                                 | 没有 |
| `druid.server.https.excludeProtocols`           | 要排除的确切协议名称列表。                                   | Jetty的默认排除协议列表                                 | 没有 |

#### 内部客户端TLS配置（需要`simple-client-sslcontext`扩展）

| 属性                                     | 描述                                                         | 默认                                                      | 需要 |
| ---------------------------------------- | ------------------------------------------------------------ | --------------------------------------------------------- | ---- |
| `druid.client.https.protocol`            | 要使用的SSL协议。                                            | `TLSv1.2`                                                 | 没有 |
| `druid.client.https.trustStoreType`      | 存储受信任根证书的密钥库的类型。                             | `java.security.KeyStore.getDefaultType()`                 | 没有 |
| `druid.client.https.trustStorePath`      | 存储受信任根证书的TLS / SSL密钥库的文件路径或URL。           | 没有                                                      | 是   |
| `druid.client.https.trustStoreAlgorithm` | TrustManager用于验证证书链的算法                             | `javax.net.ssl.TrustManagerFactory.getDefaultAlgorithm()` | 没有 |
| `druid.client.https.trustStorePassword`  | Trust Store 的[密码提供程序](http://druid.io/docs/0.12.3/operations/password-provider.html)或字符串密码。 | 没有                                                      | 是   |

本[文档](http://docs.oracle.com/javase/8/docs/technotes/guides/security/StandardNames.html)列出了Java实现提供的上述配置的所有可能值。

### 身份验证和授权

| 属性                                         | 类型           | 描述                                                         | 默认         | 需要 |
| -------------------------------------------- | -------------- | ------------------------------------------------------------ | ------------ | ---- |
| `druid.auth.authenticationChain`             | JSON字符串列表 | Authenticator类型名称列表                                    | [“允许全部”] | 没有 |
| `druid.escalator.type`                       | 串             | 应该用于内部德鲁伊通信的自动扶梯的类型。此自动扶梯必须使用身份验证器支持的身份验证方案`druid.auth.authenticationChain`。 | “空操作”     | 没有 |
| `druid.auth.authorizers`                     | JSON字符串列表 | 授权者类型名称列表                                           | [“允许全部”] | 没有 |
| `druid.auth.allowUnauthenticatedHttpOptions` | 布尔           | 如果为true，则跳过HTTP OPTIONS请求的身份验证检查。某些用例需要这样做，例如支持CORS飞行前请求。请注意，禁用OPTIONS请求的身份验证检查将允许未经身份验证的用户确定哪些Druid端点有效（通过检查OPTIONS请求是否返回200而不是404），因此启用此选项可能会显示有关服务器配置的信息，包括有关哪些扩展的信息加载（如果这些扩展添加端点）。 | 假           | 没有 |

有关详细信息，请参阅[身份验证和授权](http://druid.io/docs/0.12.3/design/auth.html)。

有关特定身份验证扩展的配置选项，请参阅扩展文档。

### 启动日志记录

所有节点都可以在启动时记录调试信息。

| 属性                                   | 描述                                                         | 默认     |
| -------------------------------------- | ------------------------------------------------------------ | -------- |
| `druid.startup.logging.logProperties`  | 在启动时记录所有属性（来自common.runtime.properties，runtime.properties和JVM命令行）。 | 假       |
| `druid.startup.logging.maskProperties` | 屏蔽包含这些单词的敏感属性（例如密码）。                     | [“密码”] |

请注意，如果启用了这些设置，则可能会记录一些敏感信息。

### 请求记录

所有可以提供查询的节点也可以记录他们看到的查询请求。

| 属性                         | 描述                                                         | 默认   |
| ---------------------------- | ------------------------------------------------------------ | ------ |
| `druid.request.logging.type` | 选择：noop，file，emitter，slf4j，filtered，composing。如何记录每个查询请求。 | 空操作 |

请注意，您可以通过将“io.druid.jetty.RequestLog”设置为DEBUG级别来启用将所有HTTP请求发送到日志。请参阅[记录](http://druid.io/docs/0.12.3/configuration/logging.html)

#### 文件请求记录

每日请求日志存储在磁盘上。

| 属性                        | 描述                                                         | 默认 |
| --------------------------- | ------------------------------------------------------------ | ---- |
| `druid.request.logging.dir` | 历史，实时和代理节点维护它们获得的所有请求的请求日志（interacton是通过POST，因此正常的请求日志通常不会捕获有关实际查询的信息），这指定了存储请求日志的目录。 | 没有 |

#### 发射器请求记录

每个请求都会发送到某个外部位置。

| 属性                         | 描述             | 默认 |
| ---------------------------- | ---------------- | ---- |
| `druid.request.logging.feed` | 请求的Feed名称。 | 没有 |

#### SLF4J请求记录

每个请求都通过SLF4J记录。无论SJF4J格式规范如何，查询都会在日志消息中序列化为JSON。他们将被记录在课堂下`io.druid.server.log.LoggingRequestLogger`。

| 属性                                  | 描述                                                         | 默认 |
| ------------------------------------- | ------------------------------------------------------------ | ---- |
| `druid.request.logging.setMDC`        | 如果应在日志条目中设置MDC条目。您的日志记录设置仍然必须配置为处理MDC以格式化此数据 | 假   |
| `druid.request.logging.setContextMDC` | 如果`context`应该将druid查询添加到MDC条目中。除非`setMDC`是没有效果`true` | 假   |

填充MDC字段`setMDC`：

| MDC领域          | 描述                 |
| ---------------- | -------------------- |
| `queryId`        | 查询ID               |
| `dataSource`     | 查询所针对的数据源   |
| `queryType`      | 查询的类型           |
| `hasFilters`     | 如果查询有任何过滤器 |
| `remoteAddr`     | 请求客户端的远程地址 |
| `duration`       | 查询间隔的持续时间   |
| `resultOrdering` | 结果的排序           |
| `descending`     | 如果查询是降序查询   |

#### 过滤的请求记录

Filtered Request Logger根据可配置的查询/时间阈值过滤请求。仅发出查询/时间高于阈值的请求日志。

| 属性                                         | 描述                            | 默认        |
| -------------------------------------------- | ------------------------------- | ----------- |
| `druid.request.logging.queryTimeThresholdMs` | 查询/时间的阈值，以毫秒为单位。 | 0即没有过滤 |
| `druid.request.logging.delegate`             | 委托请求记录器来记录请求。      | 没有        |

#### 复合请求记录

Composite Request Logger将请求日志发送到多个请求记录器。

| 属性                                    | 描述                           | 默认 |
| --------------------------------------- | ------------------------------ | ---- |
| `druid.request.logging.loggerProviders` | 发出请求日志的请求记录器列表。 | 没有 |

### 启用指标

德鲁伊节点定期发布指标，并且可以包括不同的指标监视器。每个节点都可以覆盖默认的监视器列表。

| 属性                              | 描述                                                         | 默认             |
| --------------------------------- | ------------------------------------------------------------ | ---------------- |
| `druid.monitoring.emissionPeriod` | 指标的发放频率。                                             | PT1M             |
| `druid.monitoring.monitors`       | 设置节点使用的德鲁伊监视器列表。请参阅下面的名称和更多信息。例如，您可以为Broker指定监视器`druid.monitoring.monitors=["io.druid.java.util.metrics.SysMonitor","io.druid.java.util.metrics.JvmMonitor"]`。 | 无（没有监视器） |

可以使用以下监视器：

| 名称                                                   | 描述                                                         |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| `io.druid.client.cache.CacheMonitor`                   | 发出有关Historical和Broker节点的段结果缓存的度量标准（到日志）。报告典型的缓存统计信息包括命中，未命中，速率和大小（字节和条目数），以及超时和错误。 |
| `io.druid.java.util.metrics.SysMonitor`                | 这使用[SIGAR库](http://www.hyperic.com/products/sigar)来报告各种系统活动和状态。 |
| `io.druid.server.metrics.HistoricalMetricsMonitor`     | 报告历史节点的统计信息。                                     |
| `io.druid.java.util.metrics.JvmMonitor`                | 报告各种与JVM相关的统计信息。                                |
| `io.druid.java.util.metrics.JvmCpuMonitor`             | 报告JVM的CPU消耗统计信息。                                   |
| `io.druid.java.util.metrics.CpuAcctDeltaMonitor`       | 报告根据cpuacct cgroup消耗CPU。                              |
| `io.druid.java.util.metrics.JvmThreadsMonitor`         | 报告JVM中的线程统计信息，例如总计，守护程序，已启动，已死亡线程的数量。 |
| `io.druid.segment.realtime.RealtimeMetricsMonitor`     | 报告实时节点的统计信息。                                     |
| `io.druid.server.metrics.EventReceiverFirehoseMonitor` | 报告EventReceiverFirehose中已排队的事件数。                  |
| `io.druid.server.metrics.QueryCountStatsMonitor`       | 报告已成功/失败/中断的查询数量。                             |
| `io.druid.server.emitter.HttpEmitterMonitor`           | 报告内部指标`http`或`parametrized`发射器（见下文）。不得与其他发射器类型一起使用。请参阅此处的指标说明：https：//github.com/druid-io/druid/pull/4973。 |

### 发出指标

德鲁伊服务器通过我们称之为发射器的东西[发出各种指标](http://druid.io/docs/0.12.3/operations/metrics.html)和警报。代码中包含三个发射器实现，一个“noop”发射器，一个只记录到log4j（“logging”，如果没有指定发射器则默认使用）和一个将JSON事件的POST发送到服务器（ “HTTP”）。使用日志记录发射器的属性如下所述。

| 属性            | 描述                                                         | 默认   |
| --------------- | ------------------------------------------------------------ | ------ |
| `druid.emitter` | 将此值设置为“noop”，“logging”，“http”或“parametrized”将初始化其中一个发射器模块。值“组合”可用于初始化多个发射器模块。 | 空操作 |

#### 记录发射器模块

| 属性                                | 描述                                                         | 默认           |
| ----------------------------------- | ------------------------------------------------------------ | -------------- |
| `druid.emitter.logging.loggerClass` | 选择：HttpPostEmitter，LoggingEmitter，NoopServiceEmitter，ServiceEmitter。用于记录的类。 | LoggingEmitter |
| `druid.emitter.logging.logLevel`    | 选择：调试，信息，警告，错误。记录消息的日志级别。           | 信息           |

#### Http发射器模块

| 属性                                      | 描述                                                         | 默认                                                  |
| ----------------------------------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
| `druid.emitter.http.flushMillis`          | 刷新内部消息缓冲区的频率（发送数据）。                       | 60000                                                 |
| `druid.emitter.http.flushCount`           | 在刷新（发送）之前，内部消息缓冲区可以容纳多少条消息。       | 500                                                   |
| `druid.emitter.http.basicAuthentication`  | 以“登录：密码”形式进行身份验证的登录名和密码，例如`druid.emitter.http.basicAuthentication=admin:adminpassword` | 未指定=无认证                                         |
| `druid.emitter.http.flushTimeOut          | 应该将事件发送到端点的超时（即使未填充内部缓冲区），以毫秒为单位。 | 未指定=没有超时                                       |
| `druid.emitter.http.batchingStrategy`     | 批量格式化的策略。“ARRAY”表示`[event1,event2]`“NEWLINES”表示`event1\nevent2`ONLY_EVENTS表示`event1event2`。 | ARRAY                                                 |
| `druid.emitter.http.maxBatchSize`         | 最大批量大小（以字节为单位）。                               | 最小值（JVM堆大小的10％除以2）或（5191680（即5 MB）） |
| `druid.emitter.http.batchQueueSizeLimit`  | 发射器队列中的最大批次数，如果有发射问题。                   | 最大值为（2）或（JVM堆大小的10％除以5MB）             |
| `druid.emitter.http.minHttpTimeoutMillis` | 如果填充批次的速度超过了小于此的速度，甚至没有尝试将批次发送到端点，因为它可能会失败，无法快速发送数据。根据emitter / successfulSending / minTimeMs指标配置此项。合理的值是10ms..100ms。 | 0                                                     |
| `druid.emitter.http.recipientBaseUrl`     | 发送消息的基本URL。Druid将POST JSON在此属性指定的HTTP端点处使用。 | 无，必需的配置                                        |

#### 参数化Http发射器模块

`druid.emitter.parametrized.httpEmitting.*`配置对应于Http Emitter Modules的配置，见上文。除外`recipientBaseUrl`。E. g。`druid.emitter.parametrized.httpEmitting.flushMillis`， `druid.emitter.parametrized.httpEmitting.flushCount`等等。

额外的配置是：

| 属性                                                 | 描述                                                         | 默认           |
| ---------------------------------------------------- | ------------------------------------------------------------ | -------------- |
| `druid.emitter.parametrized.recipientBaseUrlPattern` | 基于事件的Feed发送事件的URL模式。E. g。`http://foo.bar/{feed}`，`http://foo.bar/metrics`如果事件的提要是“指标” ，那将发送事件。 | 无，必需的配置 |

#### 编写发射器模块

| 属性                               | 描述                                              | 默认 |
| ---------------------------------- | ------------------------------------------------- | ---- |
| `druid.emitter.composing.emitters` | 要加载的发射器模块列表，例如[“logging”，“http”]。 | []   |

#### 石墨发射器

使用石墨作为发射器组`druid.emitter=graphite`。有关配置详情，请点击此[链接](http://druid.io/docs/0.12.3/development/extensions-contrib/graphite.html)。

### 元数据存储

这些属性指定元数据存储周围的jdbc连接和其他配置。使用这些属性连接到元数据存储的唯一进程是[协调器](http://druid.io/docs/0.12.3/design/coordinator.html)，[索引服务](http://druid.io/docs/0.12.3/design/indexing-service.html)和[实时节点](http://druid.io/docs/0.12.3/design/realtime.html)。

| 属性                                            | 描述                                                         | 默认              |
| ----------------------------------------------- | ------------------------------------------------------------ | ----------------- |
| `druid.metadata.storage.type`                   | 要使用的元数据存储的类型。从“mysql”，“postgresql”或“derby”中选择。 | 德比              |
| `druid.metadata.storage.connector.connectURI`   | 用于数据库连接的jdbc uri                                     | 没有              |
| `druid.metadata.storage.connector.user`         | 要连接的用户名。                                             | 没有              |
| `druid.metadata.storage.connector.password`     | 用于连接的[密码提供程序](http://druid.io/docs/0.12.3/operations/password-provider.html)或字符串密码。 | 没有              |
| `druid.metadata.storage.connector.createTables` | 如果德鲁伊需要一张桌子并且它不存在，那么创建它吧？           | 真正              |
| `druid.metadata.storage.tables.base`            | 表的基本名称。                                               | 德鲁伊            |
| `druid.metadata.storage.tables.segments`        | 用于查找段的表。                                             | druid_segments    |
| `druid.metadata.storage.tables.rules`           | 用于查找段加载/删除规则的表。                                | druid_rules       |
| `druid.metadata.storage.tables.config`          | 用于查找配置的表。                                           | druid_config      |
| `druid.metadata.storage.tables.tasks`           | 索引服务用于存储任务。                                       | druid_tasks       |
| `druid.metadata.storage.tables.taskLog`         | 索引服务用于存储任务日志。                                   | druid_taskLog     |
| `druid.metadata.storage.tables.taskLock`        | 索引服务用于存储任务锁。                                     | druid_taskLock    |
| `druid.metadata.storage.tables.supervisors`     | 索引服务用于存储管理程序配置。                               | druid_supervisors |
| `druid.metadata.storage.tables.audit`           | 用于配置更改的审计历史记录的表，例如协调器规则。             | druid_audit       |

### 深度存储

这些配置涉及如何从深层存储中推送和拉取[分段](http://druid.io/docs/0.12.3/design/segments.html)。

| 属性                 | 描述                                                    | 默认 |
| -------------------- | ------------------------------------------------------- | ---- |
| `druid.storage.type` | 选择：本地，noop，s3，hdfs，c *。要使用的深层存储类型。 | 本地 |

#### 本地深度存储

本地深度存储使用本地文件系统。

| 属性                             | 描述                         | 默认                        |
| -------------------------------- | ---------------------------- | --------------------------- |
| `druid.storage.storageDirectory` | 磁盘上的目录，用作深度存储。 | 的/ tmp /德/ localStorage的 |

#### Noop Deep Storage

这个深层存储没有做任何事情。没有配置。

#### S3深度存储

这个深度存储用于与亚马逊S3的接口。请注意，`druid-s3-extensions`必须加载扩展名。

| 属性                           | 描述                                             | 默认 |
| ------------------------------ | ------------------------------------------------ | ---- |
| `druid.s3.accessKey`           | 用于访问S3的访问密钥。                           | 没有 |
| `druid.s3.secretKey`           | 用于访问S3的密钥。                               | 没有 |
| `druid.storage.bucket`         | S3存储桶名称。                                   | 没有 |
| `druid.storage.baseKey`        | S3对象密钥前缀用于存储。                         | 没有 |
| `druid.storage.disableAcl`     | ACL的布尔标志。                                  | 假   |
| `druid.storage.archiveBucket`  | 运行索引服务*归档任务*时用于归档的S3存储桶名称。 | 没有 |
| `druid.storage.archiveBaseKey` | 用于存档的S3对象键前缀。                         | 没有 |

#### HDFS深度存储

此深存储用于与HDFS连接。请注意，`druid-hdfs-storage`必须加载扩展名。

| 属性                             | 描述                   | 默认 |
| -------------------------------- | ---------------------- | ---- |
| `druid.storage.storageDirectory` | HDFS目录用作深层存储。 | 没有 |

#### Cassandra深层存储

这个深度存储用于与Cassandra连接。请注意，`druid-cassandra-storage`必须加载扩展名。

| 属性                     | 描述                 | 默认 |
| ------------------------ | -------------------- | ---- |
| `druid.storage.host`     | 卡桑德拉主持人。     | 没有 |
| `druid.storage.keyspace` | 卡桑德拉的关键空间。 | 没有 |

### 任务记录

如果以远程模式运行索引服务，则任务日志必须存储在S3，Azure Blob Store，Google Cloud Storage或HDFS中。

| 属性                      | 描述                                                         | 默认 |
| ------------------------- | ------------------------------------------------------------ | ---- |
| `druid.indexer.logs.type` | 选择：noop，s3，azure，google，hdfs，file。存储任务日志的位置 | 文件 |

您还可以通过配置以下附加属性，将Overlord配置为仅在最后x毫秒内自动保留任务日志。警告：自动日志文件删除通常基于后备存储上的日志文件修改时间戳，因此德鲁伊节点和后备存储节点之间的大时钟偏差可能会导致意外行为。

| 属性                                       | 描述                                                         | 默认                      |
| ------------------------------------------ | ------------------------------------------------------------ | ------------------------- |
| `druid.indexer.logs.kill.enabled`          | 是否启用删除旧任务日志的布尔值。                             | 假                        |
| `druid.indexer.logs.kill.durationToRetain` | 如果启用了kill，则为必需。以毫秒为单位，在最后x毫秒内创建要保留的任务日志。 | 没有                      |
| `druid.indexer.logs.kill.initialDelay`     | 可选的。运行第一次自动终止后，霸王启动后的毫秒数。           | 随机值小于300000（5分钟） |
| `druid.indexer.logs.kill.delay`            | 可选的。连续执行自动终止运行之间的延迟毫秒数。               | 21600000（6小时）         |

#### 文件任务日志

将任务日志存储在本地文件系统中。

| 属性                           | 描述               | 默认 |
| ------------------------------ | ------------------ | ---- |
| `druid.indexer.logs.directory` | 本地文件系统路径。 | 日志 |

#### S3任务日志

将任务日志存储在S3中。请注意，`druid-s3-extensions`必须加载扩展名。

| 属性                          | 描述           | 默认 |
| ----------------------------- | -------------- | ---- |
| `druid.indexer.logs.s3Bucket` | S3存储桶名称。 | 没有 |
| `druid.indexer.logs.s3Prefix` | S3键前缀。     | 没有 |

#### Azure Blob存储任务日志

将任务日志存储在Azure Blob Store中。

注意：`druid-azure-extensions`必须加载扩展，并且这使用与azure的深层存储模块相同的存储帐户。

| 属性                           | 描述                               | 默认 |
| ------------------------------ | ---------------------------------- | ---- |
| `druid.indexer.logs.container` | 要将日志写入的Azure Blob Store容器 | 没有 |
| `druid.indexer.logs.prefix`    | 预先添加到日志的路径               | 没有 |

#### Google云端存储任务日志

将任务日志存储在Google云端存储中。

注意：`druid-google-extensions`必须加载扩展程序，并且这使用与Google的深层存储模块相同的存储设置。

| 属性                        | 描述                                       | 默认 |
| --------------------------- | ------------------------------------------ | ---- |
| `druid.indexer.logs.bucket` | 用于将日志写入的Google Cloud Storage存储桶 | 没有 |
| `druid.indexer.logs.prefix` | 预先添加到日志的路径                       | 没有 |

#### HDFS任务日志

将任务日志存储在HDFS中。请注意，`druid-hdfs-storage`必须加载扩展名。

| 属性                           | 描述             | 默认 |
| ------------------------------ | ---------------- | ---- |
| `druid.indexer.logs.directory` | 存储日志的目录。 | 没有 |

### 索引服务发现

此配置用于使用Curator服务发现查找[索引服务](http://druid.io/docs/0.12.3/design/indexing-service.html)。仅在您实际运行索引服务时才需要。

| 属性                                   | 描述                                                         | 默认        |
| -------------------------------------- | ------------------------------------------------------------ | ----------- |
| `druid.selectors.indexing.serviceName` | 索引服务Overlord节点的druid.service名称。要使用其他名称启动Overlord，请使用此属性进行设置。 | 德鲁伊/霸主 |

### 协调员发现

此配置用于使用Curator服务发现查找[协调](http://druid.io/docs/0.12.3/design/coordinator.html)器。实时索引节点使用此配置来获取有关群集中加载的段的信息。

| 属性                                      | 描述                                                         | 默认          |
| ----------------------------------------- | ------------------------------------------------------------ | ------------- |
| `druid.selectors.coordinator.serviceName` | 协调器节点的druid.service名称。要使用其他名称启动协调器，请使用此属性进行设置。 | 德鲁伊/协调员 |

### 宣布细分

您可以配置如何在ZooKeeper中宣布和取消激活Znodes（使用Curator）。对于正常操作，您不需要覆盖任何这些配置。

##### 批量数据段播音员

在当前的德鲁伊中，可以在相同的Znode下宣布多个数据段。

| 属性                                       | 描述                                                         | 默认   |
| ------------------------------------------ | ------------------------------------------------------------ | ------ |
| `druid.announcer.segmentsPerNode`          | 每个Znode包含最多这些段的信息。                              | 50     |
| `druid.announcer.maxBytesPerNode`          | Znode的最大字节大小。                                        | 524288 |
| `druid.announcer.skipDimensionsAndMetrics` | 从细分公告中跳过维度和指标列表。注意：启用此选项还将从协调器和代理端点中删除维度和度量标准列表。 | 假     |
| `druid.announcer.skipLoadSpec`             | 从细分公告中跳过细分市场LoadSpec。注意：启用此选项还将从协调程序和代理端点中删除loadspec。 | 假     |

### JavaScript的

Druid通过JavaScript函数支持动态运行时扩展。可以通过以下属性配置此功能。

| 属性                       | 描述                                                         | 默认 |
| -------------------------- | ------------------------------------------------------------ | ---- |
| `druid.javascript.enabled` | 设置为“true”以启用JavaScript功能。这会影响JavaScript解析器，过滤器，extractFn，聚合器，后聚合器，路由器策略和工作者选择策略。 | 假   |

默认情况下禁用基于JavaScript的功能。有关使用德鲁伊JavaScript功能的[指南](http://druid.io/docs/0.12.3/development/javascript.html)，请参阅德鲁伊[JavaScript编程指南](http://druid.io/docs/0.12.3/development/javascript.html)，包括如何启用它的说明。

### 双列存储

Druid的存储层使用32位浮点表示来存储由索引时的doubleSum，doubleMin和doubleMax聚合器创建的列。要为这些列使用64位浮点数，请设置系统范围的属性`druid.indexing.doubleStorage=double`。这将成为未来德鲁伊版本中的默认行为。

| 属性                           | 描述                                     | 默认 |
| ------------------------------ | ---------------------------------------- | ---- |
| `druid.indexing.doubleStorage` | 设置为“double”以对双列使用64位双重表示。 | 浮动 |

## 协调员

有关一般协调节点信息，请参见[此处](http://druid.io/docs/0.12.3/design/coordinator.html)。

### 静态配置

可以在`coordinator/runtime.properties`文件中定义这些协调器静态配置。

#### 节点配置

| 属性                  | 描述                                                         | 默认                                                   |
| --------------------- | ------------------------------------------------------------ | ------------------------------------------------------ |
| `druid.host`          | 当前节点的主机。这用于将当前进程位置通告为可从另一节点到达，并且通常应该指定为`http://${druid.host}/`可以实际与该进程通信 | InetAddress.getLocalHost（）。getCanonicalHostName（） |
| `druid.plaintextPort` | 这是实际收听的端口; 除非使用端口映射，否则这将是与之相同的端口`druid.host` | 8081                                                   |
| `druid.tlsPort`       | HTTPS连接器的TLS端口，如果设置了[druid.enableTlsPort](http://druid.io/docs/0.12.3/operations/tls-support.html)，则将使用此配置。如果`druid.host`包含端口，则该端口将被忽略。这应该是一个非负整数。 | 8281                                                   |
| `druid.service`       | 服务的名称。在发出指标和警报以区分各种服务时，这用作维度     | 德鲁伊/协调员                                          |

#### 协调员行动

| 属性                                           | 描述                                                         | 默认              |
| ---------------------------------------------- | ------------------------------------------------------------ | ----------------- |
| `druid.coordinator.period`                     | 协调员的运行期。协调器通过在存储器中维护世界的当前状态并定期查看可用的段集和服务的段来确定是否需要对数据拓扑进行任何更改来进行操作。此属性设置每个运行之间的延迟。 | PT60S             |
| `druid.coordinator.period.indexingPeriod`      | 将索引任务发送到索引服务的频率。仅在启用合并或转换时适用。   | PT1800S（30分钟） |
| `druid.coordinator.startDelay`                 | 协调器的操作基于它运行时具有世界状态的最新视图的假设，然而，当前的ZK交互代码以不允许协调器的方式编写。知道它已经完成加载当前世界状态的事实。这种延迟是一种破解，让它有足够的时间相信它拥有所有数据。 | PT300S            |
| `druid.coordinator.merge.on`                   | 布尔标志，指示协调器是否应尝试将小段合并为更优的段大小。     | 假                |
| `druid.coordinator.conversion.on`              | 用于将旧段索引版本转换为最新段索引版本的布尔标志。           | 假                |
| `druid.coordinator.load.timeout`               | 协调器将段分配给历史节点的超时持续时间。                     | PT15M             |
| `druid.coordinator.kill.pendingSegments.on`    | 布尔标志，用于指示协调器是否清除`pendingSegments`元数据存储表中的旧条目。如果设置为true，协调器将检查最近完成的任务的创建时间。如果它不存在，它将查找earlist运行/挂起/等待任务的创建时间。一旦找到创建的时间，那么对于不在`killPendingSegmentsSkipList`（参见[动态配置](http://druid.io/docs/0.12.3/configuration/index.html#dynamic-configuration)）的所有dataSource ，协调器将要求霸主清理比`pendingSegments`表中找到的创建时间早1天或更早的条目。这将根据`druid.coordinator.period`指定定期完成。 | 假                |
| `druid.coordinator.kill.on`                    | 布尔标志，指示协调器是否应为未使用的段提交终止任务，即从元数据存储和深度存储中硬删除它们。如果设置为true，则对于所有列入白名单的dataSource（或可选的全部），协调器将根据`period`指定定期提交任务。这些终止任务将删除除最后一个`durationToRetain`句点之外的所有段。可以通过动态配置设置白名单或全部`killAllDataSources`，`killDataSourceWhitelist`稍后介绍。 | 假                |
| `druid.coordinator.kill.period`                | 将kill任务发送到索引服务的频率。价值必须大于`druid.coordinator.period.indexingPeriod`。仅在打开kill时适用。 | P1D（1天）        |
| `druid.coordinator.kill.durationToRetain`      | 不要在最后杀死段`durationToRetain`，必须大于或等于0.仅适用并且必须在kill被打开时指定。请注意，默认值无效。 | PT-1S（-1秒）     |
| `druid.coordinator.kill.maxSegments`           | 每次杀死任务提交最多杀死n个段，必须大于0.仅适用且必须指定kill是否打开。请注意，默认值无效。 | 0                 |
| `druid.coordinator.balancer.strategy`          | 指定协调员用于在历史记录之间分配段的平衡策略的类型。`cachingCost`在逻辑上相当于`cost`但在大型集群上具有更高的CPU效率，并且将`cost`在未来版本中取代，用户可以尝试使用它。用于`diskNormalized`在节点之间分配段，以便磁盘统一填充并用于`random`随机选择节点以分配段。 | `cost`            |
| `druid.coordinator.loadqueuepeon.repeatDelay`  | loadqueuepeon的启动和重复延迟，用于管理段的加载和丢弃。      | PT0.050S（50 ms） |
| `druid.coordinator.asOverlord.enabled`         | 此协调器节点是否应该像霸主一样的布尔值。此配置允许用户通过不必部署任何独立的霸主节点来简化德鲁伊群集。如果设置为true，则可以使用overlord控制台，`http://coordinator-host:port/console.html`并确保`druid.coordinator.asOverlord.overlordService`也可以设置。见下。 | 假                |
| `druid.coordinator.asOverlord.overlordService` | 必需，如果`druid.coordinator.asOverlord.enabled`是`true`。这必须与`druid.service`独立的Overlord节点和`druid.selectors.indexing.serviceName`Middle Manager上的值相同。 | 空值              |

#### 细分管理

| 属性                                   | 可能的值     | 描述                                                         | 默认 |
| -------------------------------------- | ------------ | ------------------------------------------------------------ | ---- |
| `druid.announcer.type`                 | 批处理或http | 要使用的段发现方法。“http”允许使用HTTP而不是zookeeper发现段。 | 批量 |
| `druid.coordinator.loadqueuepeon.type` | 策展人或http | 是否使用“http”或“curator”实现将段加载/删除分配给历史         | 馆长 |

##### 使用“http”loadqueuepeon时的附加配置

| 属性                                             | 描述                                                         | 默认 |
| ------------------------------------------------ | ------------------------------------------------------------ | ---- |
| `druid.coordinator.loadqueuepeon.http.batchSize` | 一个HTTP请求中批处理的段加载/删除请求数。请注意，它必须小于`druid.segmentCache.numLoadingThreads`历史节点上的config。 | 1    |

#### 元数据检索

| 属性                                  | 描述                                                         | 默认  |
| ------------------------------------- | ------------------------------------------------------------ | ----- |
| `druid.manager.config.pollDuration`   | 经理轮询配置表以获取更新的频率。                             | PT1M  |
| `druid.manager.segments.pollDuration` | 协调器进行轮询之间的持续时间，以更新活动段集。通常定义协调器注意新段可能需要的滞后时间量。 | PT1M  |
| `druid.manager.rules.pollDuration`    | 协调器执行的轮询之间的持续时间，用于更新活动规则集。通常定义协调器注意规则可能需要的滞后时间量。 | PT1M  |
| `druid.manager.rules.defaultTier`     | 将从中加载默认规则的默认层。                                 | _默认 |
| `druid.manager.rules.alertThreshold`  | 轮询失败后应发出警报的持续时间。                             | PT10M |

### 动态配置

协调器具有动态配置，可以动态更改某些行为。协调器使用Druid [元数据存储](http://druid.io/docs/0.12.3/dependencies/metadata-storage.html)配置表中的JSON规范对象。该对象详述如下：

建议您使用协调器控制台配置这些参数。但是，如果您需要通过HTTP执行此操作，可以通过POST请求将JSON对象提交给协调器：

```text
http://<COORDINATOR_IP>:<PORT>/druid/coordinator/v1/config
```

还可以指定用于审核配置更改的可选标头参数。

| 标题参数名称      | 描述                   | 默认 |
| ----------------- | ---------------------- | ---- |
| `X-Druid-Author`  | 作者进行配置更改       | “”   |
| `X-Druid-Comment` | 评论描述正在进行的更改 | “”   |

示例协调器动态配置JSON对象如下所示：

```json
{
  "millisToWaitBeforeDeleting": 900000,
  "mergeBytesLimit": 100000000,
  "mergeSegmentsLimit" : 1000,
  "maxSegmentsToMove": 5,
  "replicantLifetime": 15,
  "replicationThrottleLimit": 10,
  "emitBalancingStats": false,
  "killDataSourceWhitelist": ["wikipedia", "testDatasource"]
}
```

在同一URL发出GET请求将返回当前的规范。配置设置规范的说明如下所示。

| 属性                            | 描述                                                         | 默认             |
| ------------------------------- | ------------------------------------------------------------ | ---------------- |
| `millisToWaitBeforeDeleting`    | 在协调器开始在元数据存储中删除（标记未使用的）段之前，协调器需要多长时间才能处于活动状态。 | 900000（15分钟） |
| `mergeBytesLimit`               | 要合并的段的最大总未压缩大小（以字节为单位）。               | 524288000L       |
| `mergeSegmentsLimit`            | 单个[追加任务](http://druid.io/docs/0.12.3/ingestion/tasks.html)中可以包含的最大段数。 | 100              |
| `maxSegmentsToMove`             | 在任何给定时间可以移动的最大段数。                           | 五               |
| `replicantLifetime`             | 在我们开始警报之前，要复制的段的最大协调程序运行数。         | 15               |
| `replicationThrottleLimit`      | 一次可以复制的最大段数。                                     | 10               |
| `emitBalancingStats`            | 布尔标志，表示我们是否应该发出平衡统计数据。这是一项昂贵的操作。 | 假               |
| `killDataSourceWhitelist`       | 如果property `druid.coordinator.kill.on`为true，则为其发送kill任务的dataSource列表。这可以是以逗号分隔的dataSource或JSON数组的列表。 | 没有             |
| `killAllDataSources`            | 如果property `druid.coordinator.kill.on`为true，则为所有dataSources发送kill任务。如果将此值设置为true，则`killDataSourceWhitelist`不得指定或为空列表。 | 假               |
| `killPendingSegmentsSkipList`   | 如果property 为true，*则不*清除pendingSegments的dataSources列表`druid.coordinator.kill.pendingSegments.on`。这可以是以逗号分隔的dataSource或JSON数组的列表。 | 没有             |
| `maxSegmentsInNodeLoadingQueue` | 可以排队加载到任何给定服务器的最大段数。此参数可用于加速段加载过程，特别是如果群集中存在“慢”节点（加载速度低）或者计划将多个段复制到某个特定节点（更快的加载可能更好）细分市场）。期望值取决于段加载速度，可接受的复制时间和节点数。值1000可以是相当大的集群的起点。默认值为0（加载队列无界限） | 0                |

要查看协调器动态配置的审核历史记录，请向URL发出GET请求 -

```text
http://<COORDINATOR_IP>:<PORT>/druid/coordinator/v1/config/history?interval=<interval>
```

可以通过`druid.audit.manager.auditHistoryMillis`在coordinator runtime.properties中设置（如果未配置，则为1周）来指定interval的缺省值

查看最后一个 协调器动态配置的审计历史记录的条目向URL发出GET请求 -

```text
http://<COORDINATOR_IP>:<PORT>/druid/coordinator/v1/config/history?count=<n>
```

#### 查找动态配置（实验）

这些配置选项控制[查找页面中](http://druid.io/docs/0.12.3/querying/lookups.html)描述的查找动态配置的行为

| 属性                                      | 描述                                                         | 默认   |
| ----------------------------------------- | ------------------------------------------------------------ | ------ |
| `druid.manager.lookups.hostDeleteTimeout` | `DELETE`在考虑`DELETE`故障之前等待对特定节点的请求的时间     | PT1s   |
| `druid.manager.lookups.hostUpdateTimeout` | `POST`在考虑`POST`故障之前等待对特定节点的请求的时间         | PT10s  |
| `druid.manager.lookups.deleteAllTimeout`  | `DELETE`在考虑删除尝试失败之前等待所有请求完成多长时间       | PT10s  |
| `druid.manager.lookups.updateAllTimeout`  | `POST`在考虑尝试失败之前等待所有请求完成多长时间             | PT60s  |
| `druid.manager.lookups.threadPoolSize`    | 可以同时管理多少个节点（并发POST和DELETE请求）。请求此限制将在队列中等待，直到插槽可用。 | 10     |
| `druid.manager.lookups.period`            | 检查配置更改之间的时间间隔是多少毫秒                         | 30_000 |

## 霸王

### 霸主静态配置

可以在`overlord/runtime.properties`文件中定义这些霸主静态配置。

#### 霸主节点配置

| 属性                  | 描述                                                         | 默认                                                   |
| --------------------- | ------------------------------------------------------------ | ------------------------------------------------------ |
| `druid.host`          | 当前节点的主机。这用于将当前进程位置通告为可从另一节点到达，并且通常应该指定为`http://${druid.host}/`可以实际与该进程通信 | InetAddress.getLocalHost（）。getCanonicalHostName（） |
| `druid.plaintextPort` | 这是实际收听的端口; 除非使用端口映射，否则这将是与之相同的端口`druid.host` | 8090                                                   |
| `druid.tlsPort`       | HTTPS连接器的TLS端口，如果设置了[druid.enableTlsPort](http://druid.io/docs/0.12.3/operations/tls-support.html)，则将使用此配置。如果`druid.host`包含端口，则该端口将被忽略。这应该是一个非负整数。 | 8290                                                   |
| `druid.service`       | 服务的名称。在发出指标和警报以区分各种服务时，这用作维度     | 德鲁伊/霸主                                            |

#### 霸王行动

| 属性                                              | 描述                                                         | 默认                |
| ------------------------------------------------- | ------------------------------------------------------------ | ------------------- |
| `druid.indexer.runner.type`                       | 选择“本地”或“远程”。指示任务是应在本地运行还是在分布式环境中运行。 | 本地                |
| `druid.indexer.storage.type`                      | 选择是“本地”或“元数据”。指示传入任务是应存储在本地（在堆中）还是存储在元数据存储中。将传入任务存储在元数据存储中允许在霸主失败时恢复任务。 | 本地                |
| `druid.indexer.storage.recentlyFinishedThreshold` | 存储任务结果的持续时间。                                     | PT24H               |
| `druid.indexer.queue.maxSize`                     | 一次最多活动任务数。                                         | Integer.MAX_VALUE的 |
| `druid.indexer.queue.startDelay`                  | 在开始霸王队列管理之前要睡这么久。这可以用于在例如广泛的网络问题之后给予集群时间以重新定向自身。 | PT1M                |
| `druid.indexer.queue.restartDelay`                | 当霸王队列管理再次尝试之前抛出异常时，请长时间休眠。         | PT30S               |
| `druid.indexer.queue.storageSyncRate`             | 通常使用底层任务持久性机制来同步霸主状态。                   | PT1M                |

以下配置仅适用于霸主在远程模式下运行的情况。有关本地与远程模式的说明，请参阅（../design/indexing-service.html#overlord-node）。

| 属性                                                 | 描述                                                         | 默认   |
| ---------------------------------------------------- | ------------------------------------------------------------ | ------ |
| `druid.indexer.runner.taskAssignmentTimeout`         | 在抛出错误之前将任务分配给中间管理器后等待多长时间。         | PT5M   |
| `druid.indexer.runner.minWorkerVersion`              | 要将任务发送到的最小中间管理器版本。                         | “0”    |
| `druid.indexer.runner.compressZnodes`                | 指示霸主是否应该期望中间经理压缩Znodes。                     | 真正   |
| `druid.indexer.runner.maxZnodeBytes`                 | 可以在Zookeeper中创建的最大大小Znode（以字节为单位）。       | 524288 |
| `druid.indexer.runner.taskCleanupTimeout`            | 在中间管理器与Zookeeper断开连接之后，在失败任务之前等待多长时间。 | PT15M  |
| `druid.indexer.runner.taskShutdownLinkTimeout`       | 在超时之前等待关闭请求到中间管理器的时间                     | PT1M   |
| `druid.indexer.runner.pendingTasksRunnerNumThreads`  | 将待处理任务分配给工作线程的线程数必须至少为1。              | 1      |
| `druid.indexer.runner.maxRetriesBeforeBlacklist`     | 在工作人员列入黑名单之前，中间经理可以将任务失败的连续次数必须至少为1 | 五     |
| `druid.indexer.runner.workerBlackListBackoffTime`    | 在将任务再次列入白名单之前等待多长时间。此值应大于为taskBlackListCleanupPeriod设置的值。 | PT15M  |
| `druid.indexer.runner.workerBlackListCleanupPeriod`  | 清理线程启动以清理列入黑名单的工作程序的持续时间。           | PT5M   |
| `druid.indexer.runner.maxPercentageBlacklistWorkers` | 黑名单中工人的最大百分比，必须介于0到100之间。               | 20     |

自动缩放还有其他配置（如果已启用）：

| 属性                                         | 描述                                                         | 默认                       |
| -------------------------------------------- | ------------------------------------------------------------ | -------------------------- |
| `druid.indexer.autoscale.strategy`           | 选择是“noop”或“ec2”。设置在需要自动缩放时运行的策略。        | 空操作                     |
| `druid.indexer.autoscale.doAutoscale`        | 如果设置为“true”，则将启用自动缩放。                         | 假                         |
| `druid.indexer.autoscale.provisionPeriod`    | 多久检查是否应该添加新的中层管理人员。                       | PT1M                       |
| `druid.indexer.autoscale.terminatePeriod`    | 检查何时应删除中层管理人员的频率。                           | PT5M                       |
| `druid.indexer.autoscale.originTime`         | 终止时段递增的起始参考时间戳。                               | 2012-01-01T00：55：00.000Z |
| `druid.indexer.autoscale.workerIdleTimeout`  | 在可以考虑终止之前，工作人员可以闲置多长时间（而不是运行任务）。 | PT90M                      |
| `druid.indexer.autoscale.maxScalingDuration` | 在放弃之前，霸主会等待中层经理出现多长时间。                 | PT15M                      |
| `druid.indexer.autoscale.numEventsToTrack`   | 要跟踪的自动缩放相关事件（节点创建和终止）的数量。           | 10                         |
| `druid.indexer.autoscale.pendingTaskTimeout` | 在霸主试图扩大之前，任务可以处于“待定”状态多长时间。         | PT30S                      |
| `druid.indexer.autoscale.workerVersion`      | 如果设置，则仅在自动缩放期间创建设置版本的节点。覆盖动态配置。 | 空值                       |
| `druid.indexer.autoscale.workerPort`         | 中层经理将运行的端口。                                       | 8080                       |

### 霸王动态配置

霸主可以动态地改变工人的行为。

可以通过POST请求将JSON对象提交给霸主：

```text
http://<OVERLORD_IP>:<port>/druid/indexer/v1/worker
```

还可以指定用于审核配置更改的可选标头参数。

| 标题参数名称      | 描述                   | 默认 |
| ----------------- | ---------------------- | ---- |
| `X-Druid-Author`  | 作者进行配置更改       | “”   |
| `X-Druid-Comment` | 评论描述正在进行的更改 | “”   |

示例工作者配置规范如下所示：

```json
{
  "selectStrategy": {
    "type": "fillCapacity",
    "affinityConfig": {
      "affinity": {
        "datasource1": ["host1:port", "host2:port"],
        "datasource2": ["host3:port"]
      }
    }
  },
  "autoScaler": {
    "type": "ec2",
    "minNumWorkers": 2,
    "maxNumWorkers": 12,
    "envConfig": {
      "availabilityZone": "us-east-1a",
      "nodeData": {
        "amiId": "${AMI}",
        "instanceType": "c3.8xlarge",
        "minInstances": 1,
        "maxInstances": 1,
        "securityGroupIds": ["${IDs}"],
        "keyName": "${KEY_NAME}"
      },
      "userData": {
        "impl": "string",
        "data": "${SCRIPT_COMMAND}",
        "versionReplacementString": ":VERSION:",
        "version": null
      }
    }
  }
}
```

在同一URL发出GET请求将返回当前的当前工作者配置规范。上面的worker配置规范列表只是EC2的一个示例，可以扩展其他部署环境的代码库。worker配置规范的说明如下所示。

| 属性             | 描述                                                         | 默认              |
| ---------------- | ------------------------------------------------------------ | ----------------- |
| `selectStrategy` | 如何将任务分配给中层经理。选择是`fillCapacity`，`equalDistribution`和`javascript`。 | equalDistribution |
| `autoScaler`     | 仅在启用自动缩放时使用。见下文。                             | 空值              |

要查看worker配置的审核历史记录，请向URL发出GET请求 -

```text
http://<OVERLORD_IP>:<port>/druid/indexer/v1/worker/history?interval=<interval>
```

可以通过`druid.audit.manager.auditHistoryMillis`在overlord runtime.properties中设置（如果未配置，则为1周）来指定interval的默认值。

查看最后一个 worker配置的审计历史记录的条目向URL发出GET请求 -

```text
http://<OVERLORD_IP>:<port>/druid/indexer/v1/worker/history?count=<n>
```

#### 工人选择战略

工人选择策略控制德鲁伊如何将任务分配给中间管理员。

##### 平等分配

任务在任务开始运行时分配给具有最大可用容量的middleManager。如果您希望在中间管理器中均匀分布工作，这将非常有用。

| 属性             | 描述                                                         | 默认                            |
| ---------------- | ------------------------------------------------------------ | ------------------------------- |
| `type`           | `equalDistribution`。                                        | 需要; 一定是`equalDistribution` |
| `affinityConfig` | [亲和配置](http://druid.io/docs/0.12.3/configuration/index.html#affinity)对象 | null（无亲和力）                |

##### 填充容量

在任务开始运行时，任务将分配给具有当前正在运行的任务的工作人员。这对于弹性自动缩放中间管理器的情况很有用，因为它往往会打包一些并将其他人留空。空的可以安全终止。

请注意，如果`druid.indexer.runner.pendingTasksRunnerNumThreads`设置为*N* > 1，则此策略将同时填充*N个* 中间管理器，而不是单个中间管理器。

| 属性             | 描述                                                         | 默认                       |
| ---------------- | ------------------------------------------------------------ | -------------------------- |
| `type`           | `fillCapacity`。                                             | 需要; 一定是`fillCapacity` |
| `affinityConfig` | [亲和配置](http://druid.io/docs/0.12.3/configuration/index.html#affinity)对象 | null（无亲和力）           |

##### 使用Javascript

允许定义任意逻辑，以选择工作人员使用JavaScript函数运行任务。函数传递remoteTaskRunnerConfig，workerId的映射到可用的worker和要执行的任务，并返回应该运行任务的workerId，如果无法运行任务，则返回null。它可用于快速开发缺少的功能，其中工作者选择逻辑经常被改变或调整。如果选择逻辑非常复杂且无法在javascript环境中轻松测试，那么最好编写一个带有用java编写的扩展当前工作者选择策略的德鲁伊扩展模块。

| 属性       | 描述                       | 默认                     |
| ---------- | -------------------------- | ------------------------ |
| `type`     | `javascript`。             | 需要; 一定是`javascript` |
| `function` | 表示javascript函数的字符串 |                          |

示例：将batch_index_task发送到Workers 10.0.0.1和10.0.0.2以及将所有其他任务发送给其他可用worker的函数。

```text
{
"type":"javascript",
"function":"function (config, zkWorkers, task) {\nvar batch_workers = new java.util.ArrayList();\nbatch_workers.add(\"10.0.0.1\");\nbatch_workers.add(\"10.0.0.2\");\nworkers = zkWorkers.keySet().toArray();\nvar sortedWorkers = new Array()\n;for(var i = 0; i < workers.length; i++){\n sortedWorkers[i] = workers[i];\n}\nArray.prototype.sort.call(sortedWorkers,function(a, b){return zkWorkers.get(b).getCurrCapacityUsed() - zkWorkers.get(a).getCurrCapacityUsed();});\nvar minWorkerVer = config.getMinWorkerVersion();\nfor (var i = 0; i < sortedWorkers.length; i++) {\n var worker = sortedWorkers[i];\n  var zkWorker = zkWorkers.get(worker);\n  if(zkWorker.canRunTask(task) && zkWorker.isValidVersion(minWorkerVer)){\n    if(task.getType() == 'index_hadoop' && batch_workers.contains(worker)){\n      return worker;\n    } else {\n      if(task.getType() != 'index_hadoop' && !batch_workers.contains(worker)){\n        return worker;\n      }\n    }\n  }\n}\nreturn null;\n}"
}
```

默认情况下禁用基于JavaScript的功能。有关使用德鲁伊JavaScript功能的[指南](http://druid.io/docs/0.12.3/development/javascript.html)，请参阅德鲁伊[JavaScript编程指南](http://druid.io/docs/0.12.3/development/javascript.html)，包括如何启用它的说明。

##### 亲和

可以使用“affinityConfig”字段将相关性配置提供给*equalDistribution*和*fillCapacity*策略。如果未提供，则默认为根本不使用关联。

| 属性       | 描述                                                         | 默认 |
| ---------- | ------------------------------------------------------------ | ---- |
| `affinity` | JSON对象将数据源String名称映射到索引服务midManager主机列表：port String值。Druid不执行DNS解析，因此'host'值必须与middleManager上配置的内容以及middleManager宣布自己的内容相匹配（检查Overlord日志以查看middleManager宣布自己的内容）。 | {}   |
| `strong`   | 如果亲缘关系映射的middleManager不能在该数据源的队列中运行所有挂起的任务，则可以使用弱亲和力（默认值）将dataSource的任务分配给其他middleManager。凭借强大的亲和力，dataSource的任务将仅分配给其亲缘关系映射的middleManager，并在必要时在待处理队列中等待。 | 假   |

#### 自动配置器

亚马逊的EC2是目前唯一支持的自动缩放器。

| 属性               | 描述                                                         | 默认         |
| ------------------ | ------------------------------------------------------------ | ------------ |
| `minNumWorkers`    | 在任何给定时间可以在群集中的最小工作数。                     | 0            |
| `maxNumWorkers`    | 在任何给定时间可以在群集中的最大工作数。                     | 0            |
| `availabilityZone` | 要运行的可用区域。                                           | 没有         |
| `nodeData`         | 描述如何启动新节点的JSON对象。                               | 没有; 需要   |
| `userData`         | 描述如何配置新节点的JSON对象。如果你设置了druid.indexer.autoscale.workerVersion，那么它必须有一个versionReplacementString。否则，不需要versionReplacementString。 | 没有; 可选的 |

## MiddleManager和Peons

可以在`middleManager/runtime.properties`文件中定义这些MiddleManager和Peon配置。

### MiddleManager节点配置

| 属性                  | 描述                                                         | 默认                                                   |
| --------------------- | ------------------------------------------------------------ | ------------------------------------------------------ |
| `druid.host`          | 当前节点的主机。这用于将当前进程位置通告为可从另一节点到达，并且通常应该指定为`http://${druid.host}/`可以实际与该进程通信 | InetAddress.getLocalHost（）。getCanonicalHostName（） |
| `druid.plaintextPort` | 这是实际收听的端口; 除非使用端口映射，否则这将是与之相同的端口`druid.host` | 8091                                                   |
| `druid.tlsPort`       | HTTPS连接器的TLS端口，如果设置了[druid.enableTlsPort](http://druid.io/docs/0.12.3/operations/tls-support.html)，则将使用此配置。如果`druid.host`包含端口，则该端口将被忽略。这应该是一个非负整数。 | 8291                                                   |
| `druid.service`       | 服务的名称。在发出指标和警报以区分各种服务时，这用作维度     | 德鲁伊/ middlemanager                                  |

### MiddleManager配置

中层管理人员将他们的配置传递给他们的孩子。中层经理需要以下配置：

| 属性                                             | 描述                                                         | 默认                                                         |
| ------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `druid.indexer.runner.allowedPrefixes`           | 用于配置的前缀的白名单，可以传递给子工具。                   | “com.metamx”，“druid”，“io.druid”，“user.timezone”，“file.encoding”，“java.io.tmpdir”，“hadoop” |
| `druid.indexer.runner.compressZnodes`            | 指示中间管理员是否应该压缩Znodes。                           | 真正                                                         |
| `druid.indexer.runner.classpath`                 | peon的Java类路径。                                           | System.getProperty（ “java.class.path”）                     |
| `druid.indexer.runner.javaCommand`               | 执行java所需的命令。                                         | java的                                                       |
| `druid.indexer.runner.javaOpts`                  | *DEPRECATED*一串-X Java选项，传递给peon的JVM。鼓励使用带空格的可引用参数或参数来使用javaOptsArray | “”                                                           |
| `druid.indexer.runner.javaOptsArray`             | 一个json数组的字符串作为选项传递给peon的jvm。这是javaOpts的附加内容，建议用于正确处理包含引号或空格的参数`["-XX:OnOutOfMemoryError=kill -9 %p"]` | `[]`                                                         |
| `druid.indexer.runner.maxZnodeBytes`             | 可以在Zookeeper中创建的最大大小Znode（以字节为单位）。       | 524288                                                       |
| `druid.indexer.runner.startPort`                 | 用于peon进程的起始端口，应大于1023。                         | 8100                                                         |
| `druid.indexer.runner.tlsStartPort`              | 启动Peon进程的TLS端口，应大于1023。                          | 8300                                                         |
| `druid.indexer.runner.separateIngestionEndpoint` | *已过时。*使用单独的服务器，因此单独的jetty线程池用于摄取事件。TLS不支持。 | 假                                                           |
| `druid.worker.ip`                                | 工人的知识产权。                                             | 本地主机                                                     |
| `druid.worker.version`                           | 中层经理的版本标识符。                                       | 0                                                            |
| `druid.worker.capacity`                          | 中层经理可以接受的最大任务数。                               | 可用处理器数量 - 1                                           |

### Peon Processing

在Middlemanager上设置的处理属性将传递给Peons。

| 属性                                        | 描述                                                         | 默认                                      |
| ------------------------------------------- | ------------------------------------------------------------ | ----------------------------------------- |
| `druid.processing.buffer.sizeBytes`         | 这指定了用于存储中间结果的缓冲区大小。Historical和Realtime节点中的计算引擎将使用此大小的暂存缓冲区在堆外执行所有中间计算。较大的值允许在数据上单次传递更多聚合，而较小的值可能需要更多传递，具体取决于正在执行的查询。 | 1073741824（1GB）                         |
| `druid.processing.buffer.poolCacheMaxCount` | 处理缓冲池缓存缓冲区供以后使用，这是缓存增长到的最大计数。请注意，如果需要，池可以创建比缓存更多的缓冲区。 | Integer.MAX_VALUE的                       |
| `druid.processing.formatString`             | 实时和历史节点使用此格式字符串来命名其处理线程。             | 处理 - ％S                                |
| `druid.processing.numMergeBuffers`          | 可用于合并查询结果的直接内存缓冲区的数量。缓冲区的大小为`druid.processing.buffer.sizeBytes`。对于需要合并缓冲区的查询，此属性实际上是并发限制。如果您正在使用任何需要合并缓冲区的查询（目前只是groupBy v2），那么您应该至少有两个这样的查询。 | `max(2, druid.processing.numThreads / 4)` |
| `druid.processing.numThreads`               | 可用于并行处理段的处理线程数。我们的经验法则是`num_cores - 1`，这意味着即使在高负荷下，仍然会有一个核心可用于执行后台任务，例如与ZooKeeper交谈和下拉段。如果只有一个核心可用，则此属性默认为该值`1`。 | 核心数量 - 1（或1）                       |
| `druid.processing.columnCache.sizeBytes`    | 维值查找缓存的最大大小（以字节为单位）。任何大于的值都会`0`启用缓存。它目前默认是禁用的。启用查找缓存可以显着提高在维度值上运行的聚合器的性能，例如JavaScript聚合器或基数聚合器，但如果缓存命中率较低（即具有较少重复值的维度），则可能会降低速度。启用它可能还需要额外的垃圾收集调整，以避免长时间的GC暂停。 | `0` （禁用）                              |
| `druid.processing.fifo`                     | 如果处理队列应以FIFO方式处理具有相同优先级的任务             | `false`                                   |
| `druid.processing.tmpDir`                   | 应存储处理查询时创建的临时文件的路径。如果指定，则此配置优先于默认`java.io.tmpdir`路径。 | 路径代表 `java.io.tmpdir`                 |

德鲁伊所需的直接记忆量至少是 `druid.processing.buffer.sizeBytes * (druid.processing.numMergeBuffers + druid.processing.numThreads + 1)`。您可以确保至少这直接内存量可通过提供`-XX:MaxDirectMemorySize=<VALUE>`在`druid.indexer.runner.javaOptsArray`如上记录。

### Peon查询配置

请参阅[常规查询配置](http://druid.io/docs/0.12.3/configuration/index.html#general-query-configuration)。

### Peon缓存

您可以选择通过在此处设置缓存配置来配置要在Peons上启用的缓存。

| 属性                                 | 可能的值           | 描述                   | 默认                    |
| ------------------------------------ | ------------------ | ---------------------- | ----------------------- |
| `druid.realtime.cache.useCache`      | 真假               | 实时启用缓存。         | 假                      |
| `druid.realtime.cache.populateCache` | 真假               | 在实时填充缓存。       | 假                      |
| `druid.realtime.cache.unCacheable`   | 所有德鲁伊查询类型 | 所有不缓存的查询类型。 | `["groupBy", "select"]` |

有关如何配置缓存设置，请参阅[缓存配置](http://druid.io/docs/0.12.3/configuration/index.html#cache-configuration)。

### 额外的Peon配置

尽管peons继承了其父级中间管理器的配置，但中级管理器中的显式子peon配置可以通过为它们添加前缀来设置：

```text
druid.indexer.fork.property
```

额外的peon配置包括：

| 属性                                          | 描述                                                         | 默认                                             |
| --------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------ |
| `druid.peon.mode`                             | 选择是“本地”和“远程”。将此设置为local表示您打算将peon作为独立节点运行（不推荐）。 | 远程                                             |
| `druid.indexer.task.baseDir`                  | 基本临时工作目录。                                           | `System.getProperty("java.io.tmpdir")`           |
| `druid.indexer.task.baseTaskDir`              | 任务的基本临时工作目录。                                     | `${druid.indexer.task.baseDir}/persistent/tasks` |
| `druid.indexer.task.defaultHadoopCoordinates` | Hadoop版本与不请求特定版本的HadoopIndexTasks一起使用。       | org.apache.hadoop：Hadoop的客户：2.3.0           |
| `druid.indexer.task.defaultRowFlushBoundary`  | 持久到磁盘之前的最高行数。用于索引生成任务。                 | 75000                                            |
| `druid.indexer.task.directoryLockTimeout`     | 等待这个漫长的僵尸工匠退出，然后放弃更换。                   | PT10M                                            |
| `druid.indexer.task.gracefulShutdownTimeout`  | 在middleManager重新启动时等待这么长时间，以便可恢复的任务正常退出。 | PT5M                                             |
| `druid.indexer.task.hadoopWorkingPath`        | Hadoop任务的临时工作目录。                                   | `/tmp/druid-indexing`                            |
| `druid.indexer.task.restoreTasksOnRestart`    | 如果为true，则middleManagers将尝试在关闭时正常停止任务，并在重新启动时恢复它们。 | 假                                               |
| `druid.indexer.server.maxChatRequests`        | 任务的聊天处理程序所服务的最大并发请求数。设置为0以禁用限制。 | 0                                                |

如果deprecated `druid.indexer.runner.separateIngestionEndpoint`属性设置为true，则以下配置可用于peon的摄取服务器：

| 属性                                                | 描述                                  | 默认                                               |
| --------------------------------------------------- | ------------------------------------- | -------------------------------------------------- |
| `druid.indexer.server.chathandler.http.numThreads`  | *已过时。*HTTP请求的线程数。          | Math.max（10，（可用处理器数量* 17）/ 16 + 2）+ 30 |
| `druid.indexer.server.chathandler.http.maxIdleTime` | *已过时。*Jetty最大空闲时间用于连接。 | PT5m                                               |

如果peon在远程模式下运行，则必须有一个霸主并且正在运行。远程模式中的Peons可以设置以下配置：

| 属性                                              | 描述                       | 默认 |
| ------------------------------------------------- | -------------------------- | ---- |
| `druid.peon.taskActionClient.retry.minWait`       | 与霸主沟通的最短重试时间。 | PT5S |
| `druid.peon.taskActionClient.retry.maxWait`       | 与霸主沟通的最长重试时间。 | PT1M |
| `druid.peon.taskActionClient.retry.maxRetryCount` | 与霸主沟通的最大重试次数。 | 60   |

#### SegmentWriteOutMediumFactory

创建新段时，Druid会将一些预处理数据临时存储在某些缓冲区中。目前，这些缓冲区存在两种类型的 *介质*：*临时文件*和*堆外存储器*。

*临时文件*（`tmpFile`）存储在任务工作目录下（参见`druid.indexer.task.baseTaskDir` 上面的配置），因此可以共享它的安装特性，例如它们可以由HDD，SSD或内存（tmpfs）支持。这种类型的介质可能会执行不必要的磁盘I / O，并且需要一些磁盘空间。

*堆外内存medium*（`offHeapMemory`）在运行任务的JVM进程*的堆外内存中*创建缓冲区。这种类型的介质是首选，但可能需要通过更改`-XX:MaxDirectMemorySize`配置来允许JVM拥有更多的堆外内存 。目前尚不清楚所需的堆外内存大小如何与正在创建的段的大小相关。但是，相比于同一JVM 配置的最大*堆*大小（`-Xmx`），添加更多额外的堆外内存肯定没有意义。

对于大多数类型的任务，可以按任务配置SegmentWriteOutMediumFactory（请参阅[任务](http://druid.io/docs/0.12.3/ingestion/tasks.html) 页面，“TuningConfig”部分），但如果没有为任务指定，或者特定任务类型不支持，则下面配置中的值为用过的：

| 属性                                             | 描述                                       | 默认      |
| ------------------------------------------------ | ------------------------------------------ | --------- |
| `druid.peon.defaultSegmentWriteOutMediumFactory` | `tmpFile`或者`offHeapMemory`，见上面的解释 | `tmpFile` |

## 经纪人

有关一般Broker Node信息，请参见[此处](http://druid.io/docs/0.12.3/design/broker.html)。

可以在`broker/runtime.properties`文件中定义这些Broker配置。

### 经纪人节点配置

| 属性                  | 描述                                                         | 默认                                                   |
| --------------------- | ------------------------------------------------------------ | ------------------------------------------------------ |
| `druid.host`          | 当前节点的主机。这用于将当前进程位置通告为可从另一节点到达，并且通常应该指定为`http://${druid.host}/`可以实际与该进程通信 | InetAddress.getLocalHost（）。getCanonicalHostName（） |
| `druid.plaintextPort` | 这是实际收听的端口; 除非使用端口映射，否则这将是与之相同的端口`druid.host` | 8082                                                   |
| `druid.tlsPort`       | HTTPS连接器的TLS端口，如果设置了[druid.enableTlsPort](http://druid.io/docs/0.12.3/operations/tls-support.html)，则将使用此配置。如果`druid.host`包含端口，则该端口将被忽略。这应该是一个非负整数。 | 8282                                                   |
| `druid.service`       | 服务的名称。在发出指标和警报以区分各种服务时，这用作维度     | 德鲁伊/经纪商                                          |

### 查询配置

#### 查询优先级

| 属性                                         | 可能的值                                      | 描述                                                         | 默认              |
| -------------------------------------------- | --------------------------------------------- | ------------------------------------------------------------ | ----------------- |
| `druid.broker.balancer.type`                 | `random`， `connectionCount`                  | 确定代理如何平衡与历史节点的连接。`random`随机选择，`connectionCount`选择活动连接数最少的节点 | `random`          |
| `druid.broker.select.tier`                   | `highestPriority`，`lowestPriority`，`custom` | 如果段在群集中的层之间进行交叉复制，则可以告诉代理更愿意选择具有特定优先级的层中的段。 | `highestPriority` |
| `druid.broker.select.tier.custom.priorities` | `An array of integer priorities.`             | 使用自定义优先级列表选择层中的服务器。                       | 没有              |

#### 并发请求

Druid使用Jetty来提供HTTP请求。

| 属性                                      | 描述                                                         | 默认                                |
| ----------------------------------------- | ------------------------------------------------------------ | ----------------------------------- |
| `druid.server.http.numThreads`            | HTTP请求的线程数。                                           | max（10，（核数* 17）/ 16 + 2）+ 30 |
| `druid.server.http.queueSize`             | Jetty服务器用于临时存储传入客户端连接的工作队列的大小。如果设置了此值并且jetty拒绝了请求，因为队列已满，那么客户端将观察到请求失败，TCP连接立即被关闭，服务器完全为空响应。 | 无界                                |
| `druid.server.http.maxIdleTime`           | Jetty最大空闲时间用于连接。                                  | PT5m                                |
| `druid.server.http.enableRequestLimit`    | 如果启用，则不会在jetty队列中排队请求，并且将发送“HTTP 429 Too Many Requests”错误响应。 | 假                                  |
| `druid.server.http.defaultQueryTimeout`   | 以毫秒为单位的查询超时，超过该超时将取消未完成的查询         | 300000                              |
| `druid.server.http.maxScatterGatherBytes` | 从数据节点（例如历史记录和实时进程）收集的最大字节数，以执行查询。这是一种先进的配置，允许在代理处于高负载且不足够快地利用在存储器中收集的数据并导致OOM的情况下进行保护。`maxScatterGatherBytes`在上下文中使用查询时可以进一步减少此限制。请注意，如果代理从不在大量并发负载下，那么具有大限制并不一定是坏的，在这种情况下，收集的数据会被快速处理并释放所使用的内存。 | Long.MAX_VALUE                      |
| `druid.broker.http.numConnections`        | Broker连接到历史和实时进程的连接池大小。如果查询数多于此数，则所有查询都需要与同一节点对话，那么它们将排队。 | 20                                  |
| `druid.broker.http.compressionCodec`      | Broker用于与历史和实时流程进行通信的压缩编解码器。可能是“gzip”或“身份”。 | gzip的                              |
| `druid.broker.http.readTimeout`           | 数据超时从历史和实时进程读取。                               | PT15M                               |
| `druid.server.http.maxQueryTimeout`       | 参数的最大允许值（以毫秒为单位）`timeout`。请参阅[query-context](http://druid.io/docs/0.12.3/querying/query-context.html)以了解更多信息`timeout`。如果查询上下文`timeout`大于此值，则拒绝查询。 | Long.MAX_VALUE                      |
| `druid.server.http.maxRequestHeaderSize`  | 请求标头的最大大小（以字节为单位）。较大的标头消耗更多内存，并且可能使服务器更容易受到拒绝服务攻击。 | 8 * 1024                            |

#### 重试政策

德鲁伊经纪人可以选择在内部重试查询以查找瞬态错误。

| 属性                                | 描述       | 默认 |
| ----------------------------------- | ---------- | ---- |
| `druid.broker.retryPolicy.numTries` | 尝试次数。 | 1    |

#### 处理

代理使用处理配置进行嵌套的groupBy查询。并且，可选地，可以将长间隔查询（任何类型）分解为更短的间隔查询，并在该线程池内并行处理。有关更多详细信息，请参阅[查询上下文](http://druid.io/docs/0.12.3/querying/query-context.html) doc中的“chunkPeriod” 。

| 属性                                        | 描述                                                         | 默认                                      |
| ------------------------------------------- | ------------------------------------------------------------ | ----------------------------------------- |
| `druid.processing.buffer.sizeBytes`         | 这指定了用于存储中间结果的缓冲区大小。Historical和Realtime节点中的计算引擎将使用此大小的暂存缓冲区在堆外执行所有中间计算。较大的值允许在数据上单次传递更多聚合，而较小的值可能需要更多传递，具体取决于正在执行的查询。 | 1073741824（1GB）                         |
| `druid.processing.buffer.poolCacheMaxCount` | 处理缓冲池缓存缓冲区供以后使用，这是缓存增长到的最大计数。请注意，如果需要，池可以创建比缓存更多的缓冲区。 | Integer.MAX_VALUE的                       |
| `druid.processing.formatString`             | 实时和历史节点使用此格式字符串来命名其处理线程。             | 处理 - ％S                                |
| `druid.processing.numMergeBuffers`          | 可用于合并查询结果的直接内存缓冲区的数量。缓冲区的大小为`druid.processing.buffer.sizeBytes`。对于需要合并缓冲区的查询，此属性实际上是并发限制。如果您正在使用任何需要合并缓冲区的查询（目前只是groupBy v2），那么您应该至少有两个这样的查询。 | `max(2, druid.processing.numThreads / 4)` |
| `druid.processing.numThreads`               | 可用于并行处理段的处理线程数。我们的经验法则是`num_cores - 1`，这意味着即使在高负荷下，仍然会有一个核心可用于执行后台任务，例如与ZooKeeper交谈和下拉段。如果只有一个核心可用，则此属性默认为该值`1`。 | 核心数量 - 1（或1）                       |
| `druid.processing.columnCache.sizeBytes`    | 维值查找缓存的最大大小（以字节为单位）。任何大于的值都会`0`启用缓存。它目前默认是禁用的。启用查找缓存可以显着提高在维度值上运行的聚合器的性能，例如JavaScript聚合器或基数聚合器，但如果缓存命中率较低（即具有较少重复值的维度），则可能会降低速度。启用它可能还需要额外的垃圾收集调整，以避免长时间的GC暂停。 | `0` （禁用）                              |
| `druid.processing.fifo`                     | 如果处理队列应以FIFO方式处理具有相同优先级的任务             | `false`                                   |
| `druid.processing.tmpDir`                   | 应存储处理查询时创建的临时文件的路径。如果指定，则此配置优先于默认`java.io.tmpdir`路径。 | 路径代表 `java.io.tmpdir`                 |

德鲁伊所需的直接记忆量至少是 `druid.processing.buffer.sizeBytes * (druid.processing.numMergeBuffers + druid.processing.numThreads + 1)`。您可以通过`-XX:MaxDirectMemorySize=<VALUE>`在命令行中提供至少这一数量的直接内存。

#### 代理查询配置

请参阅[常规查询配置](http://druid.io/docs/0.12.3/configuration/index.html#general-query-configuration)。

### SQL

通过代理上的以下属性配置Druid SQL服务器。

| 属性                                            | 描述                                                         | 默认   |
| ----------------------------------------------- | ------------------------------------------------------------ | ------ |
| `druid.sql.enable`                              | 是否要启用SQL，包括后台元数据获取。如果为false，则会覆盖所有其他与SQL相关的属性，并完全禁用SQL元数据，服务和计划。 | 假     |
| `druid.sql.avatica.enable`                      | 是否启用JDBC查询`/druid/v2/sql/avatica/`。                   | 真正   |
| `druid.sql.avatica.maxConnections`              | Avatica服务器的最大打开连接数。这些不是HTTP连接，而是可以跨多个HTTP连接的逻辑客户端连接。 | 50     |
| `druid.sql.avatica.maxRowsPerFrame`             | 在单个JDBC框架中返回的最大行数。将此属性设置为-1表示不应应用行限制。客户端可以选择在其请求中指定行限制; 如果客户端指定行限制，则将使用客户端提供的限制的较小值`maxRowsPerFrame`。 | 100000 |
| `druid.sql.avatica.maxStatementsPerConnection`  | 每个Avatica客户端连接的最大同时打开语句数。                  | 1      |
| `druid.sql.avatica.connectionIdleTimeout`       | Avatica客户端连接空闲超时。                                  | PT5M   |
| `druid.sql.http.enable`                         | 是否通过HTTP查询启用JSON`/druid/v2/sql/`。                   | 真正   |
| `druid.sql.planner.maxQueryCount`               | 要发出的最大查询数，包括嵌套查询。设置为1表示禁用子查询，或设置为0表示无限制。 | 8      |
| `druid.sql.planner.maxSemiJoinRowsInMemory`     | 内存中用于执行两阶段半连接查询的最大行数，例如`SELECT * FROM Employee WHERE DeptName IN (SELECT DeptName FROM Dept)`。 | 100000 |
| `druid.sql.planner.maxTopNLimit`                | [TopN查询的](http://druid.io/docs/0.12.3/querying/topnquery.html)最大阈值。相反，将计划更高的限制作为[GroupBy查询](http://druid.io/docs/0.12.3/querying/groupbyquery.html)。 | 100000 |
| `druid.sql.planner.metadataRefreshPeriod`       | 节流元数据刷新。                                             | PT1M   |
| `druid.sql.planner.selectPageSize`              | [选择查询的](http://druid.io/docs/0.12.3/querying/select-query.html)页面大小阈值。对于较大的结果集的选择查询将使用分页背靠背发出。 | 1000   |
| `druid.sql.planner.useApproximateCountDistinct` | 是否使用近似基数算法`COUNT(DISTINCT foo)`。                  | 真正   |
| `druid.sql.planner.useApproximateTopN`          | 当SQL查询可以表达时，是否使用近似[TopN查询](http://druid.io/docs/0.12.3/querying/topnquery.html)。如果为false，则将使用确切的[GroupBy查询](http://druid.io/docs/0.12.3/querying/groupbyquery.html)。 | 真正   |
| `druid.sql.planner.useFallback`                 | 是否评估代理上的操作时，它们不能表示为德鲁伊查询。建议不要将此选项用于生产，因为它可以生成不可伸缩的查询计划。如果为false，则无法转换为Druid查询的SQL查询将失败。 | 假     |

### 经纪人缓存

您可以选择仅在此处通过设置缓存配置来配置在代理上启用的缓存。

| 属性                                     | 可能的值           | 描述                                                         | 默认                    |
| ---------------------------------------- | ------------------ | ------------------------------------------------------------ | ----------------------- |
| `druid.broker.cache.useCache`            | 真假               | 在代理上启用缓存。                                           | 假                      |
| `druid.broker.cache.populateCache`       | 真假               | 填充代理上的缓存。                                           | 假                      |
| `druid.broker.cache.unCacheable`         | 所有德鲁伊查询类型 | 所有不缓存的查询类型。                                       | `["groupBy", "select"]` |
| `druid.broker.cache.cacheBulkMergeLimit` | 正整数或0          | 具有比此数字更多的段的查询将不会尝试从代理级别的缓存中获取，从而将潜在的缓存提取（以及缓存结果合并）留给历史记录 | `Integer.MAX_VALUE`     |

有关如何配置缓存设置，请参阅[缓存配置](http://druid.io/docs/0.12.3/configuration/index.html#cache-configuration)。

### 段发现

| 属性                                      | 可能的值     | 描述                                                         | 默认 |
| ----------------------------------------- | ------------ | ------------------------------------------------------------ | ---- |
| `druid.announcer.type`                    | 批处理或http | 要使用的段发现方法。“http”允许使用HTTP而不是zookeeper发现段。 | 批量 |
| `druid.broker.segment.watchedTiers`       | 字符串列表   | Broker监视来自服务段的节点的段声明以构建哪个节点正在服务哪些段的缓存，该配置允许仅考虑从层的白名单服务的段。默认情况下，Broker会考虑所有层级。这可用于在特定历史层中对dataSource进行分区，并在分区中配置代理，以便它们仅可查询特定的dataSource。 | 没有 |
| `druid.broker.segment.watchedDataSources` | 字符串列表   | Broker监视来自服务段的节点的段声明以构建哪个节点正在服务哪些段的缓存，该配置允许仅考虑从dataSource的白名单服务的段。默认情况下，Broker会考虑所有数据源。这可用于在分区中配置代理，以便它们仅可查询特定的dataSource。 | 没有 |

## 历史的

有关常规历史节点信息，请参见[此处](http://druid.io/docs/0.12.3/design/historical.html)。

可以在`historical/runtime.properties`文件中定义这些历史配置。

### 历史节点配置

| 属性                  | 描述                                                         | 默认                                                   |
| --------------------- | ------------------------------------------------------------ | ------------------------------------------------------ |
| `druid.host`          | 当前节点的主机。这用于将当前进程位置通告为可从另一节点到达，并且通常应该指定为`http://${druid.host}/`可以实际与该进程通信 | InetAddress.getLocalHost（）。getCanonicalHostName（） |
| `druid.plaintextPort` | 这是实际收听的端口; 除非使用端口映射，否则这将是与之相同的端口`druid.host` | 8083                                                   |
| `druid.tlsPort`       | HTTPS连接器的TLS端口，如果设置了[druid.enableTlsPort](http://druid.io/docs/0.12.3/operations/tls-support.html)，则将使用此配置。如果`druid.host`包含端口，则该端口将被忽略。这应该是一个非负整数。 | 8283                                                   |
| `druid.service`       | 服务的名称。在发出指标和警报以区分各种服务时，这用作维度     | 德鲁伊/历史                                            |

### 历史一般配置

| 属性                    | 描述                                                         | 默认            |
| ----------------------- | ------------------------------------------------------------ | --------------- |
| `druid.server.maxSize`  | 节点要分配给它的最大字节数。这不是Historical节点实际强制执行的限制，只是发布到Coordinator节点的值，因此可以相应地进行规划。 | 0               |
| `druid.server.tier`     | 用于命名存储节点所属的分发层的字符串。[协调器节点用于](http://druid.io/docs/0.12.3/operations/rule-configuration.html)管理段的许多[规则](http://druid.io/docs/0.12.3/operations/rule-configuration.html)可以在层上键入。 | `_default_tier` |
| `druid.server.priority` | 在分层体系结构中，层的优先级，从而允许控制查询哪些节点。数字越大表示优先级越高。默认（无优先级）适用于没有交叉复制的架构（没有数据存储重叠的层）。数据中心通常具有相同的优先级 | 0               |

### 存储细分

| 属性                                        | 描述                                                         | 默认                           |
| ------------------------------------------- | ------------------------------------------------------------ | ------------------------------ |
| `druid.segmentCache.locations`              | 分配给Historical节点的段首先存储在本地文件系统上（在磁盘缓存中），然后由Historical节点提供服务。这些位置定义本地缓存所在的位置。该值不能为NULL或EMPTY。这是一个例子`druid.segmentCache.locations=[{"path": "/mnt/druidSegments", "maxSize": 10000, "freeSpacePercent": 1.0}]`。“freeSpacePercent”是可选的，如果提供，则在存储段时强制执行大量的可用磁盘分区空间。但是，它依赖于File.getTotalSpace（）和File.getFreeSpace（）方法，因此只有当它们适用于您的文件系统时才启用。 | 没有                           |
| `druid.segmentCache.deleteOnRemove`         | 一旦节点不再为段提供服务，就从缓存中删除段文件。             | 真正                           |
| `druid.segmentCache.dropSegmentDelayMillis` | 节点在完全丢弃段之前延迟多长时间。                           | 30000（30秒）                  |
| `druid.segmentCache.infoDir`                | 历史节点跟踪它们正在服务的段，以便在重新启动进程时，它们可以重新加载相同的段，而无需等待协调器重新分配。此路径定义此元数据的保存位置。如果需要，将创建目录。 | $ {} first_location / info_dir |
| `druid.segmentCache.announceIntervalMillis` | 在从缓存加载段时频繁地声明段。将此值设置为零以等待在宣布之前加载所有段。 | 5000（5秒）                    |
| `druid.segmentCache.numLoadingThreads`      | 从深层存储中同时删除或加载的段数。                           | 10                             |
| `druid.segmentCache.numBootstrapThreads`    | 启动时从本地存储同时加载的段数。                             | 与numLoadingThreads相同        |

在`druid.segmentCache.locations`，添加了*freeSpacePercent*，因为*maxSize*设置只是一个理论限制，并假设有足够的空间可用于存储段。如果任何德鲁伊bug导致未记录的段文件留在磁盘上或其他一些进程将内容写入磁盘，则此检查可能会在完全填满磁盘之前提前启动段加载失败，否则将使主机可用。

### 历史查询配置

#### 并发请求

Druid使用Jetty来提供HTTP请求。

| 属性                                     | 描述                                                         | 默认                                |
| ---------------------------------------- | ------------------------------------------------------------ | ----------------------------------- |
| `druid.server.http.numThreads`           | HTTP请求的线程数。                                           | max（10，（核数* 17）/ 16 + 2）+ 30 |
| `druid.server.http.queueSize`            | Jetty服务器用于临时存储传入客户端连接的工作队列的大小。如果设置了此值并且jetty拒绝了请求，因为队列已满，那么客户端将观察到请求失败，TCP连接立即被关闭，服务器完全为空响应。 | 无界                                |
| `druid.server.http.maxIdleTime`          | Jetty最大空闲时间用于连接。                                  | PT5m                                |
| `druid.server.http.enableRequestLimit`   | 如果启用，则不会在jetty队列中排队请求，并且将发送“HTTP 429 Too Many Requests”错误响应。 | 假                                  |
| `druid.server.http.defaultQueryTimeout`  | 以毫秒为单位的查询超时，超过该超时将取消未完成的查询         | 300000                              |
| `druid.server.http.maxQueryTimeout`      | 参数的最大允许值（以毫秒为单位）`timeout`。请参阅[query-context](http://druid.io/docs/0.12.3/querying/query-context.html)以了解更多信息`timeout`。如果查询上下文`timeout`大于此值，则拒绝查询。 | Long.MAX_VALUE                      |
| `druid.server.http.maxRequestHeaderSize` | 请求标头的最大大小（以字节为单位）。较大的标头消耗更多内存，并且可能使服务器更容易受到拒绝服务攻击。 | 8 * 1024                            |

#### 处理

| 属性                                        | 描述                                                         | 默认                                      |
| ------------------------------------------- | ------------------------------------------------------------ | ----------------------------------------- |
| `druid.processing.buffer.sizeBytes`         | 这指定了用于存储中间结果的缓冲区大小。Historical和Realtime节点中的计算引擎将使用此大小的暂存缓冲区在堆外执行所有中间计算。较大的值允许在数据上单次传递更多聚合，而较小的值可能需要更多传递，具体取决于正在执行的查询。 | 1073741824（1GB）                         |
| `druid.processing.buffer.poolCacheMaxCount` | 处理缓冲池缓存缓冲区供以后使用，这是缓存增长到的最大计数。请注意，如果需要，池可以创建比缓存更多的缓冲区。 | Integer.MAX_VALUE的                       |
| `druid.processing.formatString`             | 实时和历史节点使用此格式字符串来命名其处理线程。             | 处理 - ％S                                |
| `druid.processing.numMergeBuffers`          | 可用于合并查询结果的直接内存缓冲区的数量。缓冲区的大小为`druid.processing.buffer.sizeBytes`。对于需要合并缓冲区的查询，此属性实际上是并发限制。如果您正在使用任何需要合并缓冲区的查询（目前只是groupBy v2），那么您应该至少有两个这样的查询。 | `max(2, druid.processing.numThreads / 4)` |
| `druid.processing.numThreads`               | 可用于并行处理段的处理线程数。我们的经验法则是`num_cores - 1`，这意味着即使在高负荷下，仍然会有一个核心可用于执行后台任务，例如与ZooKeeper交谈和下拉段。如果只有一个核心可用，则此属性默认为该值`1`。 | 核心数量 - 1（或1）                       |
| `druid.processing.columnCache.sizeBytes`    | 维值查找缓存的最大大小（以字节为单位）。任何大于的值都会`0`启用缓存。它目前默认是禁用的。启用查找缓存可以显着提高在维度值上运行的聚合器的性能，例如JavaScript聚合器或基数聚合器，但如果缓存命中率较低（即具有较少重复值的维度），则可能会降低速度。启用它可能还需要额外的垃圾收集调整，以避免长时间的GC暂停。 | `0` （禁用）                              |
| `druid.processing.fifo`                     | 如果处理队列应以FIFO方式处理具有相同优先级的任务             | `false`                                   |
| `druid.processing.tmpDir`                   | 应存储处理查询时创建的临时文件的路径。如果指定，则此配置优先于默认`java.io.tmpdir`路径。 | 路径代表 `java.io.tmpdir`                 |

德鲁伊所需的直接记忆量至少是 `druid.processing.buffer.sizeBytes * (druid.processing.numMergeBuffers + druid.processing.numThreads + 1)`。您可以通过`-XX:MaxDirectMemorySize=<VALUE>`在命令行中提供至少这一数量的直接内存。

#### 历史查询配置

请参阅[常规查询配置](http://druid.io/docs/0.12.3/configuration/index.html#general-query-configuration)。

### 历史缓存

您可以选择仅通过在此处设置缓存配置来配置对历史记录启用的缓存。

| 属性                                   | 可能的值           | 描述                   | 默认                |
| -------------------------------------- | ------------------ | ---------------------- | ------------------- |
| `druid.historical.cache.useCache`      | 真假               | 在历史记录上启用缓存。 | 假                  |
| `druid.historical.cache.populateCache` | 真假               | 填充历史记录上的缓存。 | 假                  |
| `druid.historical.cache.unCacheable`   | 所有德鲁伊查询类型 | 所有不缓存的查询类型。 | [“groupBy”，“选择”] |

有关如何配置缓存设置，请参阅[缓存配置](http://druid.io/docs/0.12.3/configuration/index.html#cache-configuration)。

## 缓存配置

本节介绍了broker，historical和middleManager / peon节点通用的缓存配置。

可以选择在代理，历史和实时处理上启用缓存。有关 如何为不同进程启用它的信息，请参阅[代理](http://druid.io/docs/0.12.3/configuration/broker.html#caching)， [历史](http://druid.io/docs/0.12.3/configuration/historical.html#caching)和[实时](http://druid.io/docs/0.12.3/configuration/realtime.html#caching)
配置选项。

除非指定了不同类型的缓存，否则Druid默认使用本地内存缓存。使用`druid.cache.type`配置设置不同类型的缓存。

缓存设置是全局设置的，因此在公共属性文件中定义时，可以为代理和历史节点重用相同的配置。

### 缓存类型

| 属性               | 可能的值                                   | 描述                                                   | 默认       |
| ------------------ | ------------------------------------------ | ------------------------------------------------------ | ---------- |
| `druid.cache.type` | `local`，`memcached`，`hybrid`，`caffeine` | 用于查询的缓存类型。请参阅下面的每种缓存类型的配置选项 | `caffeine` |

#### 本地缓存

弃用：使用咖啡因（默认为v0.12.0）

不推荐使用本地缓存以支持Caffeine缓存，并且可能会在将来的Druid版本中删除。与`local`缓存相比，Caffeine缓存提供了明显更好的性能和对驱逐行为的控制，建议您在使用JRE 8u60或更高版本的任何情况下。

一个简单的内存LRU缓存。本地缓存驻留在JVM堆内存中，因此如果启用它，请确保相应地增加堆大小。

| 属性                           | 描述                                                 | 默认   |
| ------------------------------ | ---------------------------------------------------- | ------ |
| `druid.cache.sizeInBytes`      | 最大缓存大小（字节）。零禁用缓存。                   | 0      |
| `druid.cache.initialSize`      | 支持缓存的哈希表的初始大小。                         | 500000 |
| `druid.cache.logEvictionCount` | 如果非零，则日志缓存逐出每个`logEvictionCount`项目。 | 0      |

#### 咖啡因缓存

基于[Caffeine的](https://github.com/ben-manes/caffeine)德鲁伊高性能本地缓存实现。如果使用，需要JRE8u60或更高版本`COMMON_FJP`。

##### 组态

以下是此模块已知的配置选项：

| `runtime.properties`               | 描述                                                         | 默认                               |
| ---------------------------------- | ------------------------------------------------------------ | ---------------------------------- |
| `druid.cache.type`                 | 将此设置为`caffeine`或省略参数                               | `caffeine`                         |
| `druid.cache.sizeInBytes`          | 堆上的最大缓存大小（以字节为单位）。                         | 无（无限制）                       |
| `druid.cache.expireAfter`          | 高速缓存条目可能过期的访问后的时间（以毫秒为单位）           | 没有（没有时间限制）               |
| `druid.cache.cacheExecutorFactory` | 执行者工厂用于咖啡因维护。其中一个`COMMON_FJP`，`SINGLE_THREAD`或`SAME_THREAD` | ForkJoinPool公共池（`COMMON_FJP`） |
| `druid.cache.evictOnClose`         | 如果关闭命名空间（例如：从节点中删除段）应该导致急切驱逐关联的缓存值 | `false`                            |

##### `druid.cache.cacheExecutorFactory`

以下是可能的值`druid.cache.cacheExecutorFactory`，用于控制维护任务的运行方式

- `COMMON_FJP`（默认）使用常见的ForkJoinPool。应与[JRE 8u60或更高版本配合使用](https://github.com/druid-io/druid/pull/4810#issuecomment-329922810)。较旧版本的JRE可能比较新的JRE版本具有更差的性能。
- `SINGLE_THREAD` 使用单线程执行程序。
- `SAME_THREAD` 缓存维护是急切的。

#### 度量

除了正常的高速缓存规格，咖啡因缓存实现还报告了以下两个`total`和`delta`

| 公                                     | 描述                                         | 正常值                                                       |
| -------------------------------------- | -------------------------------------------- | ------------------------------------------------------------ |
| `query/cache/caffeine/*/requests`      | 点击次数或未命中次数                         | 命中+想念                                                    |
| `query/cache/caffeine/*/loadTime`      | 咖啡因花费加载新值的时间长度（未使用的功能） | 0                                                            |
| `query/cache/caffeine/*/evictionBytes` | 已从缓存中逐出的大小（以字节为单位）         | 变化，应该调整缓存，`sizeInBytes`以便`sizeInBytes`/ `evictionBytes`大约是您想要的缓存流失率 |

#### Memcached的

使用memcached作为缓存后端。这允许所有节点共享相同的缓存。

| 属性                          | 描述                                                         | 默认              |
| ----------------------------- | ------------------------------------------------------------ | ----------------- |
| `druid.cache.expiration`      | Memcached [到期时间](https://code.google.com/p/memcached/wiki/NewCommands#Standard_Protocol)。 | 2592000（30天）   |
| `druid.cache.timeout`         | 等待Memcached响应的最长时间（以毫秒为单位）。                | 500               |
| `druid.cache.hosts`           | 以逗号分隔的Memcached主机列表`<host:port>`。                 | 没有              |
| `druid.cache.maxObjectSize`   | Memcached对象的最大对象大小（以字节为单位）。                | 52428800（50 MB） |
| `druid.cache.memcachedPrefix` | Memcached中所有键的键前缀。                                  | 德鲁伊            |
| `druid.cache.numConnections`  | 要使用的memcached连接数。                                    | 1                 |

#### 混合动力

使用任意两个缓存的组合作为两级L1 / L2缓存。这可以用于将本地内存缓存与远程memcached缓存组合。

在检查L2之前，缓存请求将首先检查L1缓存。如果存在L1未命中和L2命中，则它也将填充L1。

| 属性                     | 描述                                                         | 默认                       |
| ------------------------ | ------------------------------------------------------------ | -------------------------- |
| `druid.cache.l1.type`    | 用于L1缓存的缓存类型。请参阅`druid.cache.type`有效类型的配置。 | `caffeine`                 |
| `druid.cache.l2.type`    | 用于L2缓存的缓存类型。请参阅`druid.cache.type`有效类型的配置。 | `caffeine`                 |
| `druid.cache.l1.*`       | 可以使用此前缀设置对给定类型的L1缓存有效的任何属性。例如，如果您使用的是`caffeine`L1缓存，请指定`druid.cache.l1.sizeInBytes`设置其大小。 | 默认值与给定缓存类型相同。 |
| `druid.cache.l2.*`       | L2缓存设置的前缀，请参阅L1的说明。                           | 默认值与给定缓存类型相同。 |
| `druid.cache.useL2`      | 一个布尔值，指示是否查询L2缓存，如果它是L1中的未命中。将此配置为`false`历史节点是有意义的，如果L2是远程缓存`memcached`，并且此缓存也用于代理，因为在这种情况下，如果查询达到历史，则意味着代理在相同的结果中找不到相应的结果远程缓存，因此从历史到远程缓存的查询保证是一个未命中。 | `true`                     |
| `druid.cache.populateL2` | 一个布尔值，指示是否将结果放入L2缓存中。                     | `true`                     |

## 一般查询配置

本节描述控制Druid查询类型行为的配置，适用于代理，历史和中间管理器节点。

### TopN查询配置

| 属性                                | 描述                                                         | 默认 |
| ----------------------------------- | ------------------------------------------------------------ | ---- |
| `druid.query.topN.minTopNThreshold` | 有关详细信息，请参阅[TopN别名](http://druid.io/docs/0.12.3/querying/topnquery.html#aliasing)。 | 1000 |

### 搜索查询配置

| 属性                                | 描述                     | 默认       |
| ----------------------------------- | ------------------------ | ---------- |
| `druid.query.search.maxSearchLimit` | 要返回的最大搜索结果数。 | 1000       |
| `druid.query.search.searchStrategy` | 默认搜索查询策略。       | useIndexes |

### 段元数据查询配置

| 属性                                               | 描述                                                         | 默认                       |
| -------------------------------------------------- | ------------------------------------------------------------ | -------------------------- |
| `druid.query.segmentMetadata.defaultHistory`       | 如果查询中未指定间隔，请在ISO8601格式指定的最新段结束时间之前使用defaultHistory的默认间隔。此属性还控制GET / druid / v2 / datasources / {dataSourceName}交互用于检索数据源维度/指标的默认间隔的持续时间。 | P1W                        |
| `druid.query.segmentMetadata.defaultAnalysisTypes` | 这可以用于为所有段元数据查询设置默认分析类型，这可以在进行查询时被覆盖 | [“基数”，“间隔”，“minmax”] |

### GroupBy查询配置

本节介绍groupBy查询的配置。您可以`runtime.properties`在broker，historical和MiddleManager节点上的文件中设置运行时属性。您可以通过[查询上下文](http://druid.io/docs/0.12.3/querying/query-context.html)设置查询上下文参数。

#### groupBy v2的配置

支持的运行时属性

| 属性                                           | 描述                                                         | 默认      |
| ---------------------------------------------- | ------------------------------------------------------------ | --------- |
| `druid.query.groupBy.maxMergingDictionarySize` | 合并期间用于字符串字典的最大堆空间量（大约）。当字典超过此大小时，将触发溢出到磁盘。 | 亿        |
| `druid.query.groupBy.maxOnDiskStorage`         | 合并缓冲区或字典填满时，用于将结果集溢出到磁盘的最大磁盘空间量（每次查询）。超过此限制的查询将失败。设置为零以禁用磁盘溢出。 | 0（禁用） |

支持的查询上下文：

| 键                         | 描述                                                         |
| -------------------------- | ------------------------------------------------------------ |
| `maxMergingDictionarySize` | 可用于降低`druid.query.groupBy.maxMergingDictionarySize`此查询的值。 |
| `maxOnDiskStorage`         | 可用于降低`druid.query.groupBy.maxOnDiskStorage`此查询的值。 |

### 高级配置

#### 所有groupBy策略的常见配置

支持的运行时属性

| 属性                                  | 描述                   | 默认 |
| ------------------------------------- | ---------------------- | ---- |
| `druid.query.groupBy.defaultStrategy` | 默认groupBy查询策略。  | V2   |
| `druid.query.groupBy.singleThreaded`  | 使用单个线程合并结果。 | 假   |

支持的查询上下文：

| 键                        | 描述                                                  |
| ------------------------- | ----------------------------------------------------- |
| `groupByStrategy`         | 覆盖`druid.query.groupBy.defaultStrategy`此查询的值。 |
| `groupByIsSingleThreaded` | 覆盖`druid.query.groupBy.singleThreaded`此查询的值。  |

#### GroupBy v2配置

支持的运行时属性

| 属性                                              | 描述                                                         | 默认      |
| ------------------------------------------------- | ------------------------------------------------------------ | --------- |
| `druid.query.groupBy.bufferGrouperInitialBuckets` | 用于分组结果的堆外哈希表中的初始桶数。设置为0以使用合理的默认值（1024）。 | 0         |
| `druid.query.groupBy.bufferGrouperMaxLoadFactor`  | 用于分组结果的堆外哈希表的最大加载因子。当负载系数超过此大小时，表将增长或溢出到磁盘。设置为0以使用合理的默认值（0.7）。 | 0         |
| `druid.query.groupBy.forceHashAggregation`        | 强制使用基于散列的聚合。                                     | 假        |
| `druid.query.groupBy.intermediateCombineDegree`   | 组合树中组合在一起的中间节点数。更高的度数将需要更少的线程，如果服务器具有足够强大的CPU核心，则可以通过减少太多线程的开销来帮助提高查询性能。 | 8         |
| `druid.query.groupBy.numParallelCombineThreads`   | 提示并行组合线程的数量。这应该大于1以打开并行组合功能。用于并行组合的实际线程数是min（`druid.query.groupBy.numParallelCombineThreads`，`druid.processing.numThreads`）。 | 1（禁用） |

支持的查询上下文：

| 键                            | 描述                                                         | 默认 |
| ----------------------------- | ------------------------------------------------------------ | ---- |
| `bufferGrouperInitialBuckets` | 覆盖`druid.query.groupBy.bufferGrouperInitialBuckets`此查询的值。 | 没有 |
| `bufferGrouperMaxLoadFactor`  | 覆盖`druid.query.groupBy.bufferGrouperMaxLoadFactor`此查询的值。 | 没有 |
| `forceHashAggregation`        | 覆盖的值`druid.query.groupBy.forceHashAggregation`           | 没有 |
| `intermediateCombineDegree`   | 覆盖的值`druid.query.groupBy.intermediateCombineDegree`      | 没有 |
| `numParallelCombineThreads`   | 覆盖的值`druid.query.groupBy.numParallelCombineThreads`      | 没有 |
| `sortByDimsFirst`             | 首先按维度值排序结果，然后按时间戳排序。                     | 假   |
| `forceLimitPushDown`          | 当orderby中的所有字段都是分组键的一部分时，代理会将限制应用程序下推到历史节点。当排序顺序使用不在分组键中的字段时，应用此优化可能会导致具有未知准确度的近似结果，因此在这种情况下默认情况下禁用此优化。启用此上下文标志会打开包含非分组键列的limit / orderbys的限制下推。 | 假   |

#### GroupBy v1配置

支持的运行时属性

| 属性                                      | 描述                                                         | 默认   |
| ----------------------------------------- | ------------------------------------------------------------ | ------ |
| `druid.query.groupBy.maxIntermediateRows` | 每段细分引擎的最大中间行数。这是一个不会施加硬限制的调整参数; 相反，它可能会将合并工作从每段引擎转移到整体合并索引。超过此限制的查询不会失败。 | 50000  |
| `druid.query.groupBy.maxResults`          | 最大结果数。超过此限制的查询将失败。                         | 500000 |

支持的查询上下文：

| 键                    | 描述                                                         | 默认 |
| --------------------- | ------------------------------------------------------------ | ---- |
| `maxIntermediateRows` | 可用于降低`druid.query.groupBy.maxIntermediateRows`此查询的值。 | 没有 |
| `maxResults`          | 可用于降低`druid.query.groupBy.maxResults`此查询的值。       | 没有 |
| `useOffheap`          | 设置为true以在合并结果时在堆外存储聚合。                     | 假   |

## 实时节点

可以在[此处](http://druid.io/docs/0.12.3/configuration/realtime.html)找到已弃用的实时节点的配置。