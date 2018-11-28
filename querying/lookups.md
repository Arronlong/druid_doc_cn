# 查找

查找是一个[实验性](http://druid.io/docs/0.12.3/development/experimental.html)功能。

查找是Druid中的一个概念，其中维度值（可选）被替换为新值，允许类似连接的功能。在Druid中应用查找类似于在数据仓库中加入维度表。有关更多信息，请参阅 [尺寸规格](http://druid.io/docs/0.12.3/querying/dimensionspecs.html)。出于这些文档的目的，“密钥”指的是要匹配的维度值，“值”指的是其替换。因此，如果你想映射 `appid-12345`到`Super Mega Awesome App`那么键将是`appid-12345`，值将是 `Super Mega Awesome App`。

值得注意的是，查找不仅支持将键一对一映射到唯一值（例如国家/地区代码和国家/地区名称）的用例，还支持多个ID映射到相同值的用例，例如多个app-id映射到单个客户经理。当查找是一对一时，德鲁伊能够在查询时应用其他优化; 请参阅下面的[查询执行](http://druid.io/docs/0.12.3/querying/lookups.html#query-execution)以获取更多详

查找没有历史记录。他们总是使用当前数据。这意味着，如果特定app-id的主要客户经理发生更改，并且您发出查询以将app-id存储到客户经理关系，则会返回该应用程序ID的当前帐户管理员。您查询的时间范围。

如果您需要数据时间范围敏感查找，则在查询时动态不支持此类用例，并且此类数据属于原始非规范化数据以供在Druid中使用。

非常小的查找（大约几十到几百的键的计数）可以在查询时作为按照[尺寸规格](http://druid.io/docs/0.12.3/querying/dimensionspecs.html)的“地图”查找传递。

其他查找类型可用作扩展，包括：

- 通过[lookups -cached-global](http://druid.io/docs/0.12.3/development/extensions-core/lookups-cached-global.html)从本地文件，远程URI或JDBC进行全局缓存查找。
- 从Kafka主题到[kafka-extraction-namespace的](http://druid.io/docs/0.12.3/development/extensions-core/kafka-extraction-namespace.html)全局缓存查找。

## 查询执行

在执行涉及查找的聚合查询时，Druid可以决定在扫描和聚合行时应用查找，或者在聚合完成后应用它们。在聚合完成后应用查找更有效，因此如果可以，德鲁伊会这样做。德鲁伊通过检查查找是否标记为“单射”来决定这一点。通常，您应该为任何自然一对一的查找设置此属性，以允许Druid尽可能快地运行您的查询。

内射查找应包括可能显示在数据集中的*所有*可能键，并且还应将所有键映射到 *唯一值*。这很重要，因为非内射查找可以将不同的键映射到相同的值，这必须在聚合期间考虑，以免查询结果包含两个应该聚合为一个的结果值。

此查找是单射的（假设它包含来自数据的所有可能的键）：

```text
1 -> Foo
2 -> Bar
3 -> Billy
```

但是这个不是，因为“2”和“3”都映射到同一个键：

```text
1 -> Foo
2 -> Bar
3 -> Bar
```

要告诉德鲁伊你的查找是单射的，你必须`"injective" : true`在查找配置中指定。德鲁伊不会自动检测到这一点。

## 动态配置

动态查找配置是一项[实验性](http://druid.io/docs/0.12.3/development/experimental.html)功能。不再支持静态配置。

以下文档说明了可通过协调器访问的群集范围配置的行为。配置通过服务器的“层”概念传播。“层”被定义为应该接收一组查找的一组服务。例如，您可能将所有历史记录都包含在其中`__default`，并且Peons将成为它们所负责的数据源的各个层的一部分。查找层完全独立于历史层。

通过以下URI模板使用JSON访问这些配置

```text
http://<COORDINATOR_IP>:<PORT>/druid/coordinator/v1/lookups/config/{tier}/{id}
```

假设下面的所有URI都有`http://<COORDINATOR_IP>:<PORT>`前缀。

如果你以前从来没有配置查找，您必须张贴一个空的JSON对象`{}`来`/druid/coordinator/v1/lookups/config`初始化配置。

这些端点将返回以下结果之一：

- 404，如果找不到资源
- 400，如果请求的格式有问题
- 202，如果请求被异步接受（`POST`和`DELETE`）
- 如果请求成功，则为200（`GET`仅）

## 配置传播行为

配置由协调器传播到查询服务节点（代理/路由器/ peon /历史）。查询服务节点具有用于管理节点上的查找的内部API以及协调器使用的API。协调器定期检查是否有任何节点需要加载/删除查找并适当更新它们。

# 用于配置查找的API

## 批量更新

可以通过发布JSON对象来批量更新查找`/druid/coordinator/v1/lookups/config`。json对象的格式如下：

```json
{
    "<tierName>": {
        "<lookupName>": {
          "version": "<version>",
          "lookupExtractorFactory": {
            "type": "<someExtractorFactoryType>",
            "<someExtractorField>": "<someExtractorValue>"
          }
        }
    }
}
```

请注意，“version”是用户分配的任意字符串，在对现有查找进行更新时，用户需要指定按字典顺序排列的更高版本。

例如，配置可能类似于：

```json
{
  "__default": {
    "country_code": {
      "version": "v0",
      "lookupExtractorFactory": {
        "type": "map",
        "map": {
          "77483": "United States"
        }
      }
    },
    "site_id": {
      "version": "v0",
      "lookupExtractorFactory": {
        "type": "cachedNamespace",
        "extractionNamespace": {
          "type": "jdbc",
          "connectorConfig": {
            "createTables": true,
            "connectURI": "jdbc:mysql:\/\/localhost:3306\/druid",
            "user": "druid",
            "password": "diurd"
          },
          "table": "lookupTable",
          "keyColumn": "country_id",
          "valueColumn": "country_name",
          "tsColumn": "timeColumn"
        },
        "firstCacheTimeout": 120000,
        "injective": true
      }
    },
    "site_id_customer1": {
      "version": "v0",
      "lookupExtractorFactory": {
        "type": "map",
        "map": {
          "847632": "Internal Use Only"
        }
      }
    },
    "site_id_customer2": {
      "version": "v0",
      "lookupExtractorFactory": {
        "type": "map",
        "map": {
          "AHF77": "Home"
        }
      }
    }
  },
  "realtime_customer1": {
    "country_code": {
      "version": "v0",
      "lookupExtractorFactory": {
        "type": "map",
        "map": {
          "77483": "United States"
        }
      }
    },
    "site_id_customer1": {
      "version": "v0",
      "lookupExtractorFactory": {
        "type": "map",
        "map": {
          "847632": "Internal Use Only"
        }
      }
    }
  },
  "realtime_customer2": {
    "country_code": {
      "version": "v0",
      "lookupExtractorFactory": {
        "type": "map",
        "map": {
          "77483": "United States"
        }
      }
    },
    "site_id_customer2": {
      "version": "v0",
      "lookupExtractorFactory": {
        "type": "map",
        "map": {
          "AHF77": "Home"
        }
      }
    }
  }
}
```

地图中的所有条目都将更新现有条目。不会删除任何条目。

## 更新查找

A `POST`到特定的查找提取器工厂`/druid/coordinator/v1/lookups/config/{tier}/{id}`将更新该特定的提取器工厂。

例如，帖子`/druid/coordinator/v1/lookups/config/realtime_customer1/site_id_customer1`可能包含以下内容：

```json
{
  "version": "v1",
  "lookupExtractorFactory": {
    "type": "map",
    "map": {
      "847632": "Internal Use Only"
    }
  }
}
```

这将取代上面定义中的`site_id_customer1`查找`realtime_customer1`。

## 获取查询

A `GET`到特定的查找提取器工厂是通过完成的`/druid/coordinator/v1/lookups/{tier}/{id}`

使用现有例如，`GET`以`/druid/coordinator/v1/lookups/config/realtime_customer2/site_id_customer2`应返回

```json
{
  "version": "v1",
  "lookupExtractorFactory": {
    "type": "map",
    "map": {
      "AHF77": "Home"
    }
  }
}
```

## 删除查找

一`DELETE`来`/druid/coordinator/v1/lookups/config/{tier}/{id}`也可以从集群删除查找。

## 列表层名称

A `GET`to `/druid/coordinator/v1/lookups/config`将返回动态配置中的已知层名称列表。要发现群集中当前处于活动状态的层列表**而不是**动态配置中已知的层列表，`discover=true`可以按参数添加该参数`/druid/coordinator/v1/lookups?discover=true`。

## 列出查找名称

A `GET`to `/druid/coordinator/v1/lookups/config/{tier}`将返回该层的已知查找名称列表。

# 与配置的查找状态相关的其他API

这些端点可用于将已配置查找的传播状态提供给查找节点（如历史记录）。

## 列出所有查找的加载状态

`GET /druid/coordinator/v1/lookups/status`带有可选的查询参数`detailed`。

## 列出层中查找的加载状态

`GET /druid/coordinator/v1/lookups/status/{tier}`带有可选的查询参数`detailed`。

## 列出单个查找的加载状态

`GET /druid/coordinator/v1/lookups/status/{tier}/{lookup}`带有可选的查询参数`detailed`。

## 列出所有节点的查找状态

`GET /druid/coordinator/v1/lookups/nodeStatus`使用可选的查询参数`discover`来发现来自zookeeper的层或配置的查找层。

## 列出层中节点的查找状态

```
GET /druid/coordinator/v1/lookups/nodeStatus/{tier}
```

## 列出单个节点的查找状态

```
GET /druid/coordinator/v1/lookups/nodeStatus/{tier}/{host:port}
```

# 内部API

Peon，Router，Broker和Historical节点都具有使用查找配置的能力。这些节点使用内部API来列出/加载/删除它们的查找`/druid/listen/v1/lookups`。它们遵循与返回值相同的约定作为群集范围的动态配置。以下端点可用于调试目的，但不能用于其他目的。

## 获取查询

A `GET`到节点at `/druid/listen/v1/lookups`将返回节点上当前活动的所有查找的json映射。返回值将是对其提取器工厂的查找的json映射。

```json
{
  "site_id_customer2": {
    "version": "v1",
    "lookupExtractorFactory": {
      "type": "map",
      "map": {
        "AHF77": "Home"
      }
    }
  }
}
```

## 获取查询

A `GET`到节点at `/druid/listen/v1/lookups/some_lookup_name`将返回LookupExtractorFactory以查找标识`some_lookup_name`。返回值将是工厂的json表示形式。

```json
{
  "version": "v1",
  "lookupExtractorFactory": {
    "type": "map",
    "map": {
      "AHF77": "Home"
    }
  }
}
```

# 组态

请参阅[查找动态配置](http://druid.io/docs/0.12.3/configuration/index.html#lookups-dynamic-configuration)以获取协调器配置。

要配置Broker / Router / Historical / Peon以宣布自己作为查找层的一部分，请使用该`druid.zk.paths.lookupTier`属性。

| 属性                                  | 描述                                                         | 默认        |
| ------------------------------------- | ------------------------------------------------------------ | ----------- |
| `druid.lookup.lookupTier`             | 用于**查找**此节点的层。这与其他层无关。                     | `__default` |
| `druid.lookup.lookupTierIsDatasource` | 对于诸如索引服务任务之类的事情，数据源在任务的运行时属性中传递。此选项从与任务的数据源相同的值中获取tierName。建议仅将其用作索引服务的peon选项（如果有的话）。如果为true，则`druid.lookup.lookupTier`不得指定 | `"false"`   |

要配置动态配置管理器的行为，请在协调器上使用以下属性：

| 属性                                   | 描述                                         | 默认               |
| -------------------------------------- | -------------------------------------------- | ------------------ |
| `druid.manager.lookups.hostTimeout`    | 处理请求的超时（以毫秒为单位）PER HOST       | `2000`（2秒）      |
| `druid.manager.lookups.allHostTimeout` | 超时（以ms为单位）完成所有节点上的查找管理。 | `900000`（15分钟） |
| `druid.manager.lookups.period`         | 管理周期之间暂停多长时间                     | `120000`（2分钟）  |
| `druid.manager.lookups.threadPoolSize` | 可以同时管理的服务节点数                     | `10`               |

## 在重新启动时保存配置

可以跨重新启动保存配置，以便节点不必等待协调器操作重新填充其查找。为此，设置以下属性：

| 属性                                     | 描述                                                         | 默认          |
| ---------------------------------------- | ------------------------------------------------------------ | ------------- |
| `druid.lookup.snapshotWorkingDir`        | 用于存储当前查找配置快照的工作路径，将此属性保留为null将禁用快照/引导实用程序 | 空值          |
| `druid.lookup.enableLookupSyncOnStartup` | 启动时使用协调器启用查找同步过程。可查询节点将从协调器获取并加载查找，而不是等待协调器为它们加载查找。如果群集中未配置查找，则用户可以选择禁用此选项。 | 真正          |
| `druid.lookup.numLookupLoadingThreads`   | 启动时并行加载查找的线程数。启动完成后，此线程池将被销毁。它不会在JVM的生命周期中保留 | 可用处理器/ 2 |
| `druid.lookup.coordinatorFetchRetries`   | 在启动时同步期间重试从协调器获取查找bean列表的次数。         | 3             |
| `druid.lookup.lookupStartRetries`        | 在启动时同步或运行时期间重试启动每次查找的次数。             | 3             |

## 反思一个查找

查找实现可以通过实现提供一些内省功能`LookupIntrospectHandler`。用户将发送请求以`/druid/lookups/v1/introspect/{lookupId}`启用给定查找的内省。

例如，您可以通过`GET`向`/druid/lookups/v1/introspect/{lookupId}/keys"`或发出请求来列出基于地图的查找的所有键/值`/druid/lookups/v1/introspect/{lookupId}/values"`

## 德鲁伊版本0.10.0升级到0.10.1升级/降级

总体德鲁伊群集查找配置保留在元数据存储中，并且单个查找节点可选地在磁盘上保存已加载查找的快照。如果从druid版本0.10.0升级到0.10.1，则会自动处理所有持久元数据的迁移。如果从0.10.1降级到0.9.0然后在0.10.1运行时查找通过协调器完成的更新，则会丢失。