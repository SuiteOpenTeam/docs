# 可观测性扩展

## ResourceProvider SPI扩展



### 为什么要扩展ResourceProvider？

opentelemetry自动配置模块不会配置resource的实例ID的resource信息，这就会导致应用程序上报的部分机器性能指标也会缺少instanceId的label信息，例如多个Java应用程序同时上报jvm内存使用率的指标，如果缺少instanceid的label，也就无法判断当前指标属于哪台机器。



### 实现

suiteboot默认扩展resource provider

1. 添加了[__service.instance.id__](https://opentelemetry.io/docs/specs/semconv/attributes-registry/service/)的resource信息，以当前机器的Hostname作为instanceId；

1. 添加了[__service.name__](https://opentelemetry.io/docs/specs/semconv/resource/#service)的resource信息，优先从[__otel.resource.attributes.service.name__](https://opentelemetry.io/docs/languages/java/automatic/spring-boot/)配置中获取，如果获取不到，会从[__spring.application.name__](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties.core.spring.application.name)配置项中获取；如果上述2个配置项都获取不到，则会默认配置成UnknownSvc；

1. 添加[__elasticsearch.index.prefix__](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter/elasticsearchexporter)的resource信息，此配置是collector向es中同步数据时的索引前缀

具体实现参考 com.suiteopen.boot.common.spi.otel.SvcDescResource



## Logback Filter扩展



### 为什么要扩展Logback Filter？

在程序运行过程中，日志大部分数据可能无用，会造成程序成本的上升，为了抑制无效数据的收集，需要过滤无效类别的数据



### 实现

suiteboot提供了两种日志类别

1. biz 类别： 只有当开发者声明当前打印的日志属于biz类，则默认会收集

1. alarm类别： 只有当开发者声明当前打印的日志属于alarm类，则默认会收集，并且会触发告警



### 使用方式

```java
private Logger logger = LoggerFactory.getLogger(TestController.class);
 

logger.atInfo()
                .addMarker(Logs.OTEL_BIZ_MARKER)
                .setMessage("This is an error log.")
                .addKeyValue("Field", "CreateTime")
                .addArgument("This is a description.")
                .addArgument(new Document("MongoVersion", "7.X"))
                .log();
```



## suiteboot多事务可观测

suiteboot内置了自定义了多事务的Trace和Metric数据，

主要提供两种可观测的数据（Metric&Trace），主要观测以下两种对象的运行情况

1. TransactionContext，一个TransactionContext会包含多个TransactionCommand

1. TransactionCommand，一个TransactionCommand对应1个具体的mongodb crud操作

[__具体细节参考文档__](https://thoughts.aliyun.com/workspaces/64473d10725662001abf2a93/docs/6621d0464c6050000151744f)