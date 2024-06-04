***

# 字段

对象创建完成后，接下来我们需要定义对象中的字段，在对象创建完成后，系统会自动生成部分系统字段。

![](https://tcs-devops.aliyuncs.com/storage/1134f440134958c6e57741e5598cf614eecb?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODA5MjgyNywiaWF0IjoxNzE3NDg4MDI3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzRmNDQwMTM0OTU4YzZlNTc3NDFlNTU5OGNmNjE0ZWVjYiJ9.x_I2x0Asn1Zq5U9Pseja11W7R1ndXEbd6m0jLbG50k4&download=image.png "")

点击右上角新建字段，进入新建字段表单，字段支持以下类型：

- 文本，支持设置默认值，最大长度；

- 文本域，支持设置默认值，最大长度；

- 整数，支持设置默认值；

- 实数，支持设置默认值，小数位数；

- 单选，需要先在自定义字典中定义好数值列表，例如性别，业务类型等字典，在该字段中关联此字典，并可设置默认值；

- 多选，同单选；

- 日期，YYYY-MM-DD格式，可设置是否为动态时间，设置为动态时间后，录入数据后会自动填入当前日期；

- 时间，YYYY-MM-DD HH:mm:ss格式，可设置动态时间；

- 货币，同实数；

- 关联关系，关联另一个对象，例如在销售订单上需要关联下单客户信息，那就可以在销售订单对象上建一个关联关系字段，选择客户对象。同时也可以利用筛选器，对关联的对象范围进行限定：

![](https://tcs-devops.aliyuncs.com/storage/1134dba058e09f0955e16ec8d81c1ee68fe6?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODA5MjgyNywiaWF0IjoxNzE3NDg4MDI3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzRkYmEwNThlMDlmMDk1NWUxNmVjOGQ4MWMxZWU2OGZlNiJ9.Tg1FQVHVL7TEj1u2K-lvGD1EER3V2oZlK-1Nc1fGTIU&download=image.png "")

筛选器（）是对关联对象的范围限定，举例，对于一个淘宝系订单，下单的时候，客户不能选到京东平台的客户，那就可以在筛选器中限定，客户所属平台=订单来源平台，这样，填写订单，选择客户时，就只能选到指定条件的客户范围。

- 主子明细：两个对象通过关联关系建立了一对一对应关系，但是对于订单和订单货品这两个对象，其是1：N的关系，那就需要建立主子明细字段。以订单和订单货品为例，需要在订单货品这个对象上建立一个主子明细对象，关联对象选择订单，同时也可以用筛选器进行范围限定。对于建立了主子明细关系的两个对象，系统会自动在主表的表单下创建一个tab，展示关联的明细表：

![](https://tcs-devops.aliyuncs.com/storage/1134a9c8fdd5b3a258a55c041db8dabb7cfc?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODA5MjgyNywiaWF0IjoxNzE3NDg4MDI3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzRhOWM4ZmRkNWIzYTI1OGE1NWMwNDFkYjhkYWJiN2NmYyJ9.A9t2xgwH-815rP9dcwo0_BXdVlcMZLf8LiFJvqXbS3k&download=image.png "")

- 电话，支持设置默认值，最大长度；

- 邮箱，支持设置默认值，最大长度；

- 网址，支持设置默认值，最大长度；

- 布尔值，支持设置默认值；

- 文件，支持设置最大数量，文件大小限制；

- 百分比；

- 引用字段，当通过关联关系或主子明细在两个对象建立了关联，就可以通过引用字段自动带出另一个对象的属性，例如，在订单上，我们想看到客户的联系方式，收货地址等，这些数据是维护在客户对象上的，那就可以在订单对象上建立引用字段，引用客户的联系方式字段，这样，在填写订单信息，选择完客户后，客户的联系方式会自动带出到订单里。

- 级联字段，需要关联一个自定义字典，以级联的方式选择和展示；

- 多选关联关系，关联关系的多选；

- 自动编号，同主属性字段的自动编号；



此外，在定义字段时，还可以设置字段的必填，是否可复制（如审批状态这种字段，在复制单据时，不希望把审批状态也复制过去，则可以把此选项勾掉），帮助文本（在前端页面上点击字段可看到此帮助文本）等属性。