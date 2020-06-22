---
title: 记录一次解决日志表的慢查询
date: 2020-06-22 21:17:45
tags: [SQL, Web]
categories: [Web Backend]
---

今天在工作中修复了两个慢查询请求。可能之后也会遇到，记录下问题，让自己更好理解。

<!--more-->

## 背景

在描述那两个慢查询场景前，需要先理下相关的表结构：

logs (日志表)

| 字段名      | 备注         |
| ----------- | ------------ |
| id          | 日志主键     |
| job_id      | 作业 ID      |
| job_group   | 作业群组标识 |
| handle_code | 执行结果编码 |
| handle_msg  | 执行结果信息 |

jobs (作业表)

| 字段名      | 备注          |
| ----------- | ------------- |
| id          | 作业主键      |
| type        | 作业类型      |
| name        | 作业名称      |
| executor_id | 作业执行器 ID |

models (模型表)

| 字段名  | 备注     |
| ------- | -------- |
| id      | 模型主键 |
| version | 模型版本 |
| job_id  | 作业 ID  |

executors (作业执行器表)

| 字段名  | 备注           |
| ------- | -------------- |
| id      | 作业执行器主键 |
| address | 作业执行器地址 |
| job_group | 作业群组标识 |

用一句话总结上面的关系是：模型会有关联一个作业，但作业有不同类型（可能不是模型），每个作业会在一个执行器中执行，然后产生执行日志信息。

好了，知道这些后，我们来看看前面提到的慢查询吧～

## TOP1 Logs

现在我们需要在模型或作业中显示其最新的执行信息，而执行信息只能在日志表中获取。这意味着我们需要从日志表中拿到作业最后的日志信息，看看下面的查询：

```sql
SELECT t1.*
FROM logs t1
WHERE t1.id = (
	SELECT MAX(id)
	FROM logs t2
	WHERE t1.job_id = t2.job_id
)
```

日志表自关联，获取每个作业的最后日志 ID, 之后通过日志 ID 获取相关记录。

看起来没什么问题，开发环境数据量少，查询起来也不会慢，so easy。

但随着日志越来越多，慢慢地就会发现，这个查询怎么 5、6 分钟都没出结果。

让我们一步步来看看上面的查询，首先根据日志中的作业 ID 获取关联作业的所有日志信息（耗时为 T1),然后从关联作业的所有日志中获取最后的 ID(耗时为 T2), 最后根据日志 ID 获取相关记录（耗时为 T3，相当于扫描一次日志表的时间)。

那上面查询的总耗时大概为： `(T1 + T2) * T3 + T3`

当日志越来越多时，T1, T2, T3 都会越来越大，此时上面查询时间复杂度为 `O(N^2)`

知道问题后，我们来优化这个查询，通过作业 ID 获取最后的日志信息，我们可以联想到使用 `GROUP BY`:

```sql
SELECT t1.*
FROM logs t1
GROUP BY t1.job_id
HAVING t1.id = MAX(t1.id)
```

但有些数据库在默认设置下执行前面的 SQL 会出现以下错误：

> Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'your_database.t1.id' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by

为了解决上面的错误，我们可以修改数据库的配置信息，参考：

```sql
SET GLOBAL sql_mode=(SELECT REPLACE(@@sql_mode,'ONLY_FULL_GROUP_BY',''));
```

如果在不修改数据库的前提下，我们可以在上面 GROUP 查询外增加一个查询，如下：

```sql
SELECT t1.*
FROM logs t1
JOIN (
	SELECT MAX(id) as max_id
	FROM logs t3
	GROUP BY t3.job_id
) t2 ON t2.max_id = t1.id
```

我们也按之前的方法分析下上面的查询耗时，首先 `GROUP BY` 将遍历一次全表并将 job id 进行分组（耗时为 T1），然后获取每个分组中最大 ID(耗时为 T2)，最后进行联表（耗时为 T3，没索引情况下，相当于两次全表扫描）。

那么上面查询总耗时为 `T1 + T2 + T3`，时间复杂度约为 `O(N)`。

## Join Logs

当我们需要在日志中显示其作业、执行器信息时，那么需要将日志表和其他表进行关联，如下面的查询：

```sql
SELECT t1.*, t2.address, t3.version, t4.name
FROM logs t1
JOIN executors t2 ON t2.job_group = t1.job_group
LEFT JOIN models t3 ON t3.job_id = t1.job_id
JOIN jobs t4 ON t4.id = t1.job_id
```

看起来上面的查询没有可以优化的空间，但随着日志表越来越大，执行将变得越来越慢。

考虑到日志表很大，尝试联表时尽可能不关联日志表，优化后查询如下：

```sql
SELECT t1.*, t2.name, t3.address, t4.version
FROM logs t1
JOIN jobs t2 ON t2.id = t1.job_id
JOIN executors t3 ON t3.id = t2.executor_id
LEFT JOIN models t4 ON t4.job_id = t2.id
```

由于使用作业表代替日志表的引用，这样联表时减少对日志表的扫描，从而提升执行速度。

## 总结

当我们对大数据量的表进行查询时，需要分外注意查询效率。联表使用其他表代替大表的关联，使用适合的聚合函数，这样就可以减小对大表的扫描，从而加快查询。

## 参考

1. [SELECT list is not in GROUP BY clause and contains nonaggregated column … incompatible with sql_mode=only_full_group_by](https://stackoverflow.com/questions/41887460/select-list-is-not-in-group-by-clause-and-contains-nonaggregated-column-inc)
2. [SQL数据分析之 group by 的实现原理](https://zhuanlan.zhihu.com/p/86613661)