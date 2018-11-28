# TLS支持

# 一般配置

| 属性                        | 描述                   | 默认    |
| --------------------------- | ---------------------- | ------- |
| `druid.enablePlaintextPort` | 启用/禁用HTTP连接器。  | `true`  |
| `druid.enableTlsPort`       | 启用/禁用HTTPS连接器。 | `false` |

虽然不推荐，但可以同时启用HTTP和HTTPS连接器，并且可以使用每个节点上的`druid.plaintextPort` 和`druid.tlsPort`属性配置相应的端口。请参阅`Configuration`各个节点的部分以检查这些端口的有效值和默认值。

# Jetty服务器TLS配置

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

# 德鲁伊在TLS上的内部沟通

只要有可能，德鲁伊节点就会使用HTTPS相互通信。要启用此通信，需要使用能够验证服务器证书的正确[SSLContext](http://docs.oracle.com/javase/8/docs/api/javax/net/ssl/SSLContext.html)配置Druid的HttpClient ，否则通信将失败。

因为，有多种方法可以配置SSLContext，默认情况下，Druid会在创建HttpClient时查找SSLContext Guice绑定的实例。这种绑定可以通过编写 可以提供SSLContext实例的[Druid扩展](http://druid.io/docs/0.12.3/development/extensions.html)来实现。德鲁伊配备了目前的简单扩展[这里](http://druid.io/docs/0.12.3/development/extensions-core/simple-client-sslcontext.html) 应该是大多数简单的情况下足够有用的，看到[这](http://druid.io/docs/0.12.3/operations/including-extensions.html)对如何将扩展。如果此扩展程序不满足要求，请按照扩展程序[实施](https://github.com/druid-io/druid/tree/master/extensions-core/simple-client-sslcontext) 创建您自己的扩展程序。

# 升级与Overlord或Coordinator交互的客户端

当Druid Coordinator / Overlord同时启用HTTP和HTTPS并且客户端向非领导节点发送请求时，Client始终会重定向到leader节点上的HTTPS端点。因此，应首先升级客户端以便能够处理重定向到HTTPS。然后应该升级并配置Druid Overlord / Coordinator以运行HTTP和HTTPS端口。然后应该通过HTTPS端点更改客户端配置以引用德鲁伊协调员/霸主，然后禁用德鲁伊协调员/宿主上的HTTP端口。