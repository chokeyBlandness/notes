# sql查询不是以select开始

> 翻译自：https://jvns.ca/blog/2019/10/03/sql-queries-don-t-start-with-select/

显然，许多SQL查询语句都是以`SELECT`开头的（这里只讨论`SELECT`查询，而非`INSERT`或者其他）。但是在真实的查询顺序中，SELECT并不是第一个执行的，而是第五个执行的。执行顺序如下：

- `FROM + JOIN`和所以的`ON`条件
- `WHERE`
- `GROUP BY`
- `HAVING ON`
- `SELECT`，包括窗口函数
- `ORDER BY`
- `LIMIT`

### 此执行顺序所能帮我们解决的问题

上面sql查询的执行顺序能够帮助更好的理解一条sql查询如何返回数据，同时页能回答如下的问题：

- 是否能将`GROUP BY`中得到的数据进行`WHERE`查询？（不行！WHERE发生在GROUP BY之前）
- 是否能将窗口函数返回的结果进行过滤？（不行！窗口函数在`SELECT`步骤执行，发生在`WHERE`和`GROUP BY`之后）
- 是否能`GROUP BY`的数据进行`ORDER BY `操作？（可以！`ORDER BY`基本上是最后执行的，可以排序任何数据）
- `LIMIT`什么时候执行？（最后）

##### 数据库引擎最终的执行并不是按照这个顺序

因为数据库引擎实现了大量的优化来让查询能够执行的更快。因此：

- 当你只想了解哪些查询是有效的，以及推理一个sql查询的结果时，可以参考这个执行顺序。
- 你不应该参考此顺序来推理查询性能或者任何涉及索引的因素，这是一个复制的多、变量大的多的事。

### 混淆因素：列别名

在sql查询有，有存在列别名的查询语法：

```sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name, count(*)
FROM table
GROUP BY full_name
```

这条查询看上去`GROUP BY`是在`SELECT`之后执行的，因为`GROUP BY`引用了`SELECT`中的列别名。但是实际上还是`GROUP BY`在前执行，数据库引擎只需要重写查询即可：

```sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name, count(*)
FROM table
GROUP BY CONCAT(first_name, ' ', last_name)
```

这样`GROUP BY`就能先执行了。数据库引擎也会做大量的检查，以确保在执行查询之前`SELECT`和`GROUP BY`都是有意义的。

### 真正的查询不是按照这个顺序（优化）

实际上，数据库引擎并不会通过`JOIN`、过滤、`ORDER BY`的顺序来运行查询，因为它实现了一系列优化，重新排序来使查询运行的更快，同时不会改变查询的结果。

一个简单的例子来说明为什么需要通过不同的顺序来执行查询以使其快速运行：

```sql
SELECT * FROM
owners LEFT JOIN cats ON owners.id = cats.owner
WHERE cats.name = 'mr darcy'
```

如果你只需要查询名为“mr darcy”的3只猫，那么执行整个左连接并匹配2个表中所有行将是很愚蠢的。首先对名为“mr darcy”的猫进行过滤会让查询执行的更快，同时也不会改变查询的结果。数据库引擎在实际过程中实现了许多其他的优化而可能以不同的顺序运行查询。



