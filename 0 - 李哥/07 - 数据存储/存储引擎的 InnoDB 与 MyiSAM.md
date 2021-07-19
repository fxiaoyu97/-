# 存储引擎的 InnoDB 与 MyiSAM

1. InnoDB 不支持 `FULLTEXT` 类型的索引。
2. InnoDB 中不保存表的具体行数，也就是说，执行 `select count() from table` 时，InnoDB 要扫描一遍整个表来计算有多少行，但是 MyISAM 只要简单的读出保存好的行数即可。注意的是，当 `count()` 语句包含 `where` 条件时，两种表的操作是一样的。
3. 对于 `AUTO_INCREMENT` 类型的字段，InnoDB 中必须包含只有该字段的索引，但是在 MyISAM 表中，可以和其他字段一起建立联合索引。
4. `DELETE FROM table` 时，InnoDB 不会重新建立表，而是一行一行的删除。
5. `LOAD TABLE FROM MASTER` 操作对 InnoDB 是不起作用的，解决方法是首先把 InnoDB 表改成 MyISAM 表，导入数据后再改成 InnoDB 表，但是对于使用的额外的 InnoDB 特性(例如外键)的表不适用。

另外，InnoDB 表的行锁也不是绝对的，假如在执行一个 SQL 语句时 MySQL 不能确定要扫描的范围，InnoDB 表同样会锁全表，例如 `update table set num=1 where name like “%aaa%”`