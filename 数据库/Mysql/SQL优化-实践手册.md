# 1）当只要一行数据时使用limit 1

```plain
	查询表的时候，我们预期只有一条数据的时候加上limit 1 可以增加性。这样一来，mysql数据库引擎在找到一条数据后会停止搜索，而不是继续往后查找吓一跳符合记录的数据。
```

# 2）使用not exists 代替 not in

```plain
not exists 能够使用索引，而not in不能私用索引。not in是最慢的方式，要同每条记录比较。
```

# 3）尽量不采用不利用索引的操作符

```plain
比如：in 、not in、is null、is not null、<>等
```

