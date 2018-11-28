### 从源代码构建

你可以直接从源代码构建德鲁伊。请注意，这些说明适用于构建最新稳定的德鲁伊。要在master中构建最新代码，请按照[此处](https://github.com/druid-io/druid/blob/master/docs/content/development/build.md)的说明进行操作。

构建德鲁伊需要以下内容： - [JDK 7](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html) 或[JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) - [Maven版本3.x.](http://maven.apache.org/download.cgi)

为此，请运行以下命令：

```text
git clone git@github.com:druid-io/druid.git
cd druid
mvn clean package
```

这将编译项目并在其下创建Druid二进制分发tar `distribution/target/druid-VERSION-bin.tar.gz`。

这也将创建一个包含`mysql-metadata-storage`扩展名 的tarball `distribution/target/mysql-metadata-storage-bin.tar.gz`。如果你想要装载德鲁伊`mysql-metadata-storage`，你可以先解开`druid-VERSION-bin.tar.gz`，然后去 那里`druid-<version>/extensions`解开`mysql-metadata-storage-bin.tar.gz`。现在只是指定`mysql-metadata-storage`，`druid.extensions.loadList`以便德鲁伊会接受它。有关详细信息，请参阅[包含扩展](http://druid.io/docs/0.12.3/operations/including-extensions.html)。