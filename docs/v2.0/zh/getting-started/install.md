# 一键部署



## SuiteBoot依赖基础组件



| 组件              | 版本    |
| --------------- | ----- |
| MongoDB         | 7.0   |
| MySQL           | 8.3   |
| Redis           | 7.0+  |
| ElasticSearch   | 8.0+  |
| Graalvm OpenJDK | 17    |
| SpringBoot      | 3.2.3 |



## SuiteBoot Example项目

```text
https://github.com/SuiteOpenTeam/suiteboot-example
```


## DockerCompose部署



### 先决条件

1. [__安装Docker__](https://docs.docker.com/engine/install/)

1. [__安装Docker-Compose__](https://docs.docker.com/compose/install/)



### 快速部署步骤

1. 下载示例代码

```shell
git clone git@github.com:SuiteOpenTeam/suiteboot-example.git
```

1. 切换目录到deploy

```shell
cd suiteboot-example/deploy
```

1. 执行部署命令

```shell
docker-compose up -d
```



### 访问应用

```text
http://localhost:3000
```

### 


