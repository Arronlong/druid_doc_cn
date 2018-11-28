# 密码提供商

德鲁伊需要一些密码来访问各种安全系统，如元数据存储，包含服务器证书的密钥库等。所有这些密码都具有与之关联的相应运行时属性，例如`druid.metadata.storage.connector.password`对应于元数据存储密码。

默认情况下，用户可以直接为明文设置这些运行时属性的密码，例如`druid.metadata.storage.connector.password=pwd`设置元数据存储密码以供Druid用于连接元数据存储`pwd`。除此之外，用户可以使用环境变量以下列方式获取密码 -

环境变量密码提供者通过查看指定的环境变量来提供密码。使用此选项以避免在runtime.properties文件中指定密码。例如

```json
{ "type": "environment", "variable": "METADATA_STORAGE_PASSWORD" }
```

值如下所述。

| 领域       | 类型 | 描述               | 需要               |
| ---------- | ---- | ------------------ | ------------------ |
| `type`     | 串   | 密码提供者类型     | 是： `environment` |
| `variable` | 串   | 环境变量来读取密码 | 是                 |

但是，很多时候用户可能希望以他们自己的方式在德鲁伊过程的运行时期间可选地安全地获取密码。Druid允许用户实现自己的`PasswordProvider`界面并创建一个Druid扩展来在Druid流程启动时注册这个实现。请查看此[页面](http://druid.io/docs/0.12.3/development/modules.html)上的“添加新的密码提供程序实现” 以了解更多信息。

要使用此实现，只需将相关的密码运行时属性设置为与环境变量密码提供程序类似的类似 -

```json
{ "type": "<registered_password_provider_name>", "<jackson_property>": "<value>", ... }
```