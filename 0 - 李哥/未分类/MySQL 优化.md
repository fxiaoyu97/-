# MySQL 优化

- 表关联查询时务必遵循 **小表驱动大表** 原则；
- 使用查询语句 `where` 条件时，不允许出现 **函数**，否则索引会失效；
- 使用单表查询时，相同字段尽量不要用 `OR`，因为可能导致索引失效，比如：`SELECT * FROM table WHERE name = '手机' OR name = '电脑'`，可以使用 `UNION` 替代；
- `LIKE` 语句不允许使用 `%` 开头，否则索引会失效；
- 组合索引一定要遵循 **从左到右** 原则，否则索引会失效；比如：`SELECT * FROM table WHERE name = '张三' AND age = 18`，那么该组合索引必须是 `name,age` 形式；
- 索引不宜过多，根据实际情况决定，尽量不要超过 10 个；
- 每张表都必须有 **主键**，达到加快查询效率的目的；
- 分表，可根据业务字段尾数中的个位或十位或百位（以此类推）做表名达到分表的目的；
- 分库，可根据业务字段尾数中的个位或十位或百位（以此类推）做库名达到分库的目的；
- 表分区，类似于硬盘分区，可以将某个时间段的数据放在分区里，加快查询速度，可以配合 **分表 + 表分区** 结合使用；

## [#](https://www.funtl.com/zh/interview/Java-面试宝典-MySQL-优化.html#神器-explain-语句)神器 `EXPLAIN` 语句

`EXPLAIN` 显示了 MySQL 如何使用索引来处理 `SELECT` 语句以及连接表。可以帮助选择更好的索引和写出更优化的查询语句。

使用方法，在 `SELECT` 语句前加上 `EXPLAIN` 即可，如：

```sql
EXPLAIN SELECT * FROM tb_item WHERE cid IN (SELECT id FROM tb_item_cat)
```

![img](https://www.funtl.com/assets1/Lusifer_20190210233927.png)

- **id：** SELECT 识别符。这是 SELECT 的查询序列号

- select_type：

   

  SELECT类型,可以为以下任何一种

  - SIMPLE: 简单 SELECT(不使用 UNION 或子查询)
  - PRIMARY: 最外面的 SELECT
  - UNION: UNION 中的第二个或后面的 SELECT 语句
  - DEPENDENT UNION: UNION 中的第二个或后面的 SELECT 语句,取决于外面的查询
  - UNION RESULT: UNION 的结果
  - SUBQUERY: 子查询中的第一个 SELECT
  - DEPENDENT SUBQUERY: 子查询中的第一个 SELECT,取决于外面的查询
  - DERIVED: 导出表的 SELECT(FROM 子句的子查询)

- **table：** 输出的行所引用的表

- **partitions：** 表分区

- type：

   

  联接类型。下面给出各种联接类型，按照

   

  从最佳类型到最坏类型

   

  进行排序

  - system: 表仅有一行(=系统表)。这是 const 联接类型的一个特例。
  - const: 表最多有一个匹配行,它将在查询开始时被读取。因为仅有一行,在这行的列值可被优化器剩余部分认为是常数。const 表很快,因为它们只读取一次!
  - eq_ref: 对于每个来自于前面的表的行组合, 从该表中读取一行。这可能是最好的联接类型, 除了 const 类型。
  - ref: 对于每个来自于前面的表的行组合, 所有有匹配索引值的行将从这张表中读取。
  - ref_or_null: 该联接类型如同 ref,但是添加了 MySQL 可以专门搜索包含 NULL 值的行。
  - index_merge: 该联接类型表示使用了索引合并优化方法。
  - unique_subquery: 该类型替换了下面形式的 IN 子查询的 ref: `value IN (SELECT primary_key FROM single_table WHERE some_expr) unique_subquery` 是一个索引查找函数, 可以完全替换子查询, 效率更高。
  - index_subquery: 该联接类型类似于 unique_subquery。可以替换 IN 子查询, 但只适合下列形式的子查询中的非唯一索引: `value IN (SELECT key_column FROM single_table WHERE some_expr)`
  - range: 只检索给定范围的行,使用一个索引来选择行。
  - index: 该联接类型与 ALL 相同,除了只有索引树被扫描。这通常比 ALL 快,因为索引文件通常比数据文件小。
  - ALL: 对于每个来自于先前的表的行组合, 进行完整的表扫描。

- **possible_keys：** 指出 MySQL 能使用哪个索引在该表中找到行

- **key：** 显示 MySQL 实际决定使用的键(索引)。如果没有选择索引, 键是 NULL。

- **key_len：** 显示 MySQL 决定使用的键长度。如果键是 NULL, 则长度为 NULL。

- **ref：** 显示使用哪个列或常数与 key 一起从表中选择行。

- **rows：** 显示 MySQL 认为它执行查询时必须检查的行数。多行之间的数据相乘可以估算要处理的行数。

- **filtered：** 显示了通过条件过滤出的行数的百分比估计值。

- Extra：

   

  该列包含 MySQL 解决查询的详细信息

  - Distinct: MySQL 发现第 1 个匹配行后,停止为当前的行组合搜索更多的行。
  - Not exists: MySQL 能够对查询进行 LEFT JOIN 优化, 发现 1 个匹配 LEFT JOIN 标准的行后, 不再为前面的的行组合在该表内检查更多的行。
  - range checked for each record (index map: #): MySQL 没有发现好的可以使用的索引, 但发现如果来自前面的表的列值已知, 可能部分索引可以使用。
  - Using filesort: MySQL 需要额外的一次传递, 以找出如何按排序顺序检索行。
  - Using index: 从只使用索引树中的信息而不需要进一步搜索读取实际的行来检索表中的列信息。
  - Using temporary: 为了解决查询, MySQL 需要创建一个临时表来容纳结果。
  - Using where: WHERE 子句用于限制哪一个行匹配下一个表或发送到客户。
  - Using sort_union(...), Using union(...), Using intersect(...): 这些函数说明如何为 index_merge 联接类型合并索引扫描。
  - Using index for group-by: 类似于访问表的 Using index 方式,Using index for group-by 表示 MySQL 发现了一个索引,可以用来查询 GROUP BY 或 DISTINCT 查询的所有列, 而不要额外搜索硬盘访问实际的表。
