# 分页查询

使用 SELECT 查询时，如果结果集数据量很大，比如几万行数据，放在一个页面显示的话数据量太大，不如分页显示，每次显示 100 条。

要实现分页功能，实际上就是从结果集中显示第 1~100 条记录作为第 1 页，显示第 101~200 条记录作为第 2 页，以此类推。

因此，分页实际上就是从结果集中“截取”出第 M~N 条记录。这个查询可以通过`LIMIT <N-M> OFFSET <M>`子句实现。我们先把所有学生按照成绩从高到低进行排序：

```x-sql
-- 按score从高到低:
SELECT id, name, gender, score FROM students LIMIT 3 OFFSET 0;
```

现在，我们把结果集分页，每页 3 条记录。要获取第 1 页的记录，可以使用`LIMIT 3 OFFSET 0`：

```x-sql
-- 查询第1页:
SELECT id, name, gender, score
FROM students
ORDER BY score DESC
LIMIT 3 OFFSET 0;
```

上述查询`LIMIT 3 OFFSET 0`表示，对结果集从 0 号记录开始，最多取 3 条。注意 SQL 记录集的索引从 0 开始。

如果要查询第 2 页，那么我们只需要“跳过”头 3 条记录，也就是对结果集从 3 号记录开始查询，把`OFFSET`设定为 3：

```x-sql
-- 查询第2页:
SELECT id, name, gender, score
FROM students
ORDER BY score DESC
LIMIT 3 OFFSET 3;
```

类似的，查询第 3 页的时候，`OFFSET`应该设定为 6:

```x-sql
-- 查询第3页:
SELECT id, name, gender, score
FROM students
ORDER BY score DESC
LIMIT 3 OFFSET 6;
```

查询第 4 页的时候，`OFFSET`应该设定为 9:

```x-sql
-- 查询第4页:
SELECT id, name, gender, score
FROM students
ORDER BY score DESC
LIMIT 3 OFFSET 9;
```

由于第 4 页只有 1 条记录，因此最终结果集按实际数量 1 显示。`LIMIT 3`表示的意思是“最多 3 条记录”。

可见，分页查询的关键在于，首先要确定每页需要显示的结果数量`pageSize`（这里是 3），然后根据当前页的索引`pageIndex`（从 1 开始），确定`LIMIT`和`OFFSET`应该设定的值：

- `LIMIT`总是设定为`pageSize`；
- `OFFSET`计算公式为`pageSize * (pageIndex - 1)`。

这样就能正确查询出第 N 页的记录集。

如果原本记录集一共就 10 条记录，但我们把`OFFSET`设置为 20，会得到什么结果呢？

```x-sql
-- OFFSET设定为20:
SELECT id, name, gender, score
FROM students
ORDER BY score DESC
LIMIT 3 OFFSET 20;
```

`OFFSET`超过了查询的最大数量并不会报错，而是得到一个空的结果集。

### 注意

`OFFSET`是可选的，如果只写`LIMIT 15`，那么相当于`LIMIT 15 OFFSET 0`。

在 MySQL 中，`LIMIT 15 OFFSET 30`还可以简写成`LIMIT 30, 15`。

使用`LIMIT <M> OFFSET <N>`分页时，随着`N`越来越大，查询效率也会越来越低。

### 思考

在分页查询之前，如何计算一共有几页？

### 小结

使用`LIMIT <M> OFFSET <N>`可以对结果集进行分页，每次查询返回结果集的一部分；

分页查询需要先确定每页的数量和当前页数，然后确定`LIMIT`和`OFFSET`的值。
