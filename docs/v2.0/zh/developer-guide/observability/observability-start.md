# 可观测性使用

## 介绍

suiteboot内置了opentelemetry组件，并且提供了一键运行可观测性组件的方式



## 安装Maven依赖

```xml
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <suiteboot.version>2.0.0</suiteboot.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>com.suiteopen.boot</groupId>
            <artifactId>suiteopen-springboot3-starter</artifactId>
            <version>${suiteboot.version}</version>
        </dependency>
    </dependencies>
```



## SpringBoot配置



opentelemetry官方提供了[__对springboot的支持__](https://opentelemetry.io/docs/languages/java/automatic/spring-boot/)，以下配置示例仅仅只开启了logback、注解方式收集Trace数据的配置



注意：因为使用[__Collector Agent模式部署__](https://thoughts.aliyun.com/workspaces/64473d10725662001abf2a93/docs/661f7e0c5e119400017c8375#6620808095e6c8ae04fa3fdd)，所以这里exporter只配置colelctor的地址，所有的数据卸载到Collector即可，由Collector再进行分发

```yaml
otel:
  resource:
    attributes:
      service.name: wms
  instrumentation:
    logback-appender:
      enabled: true
    spring-web:
      enabled: false
    spring-webflux:
      enabled: false
    spring-webmvc:
      enabled: false
    annotations:
      enabled: true
  springboot:
    resource:
      enabled: false
  exporter:
    otlp:
      protocol: grpc
      endpoint: http://contrib.collector:4317
      metrics:
        headers:
          metricNamespace: wms
  logs:
    exporter: otlp
  metrics:
    exporter: otlp
  traces:
    exporter: otlp
```



## Java系统属性配置

部分组件可能默认集成了opentelemetry，但是并不读取上述配置（例如elastic search客户端库），可能会导致收集到大量无用的数据，所以需要额外配置一些配置项去抑制这些数据的收集

```bash
-Dotel.instrumentation.common.default-enabled=false
-Dotel.instrumentation.opentelemetry-api.enabled=true
-Dotel.java.global-autoconfigure.enabled=true
-Dotel.instrumentation.elasticsearch.enabled=false
```



## Logback配置

opentelemetry官方实现了日志数据的桥接，底层实现是opentelemetry扩展了Logback的Appender，通过Appender接受日志打印的事件，从而将数据发送到Collector，suiteboot框架实现了抑制数据收集的filter，[__参考文档__](https://thoughts.aliyun.com/workspaces/64473d10725662001abf2a93/docs/66207c6651244b000197389f)，在程序运行过程中，日志大部分数据可能无用，会造成程序成本的上升，为了抑制无效数据的收集，需要过滤无效类别的数据，所以以下配置示例配置了日志过滤器

```text
    <appender name="OTEL"
              class="io.opentelemetry.instrumentation.logback.appender.v1_0.OpenTelemetryAppender">
        <captureExperimentalAttributes>true</captureExperimentalAttributes>
        <captureCodeAttributes>true</captureCodeAttributes>
        <captureMarkerAttribute>true</captureMarkerAttribute>
        <captureKeyValuePairAttributes>true</captureKeyValuePairAttributes>
        <captureLoggerContext>true</captureLoggerContext>
        <captureMdcAttributes>*</captureMdcAttributes>

        <filter class="com.suiteopen.boot.common.log.OTELCollectorMarkerFilter" />
    </appender>

```



## 编写springboot启动器

[__参考suiteboot示例项目__](https://thoughts.aliyun.com/workspaces/64473d10725662001abf2a93/docs/65f3fc7151244b00017784ed)

```java
import com.suiteopen.boot.EnableSuiteOpenBoot;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@EnableSuiteOpenBoot
@SpringBootApplication
public class Application {
    public static void main(String[] args){
        SpringApplication.run(
                Application.class,
                args
        );
    }
}
```



## 遥测Trace注解使用

opentelemetry官方实现了遥测注解的方式收集数据，底层实现原理是spring AOP切面实现，提供了2个注解

1. WithSpan: 方法级别的注解，SpringAOP切面会自动创建1个Span对象

1. SpanAttribute：方法参数级别的注解，会自动将方法的参数作为span的1个resource信息

```java
public class WmsPackageController {
	private final Logger otelLogger = 
	GlobalOpenTelemetry.get().getLogsBridge().get(WmsPackageController.class.getName());

    @PostMapping("/done")  
	@WithSpan public AjaxResult done(  
	@RequestBody @SpanAttribute("PackDoneRequest") PackDoneRequest request) throws Throwable {
			 
        // 将Trace信息放到指定动态索引
		Span span = Span.current();  
		span.setAttribute("elasticsearch.index.suffix", "_http_snapshot "); 
	}
}

```

注意事项：由于spring AOP切面会有同1个类方法互相调用会导致无法触发切面逻辑，可能会导致部分的方法的生命span注解但是没有收集到span数据



## Logback API使用



在上述配置中，配置了Logback的filter，所以开发者在打印日志时需要评估当前日志是否需要收集到es中，只需要给logger api传递marker参数即可

```java
        logger.atInfo()
                .addMarker(Logs.OTEL_BIZ_MARKER)
                .setMessage("This is an error log.")
                .addKeyValue("Field", "CreateTime")
                .addArgument("This is a description.")
                .addArgument(new Document("MongoVersion", "7.X"))
                .log();
```



## suiteboot多事务可观测使用

1. 给目标方法添加WithTransaction注解

1. 声明observable为true即可

具体实现[__参考文档__](https://thoughts.aliyun.com/workspaces/64473d10725662001abf2a93/docs/66207c6651244b000197389f#66208e0a3782ba3ba2aeb6f2)

```java
    @WithSpan
    @WithTransaction(observable = true, concurrency = 7)
    @GetMapping("/otel3")
    public AjaxResult testCustomOTEL() {
        try (
                JsStatelessScriptCommands commands = new JsStatelessScriptCommands(
                        """
                                () => {
                                    var list = rclient.list( 'sales_order',  {recordId:93215},  {}, [])
                                    console.log(list);
                                    return list
                                }
                                """,
                        Map.of("rclient", proxyTxRecordClient))
                ){

            List<CustomRecord> response = commands.execute(Utils.RECORD_ARRAY_TYPE_LITERAL);
            return AjaxResult.success(response);

        } catch (Exception e) {
            throw new RuntimeException(e);}
    }

```



## 依赖的可观测组件

| 工具            | 版本      | 用途                   |
| ------------- | ------- | -------------------- |
| Elasticsearch | 8.0     | 存储Trace、Log数据        |
| Prometheus    | 2.50.1  | 存储指标数据               |
| Jaeger        | v1.55.0 | 可视化Trace数据工具         |
| Grafana       | 10+     | 自定义指标Dashboard       |
| Collector     | 0.96    | 接受Trace、Log、Metric数据 |



## 可观测组件安装

suiteboot提供了一键安装上述组件的方式，采用docker-compose快速搭建一套可观测的环境

**下载docker-compose.yaml文件**

```shell
git clone -b master git@codeup.aliyun.com:suiteopen/suiteboot/suiteopen-boot.git
cd suiteopen-boot/deploy/observable
```

**安装**

```shell
docker-compose up -d
```

![](https://tcs-devops.aliyuncs.com/storage/113516c36746946bb154dddff1ec39bd3507?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODA5NDU3NiwiaWF0IjoxNzE3NDg5Nzc2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzUxNmMzNjc0Njk0NmJiMTU0ZGRkZmYxZWMzOWJkMzUwNyJ9.2NbDSzF-LEtlqvvooqIaLVb5mp48RPM3i5XVWsOvjvk&download=image.png "")



## Trace数据展示格式

![](https://tcs-devops.aliyuncs.com/storage/113573fb198ed5bc6641280c83b199acbe95?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODA5NDU3NiwiaWF0IjoxNzE3NDg5Nzc2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzU3M2ZiMTk4ZWQ1YmM2NjQxMjgwYzgzYjE5OWFjYmU5NSJ9.Y5eNeNrlCZ5O_C6vQJI9XBoKPkyGuDbo7wm0yf597ck&download=image.png "")

## Log数据展示格式

![](https://tcs-devops.aliyuncs.com/storage/1135cb2b688a3f5b1c605b63c031ef9492bd?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODA5NDU3NiwiaWF0IjoxNzE3NDg5Nzc2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzVjYjJiNjg4YTNmNWIxYzYwNWI2M2MwMzFlZjk0OTJiZCJ9.CPpVLqVlfBXM1F7eScAWltR4dgAozrzwRWl5RIyPD5w&download=image.png "")

## Metric数据展示格式

![](https://tcs-devops.aliyuncs.com/storage/1135a6649b77231dd359c2e0c724f4449b9a?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODA5NDU3NiwiaWF0IjoxNzE3NDg5Nzc2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzVhNjY0OWI3NzIzMWRkMzU5YzJlMGM3MjRmNDQ0OWI5YSJ9.Jj958kBRlCdnKFlN7hGTeGOvUw04RRJPMJMIo9dutJw&download=image.png "")

