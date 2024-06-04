# mongo多事务支持

## 介绍



由于mongodb中的单个事务无法支持客户端并发修改操作，所以suiteboot框架实现了多事务的模式，针对于大量数据的修改操作的场景非常适合，当然也适配单事务的操作，为了保证事务读已提交的特性，事务的划分和collection对应，也就是说在一个任务上下文中，一个collection对应一个事务



## 使用

suiteboot框架的事务使用非常简单，只需要增加WithTransaction注解即可，需要配置的参数如下

1. enableLock是否开启分布式锁

1. concurrency并发度，真实的事务数量永远小于等于concurrency的值，具体事务的数量由2个因素决定

    1. 事务运行涉及到的collection的数量，优先级最高

    1. 并发度的值决定

1. observable是否开启底层可观测性，[__参考文档__](https://thoughts.aliyun.com/workspaces/64473d10725662001abf2a93/docs/6621d0464c6050000151744f)

```java
@WithTransaction(observable = true, concurrency = 7)
public AjaxResult testCustomOTEL() {

	TransactionContext.tryLockInScope(3000,"Key1","Key2");
	txRecordClient.insert(new CustomRecord());
}
```



## 概念



### TransactionContext

事务运行过程中的上下文信息和提供给开发者的一些API，提供给开发者的API如下

| 方法                      | 含义                              |
| ----------------------- | ------------------------------- |
| getUserContext          | 获取当前的用户信息                       |
|                         | 添加在事务提交之前需要发布的Event             |
|                         | 添加在事务提交之后需要发布的Event             |
|                         | 添加在事务回滚之后需要发布的Event             |
|                         | 添加在事务完成之后需要发布的Event（不论是否成功都会发布） |
| static current()        | 获取当前事务的上下文                      |
| static tryLockInScope() | 在当前事务上下文中加锁                     |



### TransactionCommand

TransactionCommand代表针对单个集合的1个原子化的事务操作（INSERT、UPDATE、REPLACE），线程安全、异步、可阻塞等待、可添加依赖的TransactionCommand，1个事务上下文中有0个或者多个TransactionCommand，每个TransactionCommand都对应唯一1个事务ID，多个TransactionCommand也可以共享1个事务；提供给开发者的API如下

| 方法                          | 含义                               |
| --------------------------- | -------------------------------- |
| getCollectionName           | 获取Command对应的Collection名称         |
| getCreateTimeNano           | 获取Command的创建时间（纳秒）               |
| getThrowable                | 获取Command运行过程中出现的异常              |
| listDependCommand           | 获取Command的依赖Command列表            |
| getCommandState             | 获取Command的状态                     |
| getTransactionOperationType | 获取Command的操作类型                   |
| getOptions                  | 获取当前Command的Record事务选项           |
| getTransactionContext       | 获取当前Command对应的TransactionContext |
| isInterrupt                 | 当前Command是否被打断执行                 |
| sync                        | 阻塞当前Command直到执行完成，返回为空           |
| get                         | 阻塞获取当前Command的执行解决，返回为对应API响应    |





### AggregateTransactionCommandImpl

AggregateTransactionCommandImpl代表执行聚合操作的Command，聚合操作和其他操作有一些特殊，MongoDB6.0之后支持了聚合操作也可以支持事务，suiteboot框架兼容了此功能，但前提是开发者在使用suiteboot多事务时，为了保证读已提交的特性，如果上下文中有聚合操作，必须只能配置并发度等于1，上下文创建的事务真实数量才会等于1，TxRecordClient#aggregate方法会创建1个AggregateTransactionCommandImpl实例



### ComplexTransactionCommandImpl

ComplexTransactionCommandImpl代表复合操作的Command，1个ComplexTransactionCommandImpl中可以嵌套多个普通的TransactionCommand，由于业务操作中可能对1批数据进行Upsert操作，那么也就有可能一批数据需要Insert，另外一批数据需要Update，ComplexTransactionCommandImpl也就对应2个TransactionCommand；TransactionCommand（INSERT）和TransactionCommand（UPDATE）；

如下图所示嵌套关系

![](https://tcs-devops.aliyuncs.com/storage/11350f2f104750e70665ea21c923116fba52?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODE1OTIwMywiaWF0IjoxNzE3NTU0NDAzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzUwZjJmMTA0NzUwZTcwNjY1ZWEyMWM5MjMxMTZmYmE1MiJ9.5JnqToMGQiCeaRGBgZCYc882zpKX-TndpKRqSV6fykw&download=image.png "")



### FindNonTransactionCommandImpl

FindNonTransactionCommandImpl代表一种不在事务中执行的Command的同步查询操作，suiteboot框架允许开发者可能不在事务上下文中执行查询操作（返回1个FindNonTransactionCommandImpl实例），当然也兼容在事务上下文中查询（返回一个普通的TransactionCommand实例）



### EmptyTransactionCommandImpl

EmptyTransactionCommandImpl代表1个无任何事务操作的Command，考虑到业务层在调用批量事务API时，可能会传递1个空的集合，EmptyTransactionCommandImpl为了兼容此种情况而产生



## 架构图

一个TransactionContext上下文中会有很多个TransactionCommand，所有的TransactionCommand并发执行，最后等待所有事务执行完成提交才会成功



![](https://tcs-devops.aliyuncs.com/storage/11353e51d57fe5cb61f279691af632a01f89?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODE1OTIwMywiaWF0IjoxNzE3NTU0NDAzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzUzZTUxZDU3ZmU1Y2I2MWYyNzk2OTFhZjYzMmEwMWY4OSJ9._wGLUr-Hql4lNzDSnyV7fecMM1DUMo5Yk7OB1LNI6c0&download=image.png "")



## 事务中断事件传播

在整个多事务的上下文运行过程中，只要有1个事务运行出错，那么所有的事务应当中断执行，suiteboot基础框架实现了利用事件传播实现了中断机制，如果当前Command对应的事务执行出错，那么Command对应的属性interrupt属性将会被设置为true



## TransactionContext对象获取

在事务上下文中声明事务注解@WithTransaction即可在上下文中获取TransactionContext对象，如下所示

```java
@WithTransaction
public void inTransactionContext() {
        TransactionContext transactionContext = TransactionContext.current();
    }
```



## TransactionCommand创建

普通的CRUD操作都会创建1个TransactionCommand对象

```java
  TransactionCommand<CustomRecord> replaceCommand = txRecordClient.replace(
                salesOrderList.get(2)
        );
```

![](https://tcs-devops.aliyuncs.com/storage/1135deb34f467a27271da053583801ba6ecd?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODE1OTIwMywiaWF0IjoxNzE3NTU0NDAzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzVkZWIzNGY0NjdhMjcyNzFkYTA1MzU4MzgwMWJhNmVjZCJ9.S6oUmNtl329RxHEL0KfH7VDcXF6cVJvpmuaf6dfIJH4&download=image.png "")

## ComplexTransactionCommand创建

对于复合的CRUD操作会触发创建ComplexTransactionCommand，例如upsert操作，部分数据可能会被insert，部分数据可能会被update，所以ComplexTransactionCommand会有多个嵌套TransactionCommand，如下所示

```java
         List<CustomRecord> salesOrderList = txRecordClient.list(
                "sales_order",
                RFilters.create().unnested().recordIdIn(93215, 93216, 93217)
        );

        salesOrderList.get(0).setRecordId(null);
        salesOrderList.get(0).setId(null);

        TransactionCommand<Void> complexTransactionCommand = txRecordClient.upsertBatch(
                List.of(
                        salesOrderList.get(0),
                        salesOrderList.get(1)
                )
        );

```

![](https://tcs-devops.aliyuncs.com/storage/1135ace782e6987288500d7ed670d7af6fe6?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODE1OTIwMywiaWF0IjoxNzE3NTU0NDAzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzVhY2U3ODJlNjk4NzI4ODUwMGQ3ZWQ2NzBkN2FmNmZlNiJ9.I43zCuMKOTL3Ystc_rkt-NlvLuXPvGNyAxOhGovd3Uw&download=image.png "")

## EmptyTransactionCommand创建

EmptyTransactionCommand代表1种无任何事务操作的Command对象，当用户输入为空的情况下会返回EmptyTransactionCommand实例，如下所示

```java
        TransactionCommand<List<CustomRecord>> emptyCommand = txRecordClient.insertBatch(
                List.of()
        );

```

![](https://tcs-devops.aliyuncs.com/storage/113591bb5c01fffe6020a8f9c2aaf9133c19?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODE1OTIwMywiaWF0IjoxNzE3NTU0NDAzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzU5MWJiNWMwMWZmZmU2MDIwYThmOWMyYWFmOTEzM2MxOSJ9.vHs4UoqdP3kVzWFPT69VfyIo_cK0V1S_CMaXD9yxD0o&download=image.png "")



## TransactionContext回调函数

suiteboot支持给当前的TransactionContext添加回调函数，分为4个阶段的回调函数（可以设置回调函数是否异步）

1. 提交所有mongo事务之前回调函数

1. 提交所有mongo事务之后回调函数

1. 回滚所有mongo事务之后的回调函数

1. 完成所有事务之后的回调函数（不论是否回滚事务，当TransactionContext生命周期结束时都会回调）

**提供给外部的增加回调函数API如下**

1. void addBeforeCommitTxCallback( TransactionContextCallback beforeSubmitCallback )

1. void addAfterCommitTxCallback( TransactionContextCallback afterCommitTxCallback  )

1. void addAfterRollbackTxCallback( TransactionContextCallback  afterRollbackTxCallback )

1. void addAfterCompleteTxCallback( TransactionContextCallback  afterCompleteTxCallback )



TransactionContext基于回调函数封装发布spring事件的API

**提供给外部的增加发布spring事件同步API如下**

1. void postEventBeforeCommit( Object event )

1. void postEventAfterCommit( Object event )

1. void postEventAfterRollback( Object event )

1. void postEventAfterCompletion( Object event )

**提供给外部的增加发布spring事件异步API如下**

1. void postEventBeforeCommitAsync( Object event )

1. void postEventAfterCommitAsync( Object event )

1. void postEventAfterRollbackAsync( Object event )

1. void postEventAfterCompletionAsync( Object event )



