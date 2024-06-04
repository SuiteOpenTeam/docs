# 内置mongodb工具

## RFilters API

### 介绍

RFilters是suiteboot为custom record内置的mongo filter构造工具，可以很轻松的构造1个条件表达式。例如下面的伪代码条件表达式

```java
if ( a == ""a && ( c == "c" || b == "b" ) ) {}
```

翻译成RFilters的API如下

```java
RFilters.and(
                        RFilters.create().eq("a","a"),

                        RFilters.or(
                                RFilters.create().eq("b","b"),
                                RFilters.create().eq("c","c")
                        )
                )
```

RFilter支持为嵌套字段构造表达式，例如CustomRecord的data嵌套字段，并且可以使用下面的流畅API表达式编写

```java
RFilters.create().nested().nestedField("data").eq("id","XXXX");
```

翻译成mongodb的filter表达式如下

```json
{
  "data.id": {
    "$eq": "XXXX"
  }
}

```

因为RFilters默认的嵌套字段的值是"data."，所以也可以编写成如下形式的API

```java
RFilters.create().nested().eq("id","XXXX").toDocument().toJson()
```



### 支持的MongoDB操作符

1. 比较查询符号

    1. $eq

    1. $gt

    1. $gte

    1. $in

    1. $nin

    1. $lt

    1. $lte

    1. $ne

1. 逻辑查询符号

    1. $and

    1. $or

    1. $nor

    1. $not

1. 元素查询符号

    1. $exists

    1. $type

1. 评估查询符号

    1. $regex

    1. $text

1. 数据查询符号

    1. $all

    1. $size

    1. $eleMatch



## RUpdates API

### 介绍

RUpdates是suiteboot为custom record内置的mongo 更新表达式的构造工具，可以很轻松的构造1个更新表达式。例如下面的伪代码条件表达式

```text
	set a = 'a' , lastUpdateTime = NOW() , version = version + 1
```

翻译成RUpdates的API如下

```java
RUpdates.create().nested().nestedField("data")
                        .set("a","a")
                        .currentDate("lastUpdateTime", RUpdates.CurrentDateType.DATE)
                        .inc("version")
```

翻译成Mongodb的更新语句表达式如下

```json
{
  "$set": {
    "data.a": "a"
  },
  "$currentDate": {
    "data.lastUpdateTime": {
      "$type": "date"
    }
  },
  "$inc": {
    "data.version": 1
  }
}
```

因为RUpdates默认的嵌套字段的值是"data."，所以也可以编写成如下形式的API

```java
RUpdates.create().nested().nestedField("data")
                        .set("a","a")
                        .currentDate("lastUpdateTime", RUpdates.CurrentDateType.DATE)
                        .inc("version")
```



### 支持的MongoDB操作符

1. 字段更新操作符

    1. $set

    1. $unset

    1. $currentDate

    1. $inc

    1. $setOnInsert

    1. $max

    1. $min

    1. $mul

1. 数组更新操作符

    1. $addToSet

    1. $pop

    1. $pull

    1. $pullAll



## RSorts API

### 介绍

RSorts是suiteboot为custom record内置的mongo排序式的构造工具，可以很轻松的构造1个排序表达式。例如下面的伪代码条件表达式

```text
sort by _id desc, score asc
```

翻译成RSorts的API如下

```java
RSorts.create().nested().nestedField("data").desc("_id").asc("score")
```

翻译成Mongodb的排序表达式如下

```json
{
  "data._id": -1,
  "data.score": 1
}
```

因为RSorts默认的嵌套字段的值是"data."，所以也可以编写成如下形式的API

```java
RSorts.create().nested().desc("_id").asc("score")
```



## RProjections API

### 介绍

RProjections是suiteboot为custom record内置的mongo字段映射的构造工具，可以很轻松的构造1个映射表达式。例如下面的伪代码条件表达式

```text
select _id as id, data.name as userName from .....
```

翻译成RProjections APi如下

```java
RProjections.create()
                        .unnested().alias("_id","id")
                        .nested().alias("name","userName");
```

翻译成Mongodb的映射表达式如下

```json
{
  "id": "$_id",
  "userName": "$data.name"
}
```

