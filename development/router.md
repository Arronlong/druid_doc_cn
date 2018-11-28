# 路由器节点

如果您的德鲁伊群集已经达到太字节范围，那么您应该只需要路由器节点。路由器节点可用于将查询路由到不同的代理节点。默认情况下，代理根据[规则](http://druid.io/docs/0.12.3/operations/rule-configuration.html)的设置方式路由查询。例如，如果将最近1个月的数据加载到`hot`群集中，则可以将最近一个月内的查询路由到一组专用代理。超出此范围的查询将路由到另一组代理。此设置提供查询隔离，以便查询更重要的数据不会受到对不太重要的数据的查询的影响。

## 运行

```text
io.druid.cli.Main server router
```

## 示例生产配置

在这个例子中，我们的生产集群中有两层：`hot`和`_default_tier`。`hot`层的查询通过一`broker-hot`组代理进行路由，并`_default_tier`通过一`broker-cold`组代理路由查询。如果发生任何异常或网络问题，查询将路由到`broker-cold`代理集。在我们的示例中，我们使用c3.2xlarge EC2节点运行。我们假设`common.runtime.properties`已经存在。

JVM设置：

```text
-server
-Xmx13g
-Xms13g
-XX:NewSize=256m
-XX:MaxNewSize=256m
-XX:+UseConcMarkSweepGC
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-XX:+UseLargePages
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/mnt/galaxy/deploy/current/
-Duser.timezone=UTC
-Dfile.encoding=UTF-8
-Djava.io.tmpdir=/mnt/tmp

-Dcom.sun.management.jmxremote.port=17071
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
```

Runtime.properties：

```text
druid.host=#{IP_ADDR}:8080
druid.port=8080
druid.service=druid/router

druid.router.defaultBrokerServiceName=druid:broker-cold
druid.router.coordinatorServiceName=druid:coordinator
druid.router.tierToBrokerMap={"hot":"druid:broker-hot","_default_tier":"druid:broker-cold"}
druid.router.http.numConnections=50
druid.router.http.readTimeout=PT5M

# Number of threads used by the router proxy http client
druid.router.http.numMaxThreads=100

druid.server.http.numThreads=100
```

## 运行时配置

路由器模块使用[Configuration中的](http://druid.io/docs/0.12.3/configuration/index.html)几个默认模块，并具有以下一组配置：

| 属性                                     | 可能的值                                                     | 描述                                     | 默认                                             |
| ---------------------------------------- | ------------------------------------------------------------ | ---------------------------------------- | ------------------------------------------------ |
| `druid.router.defaultBrokerServiceName`  | 任何字符串。                                                 | 连接的默认代理以防服务发现失败。         | 德鲁伊/经纪商                                    |
| `druid.router.tierToBrokerMap`           | 代理名称的层的有序JSON映射。经纪人的优先权基于订购。         | 对特定层数据的查询将路由到其相应的代理。 | {“_ default”：““}                                |
| `druid.router.defaultRule`               | 任何字符串。                                                 | 所有数据源的默认规则。                   | “_默认”                                          |
| `druid.router.rulesEndpoint`             | 任何字符串。                                                 | 协调器端点从中提取规则。                 | “/德鲁伊/协调员/ V1 /规则”                       |
| `druid.router.coordinatorServiceName`    | 任何字符串。                                                 | 协调器的服务发现名称。                   | 德鲁伊/协调员                                    |
| `druid.router.pollPeriod`                | 任何ISO8601持续时间。                                        | 轮询新规则的频率。                       | PT1M                                             |
| `druid.router.strategies`                | 一个有序的JSON对象数组。                                     | 用于路由的所有自定义策略。               | [{ “类型”： “timeBoundary”}，{ “类型”： “优先”}] |
| `druid.router.avatica.balancer.type`     | 表示AvaticaConnectionBalancer名称的字符串                    | 用于在代理之间平衡Avatica查询的类        | rendezvousHash                                   |
| `druid.router.http.maxRequestBufferSize` | 用于在将请求转发给代理时写入请求的缓冲区的最大大小。这应该设置为至少代理上允许的maxHeaderSize | 8 * 1024                                 |                                                  |

## 路由策略

路由器有一个可配置的策略列表，用于选择将查询路由到哪个代理。策略的顺序很重要，因为只要匹配策略条件，就会选择代理。

### timeBoundary

```json
{
  "type":"timeBoundary"
}
```

包含此策略意味着所有timeBoundary查询始终路由到优先级最高的代理。

### 优先

```json
{
  "type":"priority",
  "minPriority":0,
  "maxPriority":1
}
```

优先级设置为小于minPriority的查询将路由到最低优先级代理。优先级设置为大于maxPriority的查询将路由到优先级最高的代理。默认情况下，minPriority为0且maxPriority为1.使用这些默认值，如果发送优先级为0（默认查询优先级为0）的查询，则查询将跳过优先级选择逻辑。

### JavaScript的

允许使用JavaScript函数定义任意路由规则。该函数传递配置和要执行的查询，并返回应路由到的层，或者为默认层返回null。

*示例*：将包含三个以上聚合器的查询发送到优先级最低的代理的函数。

```json
{
  "type" : "javascript",
  "function" : "function (config, query) { if (query.getAggregatorSpecs && query.getAggregatorSpecs().size() >= 3) { var size = config.getTierToBrokerMap().values().size(); if (size > 0) { return config.getTierToBrokerMap().values().toArray()[size-1] } else { return config.getDefaultBrokerServiceName() } } else { return null } }"
}
```

默认情况下禁用基于JavaScript的功能。有关使用德鲁伊JavaScript功能的[指南](http://druid.io/docs/0.12.3/development/javascript.html)，请参阅德鲁伊[JavaScript编程指南](http://druid.io/docs/0.12.3/development/javascript.html)，包括如何启用它的说明。

## Avatica查询平衡

具有给定连接ID的所有Avatica JDBC请求必须路由到同一代理，因为Druid代理不会彼此共享连接状态。

为实现这一目标，德鲁伊提供了两个内置平衡器，它们分别使用集合散列和请求连接ID的一致散列来为经纪人分配请求。

请注意，当使用多个路由器时，所有路由器都应具有相同的平衡器配置，以确保它们做出相同的路由决策。

### Rendezvous Hash Balancer

此平衡器使用Avatica请求的连接ID上的[Rendezvous Hashing](https://en.wikipedia.org/wiki/Rendezvous_hashing)将请求分配给代理。

要使用此平衡器，请指定以下属性：

```text
druid.router.avatica.balancer.type=rendezvousHash
```

如果未`druid.router.avatica.balancer`设置任何属性，则路由器也将默认使用Rendezvous Hash Balancer。

### 一致的哈希平衡器

此平衡器使用Avatica请求的连接ID上的[Consistent Hashing](https://en.wikipedia.org/wiki/Consistent_hashing)将请求分配给代理。

要使用此平衡器，请指定以下属性：

```text
druid.router.avatica.balancer.type=consistentHash
```

这是一个非默认实现，用于实验目的。一致的hasher在初始化时和设置的代理更改时具有更长的设置时间，但在使用5个代理进行测试时具有比rendezous hasher更快的代理分配时间。这些实现的基准测试已在`ConsistentHasherBenchmark`和中提供`RendezvousHasherBenchmark`。一致的hasher也需要锁定，而会合的hasher则不需要。

## HTTP端点

路由器节点公开了几个HTTP端点以进行交互。

### 得到

- `/status`

返回Druid版本，加载的扩展，使用的内存，总内存和有关节点的其他有用信息。

- `/druid/v2/datasources`

返回可查询数据源的列表。

- `/druid/v2/datasources/{dataSourceName}`

返回数据源的维度和指标。

- `/druid/v2/datasources/{dataSourceName}/dimensions`

返回数据源的维度。

- `/druid/v2/datasources/{dataSourceName}/metrics`

返回数据源的度量标准。