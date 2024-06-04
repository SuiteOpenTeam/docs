# Distribute Lock API

## 概念



### 租约

租约是一种基于时间的机制，在此期限内拥有Lease的节点有权利（大部分情况下是独享的）对指定的对象进行某些约定好的操作。从更深层次上来看，Lease就是一把带有超时机制的分布式锁，如果没有Lease，分布式环境中的锁可能会因为锁拥有者的失败而导致死锁，有了lease死锁会被控制在超时时间之内。

租约概念来自于论文 [__An Efficient Fault-Tolerant Mechanism for Distributed File Cache Consistency__](https://www.andrew.cmu.edu/course/15-440/assets/READINGS/gray1989.pdf)



### 租约Grant

申请1个租约，默认租约过期市场是10s，在没有释放租约之前，上下文会自动延长租约的存活时间，直到lease被revoke。



### 租约Revoke

废除租约，租约所对应的分布式锁以及本身的租约信息会被自动删除



### 锁

一个租约的上下文中，可以对应多把分布式锁，锁不存在过期时间，锁的生命周期跟随租约的生命周期，当租约过期时，锁也会被自动释放。



### 锁TryLock

尝试获取多个分布式锁，如果获取失败会返回TryLockException



### 锁Unlock

解除多个分布式锁



## 实现



### Lease实现

lease基于[__MongoDB TTL Indexes__](https://www.mongodb.com/docs/manual/tutorial/expire-data/)实现，collection存储结构如下

| id        | 租约唯一ID       |
| --------- | ------------ |
| hostname  | 机器实例名称       |
| thread_id | 获取Lease的线程ID |
| create_at | 创建时间         |
| expire_at | 过期时间         |

参考com.suiteopen.boot.custom.core.lock.LeaseClient



### Lock实现

lock基于[__Mongodb Unique Indexes__](https://www.mongodb.com/docs/manual/core/index-unique/)实现，collection存储结构如下

| lock_key | 锁全局唯一ID |
| -------- | ------- |
| lease_id | 指向租约ID  |



## API Reference



### 非Spring上下文使用方式

```java
    public void testLock() throws InterruptedException {
        GrantLeaseResult result = null;
        try {
            result = lockImpl.leaseClient().grant();
            CompletableFuture<Boolean> future = lockImpl.lockClient().lock(result.getLeaseID(), "lockKey");
            future.get();
        } catch (ExecutionException | InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            TimeUnit.SECONDS.sleep(120);
            if (result != null) {
                lockImpl.leaseClient().revoke(result.getLeaseID());
            }
        }
    }
```



### Spring非事务上下文使用方式

```java
    @WithLock
    public void testLock2() {
        LockContext.current().tryLock(3000L,"LockKey1");
        // do some things
    }
```



### Spring事务上下文使用方式

```java
    @WithTransaction(enableLock = true, observable = true, concurrency = 2)
    public void testLock3() {
        TransactionContext.lock(3000L, "LockKey1");
    }
```

