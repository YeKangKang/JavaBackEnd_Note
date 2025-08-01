# Day27 · JDBC

## 1. JDBC 概述

JDBC（Java DataBase Connectivity）是 Java 提供的一套操作关系型数据库的标准接口规范，位于 `java.sql` 包中。

- JDBC 屏蔽了不同数据库间的差异，提供统一的访问方式
- 各大数据库厂商负责实现 JDBC 接口并提供对应驱动（jar 包）
- JDBC 的本质：**规范接口 + 驱动实现**

## 2. JDBC 核心对象与作用

| 对象                              | 类型 | 说明                                                      |
| --------------------------------- | ---- | --------------------------------------------------------- |
| `DriverManager`                   | 类   | 驱动管理器，注册驱动、建立连接                            |
| `Connection`                      | 接口 | 数据库连接对象，创建执行器、事务控制                      |
| `Statement` / `PreparedStatement` | 接口 | 执行 SQL 的对象（前者为静态 SQL，后者支持预编译、防注入） |
| `ResultSet`                       | 接口 | 查询结果集，用于读取数据                                  |

### 2.1 `DriverManager` API

| 方法                                                         | 说明                                                       |
| ------------------------------------------------------------ | ---------------------------------------------------------- |
| `Class.forName("com.mysql.cj.jdbc.Driver")`                  | 加载驱动类并自动完成注册，推荐方式                         |
| **`getConnection(String url, String user, String password)`** | **获取连接对象**，连接参数包含主机、端口、数据库、附加参数 |

### 2.2 `Connection` API

| 方法                                | 说明                                        |
| ----------------------------------- | ------------------------------------------- |
| **`createStatement()`**             | **获取执行静态 SQL 的对象**                 |
| **`prepareStatement(String sql)`**  | **获取预编译 SQL 执行对象**，支持参数占位符 |
| `setAutoCommit(boolean autoCommit)` | 设置事务是否自动提交                        |
| `commit()`                          | 提交当前事务                                |
| `rollback()`                        | 回滚当前事务                                |

### 2.3 `Statement` API

| 方法                            | 说明                   | 返回值                                             |
| ------------------------------- | ---------------------- | -------------------------------------------------- |
| **`executeUpdate(String sql)`** | 执行 **DDL / DML**     | 返回受影响行数（int）                              |
| **`executeQuery(String sql)`**  | 执行 **DQL**（SELECT） | 返回查询结果集（ResultSet）                        |
| `execute(String sql)`           | 执行任意 SQL 语句      | 返回 `true` 表示返回了结果集，`false` 表示无结果集 |

> `execute()` 通用但复杂，需要使用 `Statement.getResultSet()` 来获取结果集。开发中推荐使用语义明确的 `executeUpdate()` 和 `executeQuery()`。

### 2.4 `ResultSet` API

| 方法                    | 说明                             |
| ----------------------- | -------------------------------- |
| `next()`                | 光标下移一行，判断是否存在下一行 |
| `getXxx(int index)`     | 根据列索引获取值（从 1 开始）    |
| `getXxx(String column)` | 根据列名获取值                   |

> 若方法与数据类型不匹配，驱动会尝试类型转换，失败时抛出异常。

## 3. JDBC 七步法

注册驱动 → 建立连接 → 定义 SQL → 创建执行器 → 执行语句 → 处理结果 → 释放资源

```java
String url = "jdbc:mysql://域名:端口/数据库名?参数键值对...";
String user = "账号";
String password = "密码";

// 1. 注册驱动
Class.forName("com.mysql.cj.jdbc.Driver");
// 2. 建立连接
Connection conn = DriverManager.getConnection(url, user, password);
// 3. 定义 SQL
String sql = "SELECT * FROM category";
// 4. 创建执行器对象
Statement stmt = conn.createStatement();
// 5. 执行 SQL
ResultSet rs = stmt.executeQuery(sql);
// 6. 处理结果
while (rs.next()) {
    int cid = rs.getInt("cid");
    String cname = rs.getString("cname");
    System.out.println(cid + ", " + cname);
}
// 7. 释放资源
rs.close();
stmt.close();
conn.close();
```

## ✅ 总结

- JDBC 提供统一的数据库访问接口，底层由驱动实现
- 四大核心对象：`DriverManager`、`Connection`、`Statement`、`ResultSet`
- 推荐使用 `executeQuery()` 与 `executeUpdate()` 分别处理查询与数据更新
- `ResultSet` 索引从 1 开始，类型需匹配对应的 `getXxx()` 方法
- 实际开发中应封装工具类以统一管理连接与资源释放