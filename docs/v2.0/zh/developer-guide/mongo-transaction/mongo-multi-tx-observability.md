# 观测mongo多事务

## 介绍

suiteboot基础框架给事务实现了可观测的功能，主要提供两种可观测的数据（Metric&Trace），主要观测以下两种对象的运行情况

1. TransactionContext，一个TransactionContext会包含多个TransactionCommand

1. TransactionCommand，一个TransactionCommand对应1个具体的mongodb crud操作



### **TransactionContext运行情况**

**运行过程中的Trace数据包含以下信息**

1. Java调用堆栈的入口函数名称（mongo.transaction.context.function.namespace）

1. 触发用户ID（mongo.transaction.user.id）

1. 创建的事务Session数量（mongo.transaction.context.actual_session.count）

1. 每个事务Session的最终状态（mongo.transaction.context.session.state）

**Trace效果图**

![](https://tcs-devops.aliyuncs.com/storage/1135b333bc200603a78cc3a9d850692585bb?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODE1OTA4NCwiaWF0IjoxNzE3NTU0Mjg0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzViMzMzYmMyMDA2MDNhNzhjYzNhOWQ4NTA2OTI1ODViYiJ9.vTmVFBjieLZsVZRl3PPk2zl5ogL4TR1FraBGIQlW5_Q&download=image.png "")

**运行过程中的Metric数据包含以下信息**

1. 运行时长（mongo.transaction.context.duration）以秒为单位

**Metric效果图**

![](https://tcs-devops.aliyuncs.com/storage/11359af0419cc7411c0285ed4640249178b1?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODE1OTA4NCwiaWF0IjoxNzE3NTU0Mjg0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzU5YWYwNDE5Y2M3NDExYzAyODVlZDQ2NDAyNDkxNzhiMSJ9.0Om69JtJkoaCzUbh6bN_zEsee9fqF_DSviNJ4ilBmb0&download=screencapture-localhost-9099-graph-2024-04-23-10_13_42.png "")

### **TransactionCommand运行情况**

**运行过程中的Trace数据包含以下信息**

1. [__Command的类型__](https://thoughts.aliyun.com/workspaces/64473d10725662001abf2a93/docs/66207b5451244b0001973810#6621e970b8a5b602e5d4e367)（mongo.transaction.command.type）

1. Command的开始运行时间（mongo.transaction.command.create.nano）以纳秒为单位

1. Command的结束运行时间（mongo.transaction.command.start.end）以纳秒为单位

1. Command的Java调用堆栈信息（mongo.transaction.java.action）

1. Command操作的mongo Collection名称（mongo.transaction.collection.name）

1. Command的依赖列表（mongo.transaction.command.depend）

1. 触发用户ID（mongo.transaction.user.id）

1. [__Command在执行过程中是否被中断__](https://thoughts.aliyun.com/workspaces/64473d10725662001abf2a93/docs/66207b5451244b0001973810#162044a0-0111-11ef-8baf-c7cf4ffb2ba1-30437)（mongo.transaction.command.interrupt）

**Trace效果图**

![](https://tcs-devops.aliyuncs.com/storage/1135ef078e9731e5050375e5f17da7854b52?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODE1OTA4NCwiaWF0IjoxNzE3NTU0Mjg0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzVlZjA3OGU5NzMxZTUwNTAzNzVlNWYxN2RhNzg1NGI1MiJ9.MLJPXQa4RRQHQR2hNsoqDn25LSrZgx73tEIDf9dvE54&download=screencapture-localhost-16686-trace-183829da1e10a82843623fa3474b61fd-2024-04-23-10_15_00.png "")

**运行过程中的Metric数据包含以下信息**

1. 运行时长（mongo.transaction.command.duration）以秒为单位

**Metric效果图**

![](https://tcs-devops.aliyuncs.com/storage/113569208505210fb37f5b9366aff550cb1b?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODE1OTA4NCwiaWF0IjoxNzE3NTU0Mjg0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzU2OTIwODUwNTIxMGZiMzdmNWI5MzY2YWZmNTUwY2IxYiJ9.rG01sXxoRlFVk9_E_dw8CBPEv-vAXbQENjBFEJyq8Q8&download=screencapture-localhost-9099-graph-2024-04-23-10_15_54.png "")



## 如何开启多事务可观测功能

suiteboot实现的多事务开启可观测性很简单，只需要设置WithTransaction的observable属性即可，如下所示

```java
    @WithTransaction(observable = true, concurrency = 2, enableLock = false)
    @GetMapping(value = "otel5")
    public AjaxResult upsert() {
        List<CustomRecord> salesOrderList = txRecordClient.list(
                "sales_order",
                RFilters.create().unnested().recordIdIn(93215, 93216, 93217)
        );

        salesOrderList.get(0).setRecordId(null);
        salesOrderList.get(0).setId(null);

        txRecordClient.upsertBatch(
                List.of(
                        salesOrderList.get(0),
                        salesOrderList.get(1)
                )
        );
        txRecordClient.insertBatch(
                List.of()
        );
        txRecordClient.replace(
                salesOrderList.get(2)
        );

        return AjaxResult.success();
    }

```



