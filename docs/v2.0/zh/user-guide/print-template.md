# 打印模板
菜单路径：系统设置-自定义-打印模板。

点击新增添加一个新的打印模板。

1. 填写模板名称；
1. 选择模板绑定对象；
1. 选择绑定到某个对象布局表单，如果绑定了表单，则只有在这个表单上才会显示打印按钮；
1. 上传模板，模板为Excel或者Word格式，通过提前设置好的Excel或Word样式，避免了代码生成样式的复杂性和各种坑。在模板中设置好表达式（与EL表达式相似），打印时，系统会将表达式解析为实际数据。表达式指令如下：

- 空格分割

```text
- 三目运算 {{test ? obj:obj2}}
- n: 表示 这个cell是数值类型 {{n:}}
- le: 代表长度{{le:()}} 在if/else 运用{{le:() > 8 ? obj1 : obj2}}
- fd: 格式化时间 {{fd:(obj;yyyy-MM-dd)}}
- fn: 格式化数字 {{fn:(obj;###.00)}}
- fe: 遍历数据创建row
- !fe: 遍历数据不创建row
- $fe: 下移插入把当前行 下面的行全部下移.size()行 然后插入
- #fe: 横向遍历
- v_fe: 横向遍历值
- !if: 删除当前列 {{!if:(test)}}
- 单引号表示常量值 '' 比如'1' 那么输出的就是 1
- &NULL& 空格
- ]] 换行符 多行遍历导出
- sum： 统计数据
```

模板示例：

![](https://tcs-devops.aliyuncs.com/storage/1134ae564873ec4bb306009f4f10155cab83?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODA5NDQ5OSwiaWF0IjoxNzE3NDg5Njk5LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzRhZTU2NDg3M2VjNGJiMzA2MDA5ZjRmMTAxNTVjYWI4MyJ9.MhGnWTFiVYIC7-C4F2-0djhr9NYjQ38hoK_qrzBu244&download=image.png "")

在上面示例中，通过添加{{qrcode}}或{{barcode}}字段，可以生成该对象数据的二维码或条形码，内容为该对象数据的单号或编号（对象主属性字段id的值）。



