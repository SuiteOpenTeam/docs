***
# 管理自定义脚本
通过自定义对象、字段、按钮等，我们拥有了一套可定制化的表单增删改查功能的系统，对于一些简单场景，这些完全可以应付。

但对于复杂的业务系统，这些可能还不够，复杂业务系统的业务逻辑流转、数据传递对低代码平台的扩展性提出了更高的要求。

在SuiteBoot中，我们通过自定义脚本来实现。

- 客户端脚本，ClientScript，可以控制表单样式，执行校验，预加载等；

- 对象事件脚本，RecordEvent，通过加载、提交前、提交后这三个切入点来控制对象数据的业务流转；

- 开放API脚本，RestletScript，定义一个HTTP GET/POST接口，开放给其他系统调用；

- 定时任务脚本，ScheduleScript，定时任务脚本

- 按钮事件脚本，ButtonEvent，控制自定义按钮点击后的业务逻辑；

- 流程事件脚本，FLowEvent，在流程节点上控制流程运转过程中的逻辑处理；

- 工具类脚本，UtilityScript，定义工具类/方法，所有脚本均可使用。

## 管理脚本

在开发-脚本菜单中查看所有脚本。

在脚本列表中可以查看脚本名称、脚本类型、运行状态、最近更新人等信息，可直接在列表中暂停脚本、删除脚本等。

点击右上角新建脚本按钮，填写脚本名称，是否是客户端脚本等信息后提交，此时脚本列表会新增一条脚本记录，点击脚本名称，进入脚本编辑页面。

![](https://tcs-devops.aliyuncs.com/storage/113471fe0db517ebda1c022e886c10956ea9?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODA5NTA1MywiaWF0IjoxNzE3NDkwMjUzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzQ3MWZlMGRiNTE3ZWJkYTFjMDIyZTg4NmMxMDk1NmVhOSJ9.139Si54WLlfYrmAIO0JQdlGgXtInkZT5CpKtQFvx-WQ&download=image.png "")

将脚本代码粘贴进代码编辑框中，代码编辑框支持代码高亮，点击右上角保存，系统会自动识别脚本类型，类名等信息。

对于除RecordEvent脚本外的其他脚本类型，保存后脚本会立即生效。

对于RecordEvent脚本，需要继续部署，绑定至某一自定义对象。

点击左侧部署页签，进入部署页面，

![](https://tcs-devops.aliyuncs.com/storage/1134bb809cc6975ca9cbc2ce52eadc3621e3?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODA5NTA1MywiaWF0IjoxNzE3NDkwMjUzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzRiYjgwOWNjNjk3NWNhOWNiYzJjZTUyZWFkYzM2MjFlMyJ9.1YqHN10utF5CyFIGeSNkIPAC_064oXyM0vowWJIN7cQ&download=image.png "")

点击添加部署，绑定某一个自定义对象，提交即可。

