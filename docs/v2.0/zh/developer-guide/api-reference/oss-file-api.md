# OSS File API

## 内置OSS File API实现



suiteboot基础框架内置了一部分OSS File API实现，抽象了FileService，内置实现如下

1. 阿里云OSS API

1. 腾讯云OSS API

1. Oracle OSS API

可以通过spring配置来控制使用不同的API，不一样的在一个spring应用里面只能有一种OSS API 被开启



## OSS API配置示例



### Oracle OSS配置demo

```yaml
oss:
  oci:
    enabled: true
    user: xxxxxxx
    fingerprint: xxxxx
    tenancy: xxxxx
    region: us-phoenix-1
    namespace: default
    bucketName: xxxx
    privateKey: 
```



### 阿里云OSS配置demo

```yaml
oss:
  aliyun:
    enabled: false
    endpoint:
    accessKey:
    secretKey:
    bucketName:
    sts:
      regionId:
      endpoint:
      accessKey:
      secretKey:
      arn:
```



### 腾讯云OSS配置demo

```yaml
oss:
  qcloud:
    enabled: true
    region: ap-hongkong
    secretId: xx
    secretKey: xx
    bucketName: xx-xx
    sts:
      durationSeconds: 7200
      allowActions: [ "name/cos:PutObject","name/cos:PostObject","name/cos:GetObject" ]

```



## 扩展

开发可以通过实现com.suiteopen.boot.file.service.FileService接口来自定义扩展FileService逻辑，并且将FileService的实现类作为1个spring bean注入到spring容器中即可，suiteboot框架自动会使用开发者扩展的FileService作为文件API。

