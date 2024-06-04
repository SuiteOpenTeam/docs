# 自定义OTEL Collector
OpenTelemetry Collector提供了2个发行版本

1. opentelemetry-collector发行版本，定义了collector实现规范，提供了otlp收集数据、处理数据、导出数据的功能，但功能单一，不能直接集成Prometheus、ElasticSearch等开源组件；

1. opentelemetry-collecor-contrib发行版本，由开源社区贡献，对业界常见的组件做了集成，可直接集成Prometheus、ElasticSearch等开源组件，也可以直接集成阿里云日志服务、亚马逊日志服务等商业组件。

## 为什么要自定义Collector组件？

1. 官方提供的contrib collector组件过于臃肿；

1. contrib collector默认支持了所有组件，并不是需要全部的组件，这也就意味着collector组件在安全层面增加了攻击面；

1. 开源社区提供的contrib组件不满足业务需求，需求根据业务需要改动部分collector代码以适配业务场景

    1. 开源社区提供的Exporter ElasticSearch组件向ES同步数据不能按照时间自动拆分索引，在数据量大的情况下不拆分索引会降低查询效率，例如需要ES索引按照月拆分，2024年3月份的日志数据存储到 'log-2024-03 ' 索引中，4月份的日志数据存储到 'log-2024-04' 索引中

## 自定义Collector组件解决方案

opentelemetry官方提供了Opentelemetry Collector Builder组件，具体细节参考文档 [https://opentelemetry.io/docs/collector/custom-collector/](https://opentelemetry.io/docs/collector/custom-collector/)

### 构建自定义Collector

先决条件

1. 安装go 1.21版本

1. Collector版本严格设定为0.96.0

1. 因为这里需要改动社区贡献的collector contrib代码，所以需要先下载contrib代码到/go目录下

1. 设置/go为GOPATH环境变量，构建时go需要读取GOPATH下的elasticsearchexporter代码

修改elasticsearchexporter模块代码

修改elasticsearchexporter代码的目的是使的elasticsearchexporter模块支持es索引拆分

1. 增加elasticsearchexporter/config.go文件DynamicIndexSetting的成员变量partitionIndexSuffixDateFormat，配置格式是日期格式 例如2006-01

```go

type DynamicIndexSetting struct {  
    Enabled                        bool   `mapstructure:"enabled"`  
    PartitionIndexSuffixDateFormat string `mapstructure:"partitionIndexSuffixDateFormat"` // 2006-01 
}

```

1. 增加elasticsearchexporter/logs_exporter.go文件elasticsearchLogsExporter的成员变量partitionIndexSuffixDateFormat，此值从DynamicIndexSetting传入

```go

type elasticsearchLogsExporter struct {  
    logger *zap.Logger  

    index                          string  
    logstashFormat                 LogstashFormatSettings  
    dynamicIndex                   bool  
    partitionIndexSuffixDateFormat string  
    maxAttempts                    int  

    client      *esClientCurrent  
    bulkIndexer esBulkIndexerCurrent  
    model       mappingModel  
}


func newLogsExporter(logger *zap.Logger, cfg *Config) (*elasticsearchLogsExporter, error) {  
    if err := cfg.Validate(); err != nil {  
       return nil, err  
    }  

    client, err := newElasticsearchClient(logger, cfg)  
    if err != nil {  
       return nil, err  
    }  

    bulkIndexer, err := newBulkIndexer(logger, client, cfg)  
    if err != nil {  
       return nil, err  
    }  

    maxAttempts := 1  
    if cfg.Retry.Enabled {  
       maxAttempts = cfg.Retry.MaxRequests  
    }  

    model := &encodeModel{  
       dedup: cfg.Mapping.Dedup,  
       dedot: cfg.Mapping.Dedot,  
       mode:  cfg.MappingMode(),  
    }  

    indexStr := cfg.LogsIndex  
    if cfg.Index != "" {  
       indexStr = cfg.Index  
    }  
    esLogsExp := &elasticsearchLogsExporter{  
       logger:      logger,  
       client:      client,  
       bulkIndexer: bulkIndexer,  

       index:                          indexStr,  
       dynamicIndex:                   cfg.LogsDynamicIndex.Enabled,  
       partitionIndexSuffixDateFormat: cfg.LogsDynamicIndex.PartitionIndexSuffixDateFormat,  

       maxAttempts:    maxAttempts,  
       model:          model,  
       logstashFormat: cfg.LogstashFormat,  
    }  
    return esLogsExp, nil  
}

```

1. 增加elasticsearchexporter/traces_exporter.go文件elasticsearchTracesExporter的成员变量partitionIndexSuffixDateFormat，此值从DynamicIndexSetting传入

```go

type elasticsearchTracesExporter struct {  
    logger *zap.Logger  

    index                          string  
    logstashFormat                 LogstashFormatSettings  
    dynamicIndex                   bool  
    partitionIndexSuffixDateFormat string  

    maxAttempts int  

    client      *esClientCurrent  
    bulkIndexer esBulkIndexerCurrent  
    model       mappingModel  
}

func newTracesExporter(logger *zap.Logger, cfg *Config) (*elasticsearchTracesExporter, error) {  
    if err := cfg.Validate(); err != nil {  
       return nil, err  
    }  

    client, err := newElasticsearchClient(logger, cfg)  
    if err != nil {  
       return nil, err  
    }  

    bulkIndexer, err := newBulkIndexer(logger, client, cfg)  
    if err != nil {  
       return nil, err  
    }  

    maxAttempts := 1  
    if cfg.Retry.Enabled {  
       maxAttempts = cfg.Retry.MaxRequests  
    }  

    model := &encodeModel{  
       dedup: cfg.Mapping.Dedup,  
       dedot: cfg.Mapping.Dedot,  
       mode:  cfg.MappingMode(),  
    }  

    return &elasticsearchTracesExporter{  
       logger:      logger,  
       client:      client,  
       bulkIndexer: bulkIndexer,  

       index:                          cfg.TracesIndex,  
       dynamicIndex:                   cfg.TracesDynamicIndex.Enabled,  
       partitionIndexSuffixDateFormat: cfg.TracesDynamicIndex.PartitionIndexSuffixDateFormat,  

       maxAttempts:    maxAttempts,  
       model:          model,  
       logstashFormat: cfg.LogstashFormat,  
    }, nil  
}

```

1. 修改elasticsearchexporter/logs_exporter.go文件elasticsearchTracesExporter struct的方法pushLogRecord，增加索引名称拼接逻辑

```go

func (e *elasticsearchLogsExporter) pushLogRecord(ctx context.Context, resource pcommon.Resource, record plog.LogRecord, scope pcommon.InstrumentationScope) error {  
    fIndex := e.index  
    if e.dynamicIndex {  
       prefix := getFromBothResourceAndAttribute(indexPrefix, resource, record)  
       suffix := getFromBothResourceAndAttribute(indexSuffix, resource, record)  

       fIndex = fmt.Sprintf("%s%s%s", prefix, fIndex, suffix)  

       partSuffix := time.Now().Format(e.partitionIndexSuffixDateFormat)  
       if partSuffix == "" {  
          partSuffix = time.Now().Format("2006-01")  
       }  
       fIndex = fmt.Sprintf("%s-%s", fIndex, partSuffix)  
    }  

    if e.logstashFormat.Enabled {  
       formattedIndex, err := generateIndexWithLogstashFormat(fIndex, &e.logstashFormat, time.Now())  
       if err != nil {  
          return err  
       }  
       fIndex = formattedIndex  
    }  

    document, err := e.model.encodeLog(resource, record, scope)  
    if err != nil {  
       return fmt.Errorf("Failed to encode log event: %w", err)  
    }  
    return pushDocuments(ctx, e.logger, fIndex, document, e.bulkIndexer, e.maxAttempts)  
}

```

1. 修改elasticsearchexporter/traces_exporter.go文件elasticsearchTracesExporter struct的方法pushLogRecord，增加索引名称拼接逻辑

```go

func (e *elasticsearchTracesExporter) pushTraceRecord(ctx context.Context, resource pcommon.Resource, span ptrace.Span, scope pcommon.InstrumentationScope) error {  
    fIndex := e.index  
    if e.dynamicIndex {  
       prefix := getFromBothResourceAndAttribute(indexPrefix, resource, span)  
       suffix := getFromBothResourceAndAttribute(indexSuffix, resource, span)  

       fIndex = fmt.Sprintf("%s%s%s", prefix, fIndex, suffix)  

       partSuffix := time.Now().Format(e.partitionIndexSuffixDateFormat)  
       if partSuffix == "" {  
          partSuffix = time.Now().Format("2006-01")  
       }  
       fIndex = fmt.Sprintf("%s-%s", fIndex, partSuffix)  
    }  

    if e.logstashFormat.Enabled {  
       formattedIndex, err := generateIndexWithLogstashFormat(fIndex, &e.logstashFormat, time.Now())  
       if err != nil {  
          return err  
       }  
       fIndex = formattedIndex  
    }  

    document, err := e.model.encodeSpan(resource, span, scope)  
    if err != nil {  
       return fmt.Errorf("Failed to encode trace record: %w", err)  
    }  
    return pushDocuments(ctx, e.logger, fIndex, document, e.bulkIndexer, e.maxAttempts)  
}

```

下载Builder

builder安装有2种方式

1. 下载builder的可执行文件

1. 因builder使用go编写，所以可以使用go install来安装可执行文件

这里使用builder可执行文件安装

```shell

curl --proto '=https' --tlsv1.2 -fL -o ocb \
https://github.com/open-telemetry/opentelemetry-collector/releases/download/cmd%2Fbuilder%2Fv0.96.0/ocb_0.96.0_linux_amd64
chmod +x ocb


```

编写清单文件(manifest)

```yml

dist:  
  module: go.opentelemetry.io/collector/cmd/otelcorecol  # 定义构建项目模块的坐标
  name: collector     # 输出可执行collector文件名称
  description: Custom OpenTelemetry Collector binary.  
  version: v0.96.0-a    # 自定义版本
  output_path: /go/ocb_builder    # 二进制文件输出目录

receivers:  
  - gomod: go.opentelemetry.io/collector/receiver/otlpreceiver v0.96.0   # otlp receiver组件的go mod依赖
exporters:  
  - gomod: "go.opentelemetry.io/collector/exporter/loggingexporter v0.96.0"  # logging组件的go mod依赖
  - gomod: "github.com/open-telemetry/opentelemetry-collector-contrib/exporter/prometheusremotewriteexporter v0.96.0"  # prometheus远程写指标组件的go mod依赖
  - gomod: "go.opentelemetry.io/collector/exporter/otlpexporter v0.96.0"    # otlp exporter组件的go mod依赖
  - gomod: "github.com/open-telemetry/opentelemetry-collector-contrib/exporter/elasticsearchexporter v0.96.1"  # es exporter组件的go mod依赖
    import: "github.com/open-telemetry/opentelemetry-collector-contrib/exporter/elasticsearchexporter"  # go 文件中需要倒入的包依赖 对应import的内容
    name: "elasticsearchexporter"    # 模块别名 类似于 import ("elasticsearchexporter"  "github.com/open-telemetry/opentelemetry-collector-contrib/exporter/elasticsearchexporter")
    path: "/go/opentelemetry-collector-contrib/exporter/elasticsearchexporter" # 替换为本地的go es模块代码

```

构建

```shell

./ocb --config builder-config.yaml

```

### 镜像制作

这里尽量避免制作1个二进制的collector可执行文件，因为go并不跨平台，ubuntu amd64的架构上构建的二进制并不一定能在centos amd64的架构上运行，所以需要构建1个容器



**先决条件：**

1. 复制上述的manifest（custom_collector_define.yaml）到当前构建目录

1. 复制修改好的contrib代码到当前目录

1. 定义otel collector配置



**Collector配置定义：**

上述的manifest文件定义了otlp receiver、otlp exporter、es、prometheus、logging组件，所以构建出来的collector可支持以下组件

1. otlp receiver

1. prometheus

1. jeager trace（collector在0.85版本之后移除了jaeger的exporter，最近版本的collector otlp默认支持可以导出数据到jaeger）

1. elasticsearch

    1. 可在elasticsearch/log配置中增加partitionIndexSuffixDateFormat配置，这个属性是本文新增的

```yml

receivers:  
  otlp:  
    protocols:  
      grpc:  
        endpoint: 0.0.0.0:4317  
      http:  
exporters:  
  logging:  
    loglevel: debug  
  otlp/jaeger:  
    endpoint: jaeger:4317  
#    endpoint: 172.19.0.3:4317  
    tls:  
      insecure: true  
  elasticsearch/log:  
    endpoints: ["http://elasticsearch:9200"]  
#    endpoints: ["http://172.17.0.2:9200"]  
    logs_index: log-indice  
    logs_dynamic_index:  
      enabled: true  
      partitionIndexSuffixDateFormat: 2006-01  
  prometheusremotewrite:  
    endpoint: http://prometheus:9090/api/v1/write  
#    endpoint: http://172.19.0.5:9090/api/v1/write  
service:  
  pipelines:  
    traces:  
      receivers: [otlp]  
      exporters: [otlp/jaeger]  
    logs:  
      receivers: [otlp]  
      exporters: [elasticsearch/log]  
    metrics:  
      receivers: [otlp]  
      exporters: [prometheusremotewrite]

```

**Collector容器定义**

```Dockerfile

FROM golang:1.21 as build  
ARG  OTEL_VERSION=0.96.0  
WORKDIR /go  
ENV GOPATH /go  
COPY . .  
RUN go env -w GOPROXY=https://goproxy.cn,direct  
RUN go install go.opentelemetry.io/collector/cmd/builder@v${OTEL_VERSION}  
RUN CGO_ENABLED=0 builder --config=custom_collector_define.yaml  

FROM gcr.io/distroless/base-debian11  
COPY --from=build /go/ocb_builder/collector /  
COPY ./otelcol-contrib-config.yaml /etc/otelcol-contrib-config.yaml  
EXPOSE 4317/tcp  
CMD ["--config", "/etc/otelcol-contrib-config.yaml"]  
ENTRYPOINT ["/collector"]

```

本文定义的容器已经被推到docker registry了 ，容器下载使用下面的指令

```shell

docker push voyagerma/otel-collector:0.96.0

```

### 附录

1. 本文修改contrib代码git仓库 [__https://github.com/cloudtechstack/opentelemetry-collector-contrib__](https://github.com/cloudtechstack/opentelemetry-collector-contrib)

