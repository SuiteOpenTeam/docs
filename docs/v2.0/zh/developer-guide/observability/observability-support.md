# 可观测性支持

## 什么是可观测性

可观察性是通过检查系统的输出来了解系统内部状态的能力。在软件环境中，这意味着能够通过检查系统的遥测数据（包括跟踪、指标和日志）来了解系统的内部状态。 为了使系统可观察，必须对其进行 检测。也就是说，代码必须发出 跟踪、 指标或 日志。然后，必须将检测数据发送到可观测性后端。所以suiteboot内置了对opentelemetry的支持，  并在基础框架中实现了扩展了opentelemetry的SPI接口



## 集成遥测框架的原因

1. 程序可用率无法达到100%可用

    1. 控制流恢复相对简单，只需要重启java程序，

    2. 数据流恢复根据业务复杂度决定，例如恢复DB数据、矫正第三方数据（例如快递，货发出去了，钱扣了，系统因为某种不确定原因找不到对应的发货记录）。



## 为什么选择Opentelemetry

opentelemetry已经逐渐成为了业界的标准，由CNCF开发，社区活跃度高，支持多语言

| 遥测框架          | 优势  | 缺点  |
| ------------- | --- | --- |
| Skywalking    |     |     |
| Opentelemetry |     |     |



## Opentelemetry介绍

OpenTelemetry 是一个 可观察性 框架和工具包，旨在创建和管理遥测数据，例如 跟踪、 指标和 日志。
至关重要的是，OpenTelemetry 与供应商和工具无关，这意味着它可以与各种可观察性后端一起使用，
包括 Jaeger和 Prometheus等开源工具以及商业产品。 OpenTelemetry 不是像 Jaeger、Prometheus 或其他商业供应商那样的可观察性后端。 
OpenTelemetry 专注于遥测数据的生成、收集、管理和导出。 OpenTelemetry 的一个主要目标是您可以轻松地检测您的应用程序或系统，
无论其语言、基础设施或运行时环境如何。至关重要的是，遥测数据的存储和可视化有意留给其他工具。



## Collector部署架构

opentelemetry提供了[__三种collector部署模式__](https://opentelemetry.io/docs/collector/deployment/)

1. No Collector：数据可以直接发送到Jeager、ES、Prometheus等服务，没有中间环节，适用于小项目

1. Agent：代理接收器模式，程序将数据先快速卸载给Agent Collector，Agent Collector再将数据发送给Jeager、ES、Prometheus等服务

1. Gateway：网关接口器模式，程序将数据先发送给网关，网关再将数据发送给Collector集群，适用于大型项目



suiteboot框架选择了Agent部署模式，如果服务在k8s中部署，可以将colelctor作为pod的一个init container，如果在物理服务器上部署，单独部署1个Collector服务即可，Collector将数据发送给ES、Jeager、Prometheus

![](https://tcs-devops.aliyuncs.com/storage/11350399728944941232ddc06d7fe9e22838?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODA5NDU3NywiaWF0IjoxNzE3NDg5Nzc3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzUwMzk5NzI4OTQ0OTQxMjMyZGRjMDZkN2ZlOWUyMjgzOCJ9.0fn3NKeAZ6s5AMhpxLvJkmzZTi3WwV6hmfC-zygsOlc&download=image.png "")



## Opentelemetry探针类别

1. Metric - 存在问题？

2. Trace - 问题在哪里？

3. Log - 出了什么问题？

以上三者存在关系，找到错误日志后，可以根据日志中的traceId，抓到全链路的日志，suiteboot框架支持了三种探针类别



## 推荐的可观测工具栈

| 工具            | 用途             |
| ------------- | -------------- |
| Elasticsearch | 存储Trace、Log数据  |
| Prometheus    | 存储指标数据         |
| Jaeger        | 可视化Trace数据工具   |
| Grafana       | 自定义指标Dashboard |





## JVM指标支持



OpenTelemetry官方提供了JVM运行时[__JVM性能指标数据__](https://opentelemetry.io/docs/specs/semconv/runtime/jvm-metrics/)的导出，由于opentelemetry自动配置模块不会配置resource的实例ID的resource信息，这就会导致应用程序上报的部分机器性能指标也会缺少instanceId的label信息，suiteboot基础组件扩展了opentelemetry的SPI接口解决了这个问题，[__参考文档__](https://thoughts.aliyun.com/workspaces/64473d10725662001abf2a93/docs/66207c6651244b000197389f)；

suiteboot框架默认集成了JVM运行时监控maven的兼容版本依赖。



## RestTemplate客户端指标支持



## Apache HttpClient客户端指标支持

