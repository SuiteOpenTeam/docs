# 配置管理
SuiteBoot内置了简单的配置中心功能（通过Redis Pub/Sub实现），开发者也可以根据自己的实际需求切换为Nacos、Appllo等其他配置中心。

菜单路径：系统设置-配置管理

![](https://tcs-devops.aliyuncs.com/storage/11340e7fc896f46702f5dd310b1aafb766dd?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODA5NDUwOCwiaWF0IjoxNzE3NDg5NzA4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzQwZTdmYzg5NmY0NjcwMmY1ZGQzMTBiMWFhZmI3NjZkZCJ9.Znj6xwXdQqVMhvSUIlZ1Lp-ttpu6ehGvcYOkD8LmVC8&download=image.png "")

点击右上角新增配置按钮添加一条新的配置。

填写配置名称、类型、配置键、配置值和配置说明。

新增或更新配置后，系统或自动刷新最新的配置项，无需重启服务。

在代码中获取配置：

```java
// 通过配置类型获取该类型下所有的配置键值对
Map<String, String> configMap = SystemConfigHelper.getConfigsByConfigType("config_type");

// 获取某个类型的某个配置项的值
String configValue = SystemConfigHelper.getConfigValue("config_type", "config_key");
```

