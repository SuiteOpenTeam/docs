# mongo事务特性

## mongo事务隔离级别



### 读已提交（Read Committed）

只能读到其他事务已经提交的数据



事务选项设置如下

1. 只能从主节点读

1. 只能从主节点写

```java
TransactionOptions READ_COMMITTED = TransactionOptions.builder()
            .writeConcern(WriteConcern.MAJORITY)
            .readConcern(ReadConcern.MAJORITY)
            // 事务级别的修改读操作必须在主节点上进行
            .readPreference(ReadPreference.primary())
            .build();
```



### 可重复读（Repeatable read）



当某个事务读到某个文档时，则不再允许当前文档被其他事务修改，并发粒度要比[__读已提交__](https://thoughts.aliyun.com/workspaces/64473d10725662001abf2a93/docs/6621d00b5e119400017e2795#6ba64060-fdf0-11ee-965a-71c9892c9399-7593)大，可以解决不可重复读的问题，但仍然存在幻读的问题，如果有其他的insert操作，仍然存在幻读的问题



事务选项设置如下

1. 只能从主节点写

1. 只能从快照中读

```java
TransactionOptions READ_COMMITTED = TransactionOptions.builder()
            .writeConcern(WriteConcern.MAJORITY)
            .readConcern(ReadConcern.SNAPSHOT)
            // 事务级别的修改读操作必须在主节点上进行
            .readPreference(ReadPreference.primary())
            .build();
```



## mongo事务特性



### 事务冲突

多个事务同时修改1个文档会造成冲突，抛出WriteConflict错误，底层默认有[__加锁机制__](https://www.mongodb.com/docs/manual/core/transactions-production-consideration/#acquiring-locks)，可以通过事务设置选项[__maxTransactionLockRequestTimeoutMillis__](https://www.mongodb.com/docs/manual/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis)设置事务加锁等待时长，默认是5毫秒，需要注意的是如果是分片集群，则需要在每个分片的副本集群上设置此参数



### 事务线程安全

单个事务中使用多线程修改同一个文档或者不同的文档都会出现NoSuchTransaction错误，结论是mongodb的事务线程不安全



### 事务锁级别

A、B 2个线程同时修改一条数据，并且A线程修改数据的a字段，B线程修改数据的b字段，都会触发WriteConflict错误，结论是事务锁的级别是行锁