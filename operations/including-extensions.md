# 加载扩展程序

## 加载核心扩展

德鲁伊开箱即用捆绑所有[核心扩展](http://druid.io/docs/0.12.3/development/extensions.html#core-extensions)。请参阅选项[的扩展名列表](http://druid.io/docs/0.12.3/development/extensions.html#core-extensions)。您可以通过将其名称添加到common.runtime.properties`druid.extensions.loadList`属性来加载捆绑的扩展 。例如，要加载*postgresql-metadata-storage*和 *druid-hdfs-storage*扩展，请使用以下配置：

```text
druid.extensions.loadList=["postgresql-metadata-storage", "druid-hdfs-storage"]
```

这些扩展位于`extensions`发行版的目录中。

Druid捆绑了两组配置：一组用于[快速入门](http://druid.io/docs/0.12.3/tutorials/quickstart.html)，另一组用于[集群配置](http://druid.io/docs/0.12.3/tutorials/cluster.html)。确保为您的设置更新正确的common.runtime.properties。

由于许可，mysql-metadata-storage扩展不与默认的Druid tarball打包在一起。为了获得它，您可以从[druid.io](http://druid.io/downloads.html)下载它，然后解压缩并将其移动到extensions目录中。确保在loadList配置中包含扩展名。

## 加载社区和第三方扩展（contrib扩展）

您还可以加载尚未与Druid捆绑的社区和第三方扩展程序。为此，首先下载扩展，然后将其安装到您的`extensions`目录中。您可以直接从他们的经销商处下载扩展程序，或者如果他们可以从Maven获得，那么随附的[pull-deps](http://druid.io/docs/0.12.3/operations/pull-deps.html)可以为您下载。要使用*pull-deps*，请在表单中指定扩展名的完整Maven坐标`groupId:artifactId:version`。例如，对于（假设的）扩展*com.example：druid-example-extension：1.0.0*，运行：

```text
java \
  -cp "lib/*" \
  -Ddruid.extensions.directory="extensions" \
  -Ddruid.extensions.hadoopDependenciesDir="hadoop-dependencies" \
  io.druid.cli.Main tools pull-deps \
  --no-default-hadoop \
  -c "com.example:druid-example-extension:1.0.0"
```

您只需安装一次扩展程序。然后，在common.runtime.properties中添加`"druid-example-extension"`以`druid.extensions.loadList`指示Druid加载扩展。

请确保正确设置[此处](http://druid.io/docs/0.12.3/configuration/index.html#extensions)列出的所有与扩展相关的配置属性。

几乎每个[社区扩展](http://druid.io/docs/0.12.3/development/extensions.html#community-extensions)的Maven groupId 都是io.druid.extensions.contrib。artifactId是扩展名，版本是最新的Druid稳定版。

## 从类路径加载扩展

如果在运行时将扩展jar添加到类路径，Druid也会将其加载到系统中。这种机制相对容易推理，但它也意味着您必须确保类路径上的所有依赖项都是兼容的。也就是说，Druid在使用此方法维护类加载器隔离时不做任何规定，因此您必须确保类路径上的jar是相互兼容的。