# JDBC 流程

- 向 DriverManager 类注册驱动数据库驱动程序
- 调用 DriverManager.getConnection 方法， 通过 JDBC URL，用户名，密码取得数据库连接的 Connection 对象。
- 获取 Connection 后， 便可以通过 createStatement 创建 Statement 用以执行 SQL 语句。
- 有时候会得到查询结果，比如 select，得到查询结果，查询（SELECT）的结果存放于结果集（ResultSet）中。
- 关闭数据库语句，关闭数据库连接。
