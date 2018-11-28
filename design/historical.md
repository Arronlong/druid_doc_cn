# 历史节点

### 组态

对于历史节点配置，请参阅[历史配置](http://druid.io/docs/0.12.3/configuration/index.html#historical)。

### HTTP端点

有关Historical支持的API端点列表，请参阅[API参考](http://druid.io/docs/0.12.3/operations/api-reference.html#historical)。

### 运行

```text
io.druid.cli.Main server historical
```

### 加载和服务细分

每个历史节点都与Zookeeper保持连接，并监视一组可配置的Zookeeper路径以获取新的段信息。历史节点不直接相互通信或与协调器节点直接通信，而是依靠Zookeeper进行协调。

该[协调](http://druid.io/docs/0.12.3/design/coordinator.html)节点负责历史节点分配新的细分市场。通过在与历史节点相关联的加载队列路径下创建短暂的Zookeeper条目来完成分配。有关协调器如何将段分配给历史节点的更多信息，请参阅[协调员](http://druid.io/docs/0.12.3/design/coordinator.html)。

当历史节点在其加载队列路径中注意到新的加载队列条目时，它将首先检查本地磁盘目录（缓存）以获取有关段的信息。如果缓存中不存在有关该段的信息，则历史节点将下载有关要从Zookeeper提供的新段的元数据。此元数据包括有关段在深层存储中的位置以及如何解压缩和处理段的规范。有关细分元数据和德鲁伊细分市场的更多信息，请参阅[细分](http://druid.io/docs/0.12.3/design/segments.html)。一旦历史节点完成处理段，则在Zookeeper中在与节点相关联的服务段路径下宣布该段。此时，该段可用于查询。

### 从缓存加载和提供细分

回想一下，当历史节点在其加载队列路径中注意到新的段条目时，历史节点首先检查其本地磁盘上的可配置缓存目录，以查看该段是否先前已下载过。如果已存在本地缓存条目，则历史节点将直接从磁盘读取段二进制文件并加载该段。

首次启动历史节点时，也会利用段缓存。启动时，历史节点将搜索其缓存目录，并立即加载并提供找到的所有段。此功能允许历史节点在上线后立即查询。

### 查询细分

有关查询历史节点的更多信息，请参阅[查询](http://druid.io/docs/0.12.3/querying/querying.html)。

可以将历史记录配置为记录和报告其服务的每个查询的度量标准。