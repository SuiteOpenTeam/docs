***

在SuiteBoot中，当菜单关联了某个对象时（详见），点击该菜单，会展示该对象数据表格，该表格由视图控制。

# 视图

***

在对象的视图Tab下新建视图，对象创建后，系统会创建一个默认的视图。

![](https://tcs-devops.aliyuncs.com/storage/1134cf4653465f70490298df900ad9202d67?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODA5Mjg5NCwiaWF0IjoxNzE3NDg4MDk0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzRjZjQ2NTM0NjVmNzA0OTAyOThkZjkwMGFkOTIwMmQ2NyJ9.NEINWNWKTbLqQOLj__L8IcSGqqB6hSBapaGYzYg4-NU&download=image.png "")

## 视图字段

在第一个页签 基本信息 下编辑视图的基本信息。

在字段一栏中，左侧列出可在视图中展示的所有字段，包括本对象的字段以及子对象（主子明细关系）的字段，例如我们可以把订单明细的货品，数量，价格等信息和订单信息一块在视图中展示。

在右侧拖拽可调整字段显示顺序。

在排序一栏中，可以定义视图按照什么哪个字段排序，支持多字段组合排序。

视图也可以绑定某个表单，当视图绑定了表单后，在该视图页面下点击新增，会直接进入该表单的对象数据新建页面，而无需再选择表单。

## 筛选器

在筛选页签下，可以定义筛选条件以控制哪些对象数据在视图中展示。支持and和or的组合筛选条件。（）

# 视图的展示

***

当菜单关联了某个对象时，点击该菜单，会展示该对象数据表格，如下图所示：

![](https://tcs-devops.aliyuncs.com/storage/1134fb63c8e9711f0d1230e2dad327fa394e?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODA5Mjg5NCwiaWF0IjoxNzE3NDg4MDk0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzRmYjYzYzhlOTcxMWYwZDEyMzBlMmRhZDMyN2ZhMzk0ZSJ9.L4Wk6rVslQ5DOS80k9Nns64H9L6sOiLv-3eDPAHt36c&download=image.png "")

表格上方的可操作区包括以下内容：

- 刷新

- 筛选：视图可以通过筛选器控制该表格的数据范围，在此基础上，可以进一步筛选，获取精准数据。

![](https://tcs-devops.aliyuncs.com/storage/1134afc69973c0d0f05b640e5bde19a6be01?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODA5Mjg5NCwiaWF0IjoxNzE3NDg4MDk0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzRhZmM2OTk3M2MwZDBmMDViNjQwZTViZGUxOWE2YmUwMSJ9._5XGyKzhk7-XLCyro3-DrshtcA_05nJ_M4RJ4i4DbmE&download=image.png "")

- 切换视图，可以在不同视图切换来调整表格显示内容

- 批量操作，包括导入、导出该视图数据以及自定义批量操作按钮。

- 新建，新建一条对象数据，如果在视图设置中下绑定了表单，则会直接进入该表单页面，否则需要先选择在哪个表单下创建。

