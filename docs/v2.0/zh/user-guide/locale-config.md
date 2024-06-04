# 多语言
## 设置语言环境

![](https://tcs-devops.aliyuncs.com/storage/113477bcf76f60246cfecb10334d7edf14cb?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODA5NTgyNiwiaWF0IjoxNzE3NDkxMDI2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzQ3N2JjZjc2ZjYwMjQ2Y2ZlY2IxMDMzNGQ3ZWRmMTRjYiJ9.x_j2iu6p6T18ugNWhiikYPQcRTVmZPfIu3S06u1_kEw&download=image.png "")

鼠标悬浮于系统右上角姓名头像，选择语言设置，切换语言环境。



## 多语言配置

菜单路径：系统设置-本地化设置

录入或导入文本、翻译等信息完成多语言配置。

在代码中使用：

```java
LocaleHelper.translateText("非法请求: 缺少认证信息")
```

系统会根据每个用户设置的语言环境，自动将该文本转换为对应语言的翻译文本。