# 应用配置

## 先决条件



确保本地安装以下软件

1. OpenJDK17

1. Maven3.9.2



## Maven依赖



suiteboot提供了springboot3.0的starter

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



## 配置application.yml



### 配置OpenAPi Doc

```yaml
spring:
  springdoc:
    swagger-ui:
      path: /swagger-ui.html
      tags-sorter: alpha
      #operations-sorter: order
    api-docs:
      path: /v3/api-docs
    group-configs:
      - group: 'default'
        display-name: '测试'
        paths-to-match: '/**'
        packages-to-scan: com.suiteopen
    default-flat-param-object: true
```



### 配置Undertow

```yaml
server:
  port: 8080
  undertow:
    max-http-post-size: -1
    # 以下的配置会影响buffer,这些buffer会用于服务器连接的IO操作,有点类似netty的池化内存管理
    # 每块buffer的空间大小,越小的空间被利用越充分
    buffer-size: 512
    threads:
      # 设置IO线程数, 它主要执行非阻塞的任务,它们会负责多个连接, 默认设置每个CPU核心一个线程
      io: 8
      # 阻塞任务线程池, 当执行类似servlet请求阻塞操作, undertow会从这个线程池中取得线程,它的值设置取决于系统的负载
      worker: 256
    # 是否分配的直接内存
    direct-buffers: true
  error:
    include-exception: true
    include-stacktrace: ALWAYS
    include-message: ALWAYS
  servlet:
    context-path: /
  compression:
    enabled: true
    min-response-size: 1024
    mime-types: application/javascript,application/json,application/xml,text/html,text/xml,text/plain,text/css,image/*


```



### 配置redisson

```yaml
spring:
  redis:
    redisson:
      config: |
        singleServerConfig:
          address: "redis://localhost:6379"
          database: 10
          connectionMinimumIdleSize: 12
        threads: 16
        nettyThreads: 32
        transportMode: "NIO"
```



### 配置JDBC

```yaml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driverClassName: com.mysql.cj.jdbc.Driver
    druid:
      url: jdbc:mysql://l/suiteboot_wms?ocalhost:3306useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8&autoReconnect=true
      username: root
      password: root
```



### 配置MongoDB

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://localhost:27017,localhost:27018,localhost:27019/suiteboot?replicaSet=myReplicaSet
```



### 配置ElasticSearch

```yaml
spring:
  elasticsearch:
    rest:
      uris: http://localhost:9200
```



### 配置Opentelemetry

```yaml
otel:
  resource:
    attributes:
      service.name: example-app
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
  logs:
    exporter: logging
  metrics:
    exporter: logging
  traces:
    exporter: logging
```



## 新建Spring启动类

```java
package com.suiteboot.example;

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



## 运行启动类

