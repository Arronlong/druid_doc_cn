# 记录

Druid节点将发出对调试控制台有用的日志。德鲁伊节点还会发出关于其状态的周期性指标。有关指标的更多信息，请参阅[配置](http://druid.io/docs/0.12.3/configuration/index.html#enabling-metrics)。度量标准日志默认打印到控制台，可以使用`-Ddruid.emitter.logging.logLevel=debug`。

Druid使用[log4j2](http://logging.apache.org/log4j/2.x/)进行日志记录。可以使用log4j2.xml文件配置日志记录。如果要覆盖默认的德鲁伊日志配置，请将包含log4j2.xml文件的目录（例如_common / dir）添加到类路径中。请注意，此目录应该在类路径中比druid jar更早。最简单的方法是使用config dir为classpath添加前缀。

要使java日志记录通过log4j2，请设置`-Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager`server参数。

示例log4j2.xml在config / _common / log4j2.xml下附带了Druid，示例文件也如下所示：

```text
<?xml version="1.0" encoding="UTF-8" ?>
<Configuration status="WARN">
  <Appenders>
    <Console name="Console" target="SYSTEM_OUT">
      <PatternLayout pattern="%d{ISO8601} %p [%t] %c - %m%n"/>
    </Console>
  </Appenders>
  <Loggers>
    <Root level="info">
      <AppenderRef ref="Console"/>
    </Root>

    <!-- Uncomment to enable logging of all HTTP requests
    <Logger name="io.druid.jetty.RequestLog" additivity="false" level="DEBUG">
        <AppenderRef ref="Console"/>
    </Logger>
    -->
  </Loggers>
</Configuration>
```