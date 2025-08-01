# Day27 · MySQL（事务、事务提交、ACID、并发事务问题、事务隔离）

## 1. 事务概述

事务是一组操作的集合，是数据库的最小执行单元。事务具有“**要么全部执行，要么全部失败**”的特性，必须作为一个整体提交或回滚，保证数据的一致性与可靠性。

---

## 2. MySQL 事务

MySQL 在客户端连接时创建事务日志。事务开始后，所有操作会保存在事务日志中，只有执行 `COMMIT` 时才会同步至数据库。

### 2.1 自动提交

MySQL 默认每条 DML 语句自动提交事务：

```sql
SELECT @@autocommit; -- 查看自动提交状态（1 开启，0 关闭）
SET @@autocommit = 0; -- 关闭自动提交（当前会话）
```

### 2.2 手动提交

```sql
START TRANSACTION; 或 BEGIN; -- 开启事务
COMMIT; -- 提交事务
ROLLBACK; -- 回滚事务
SAVEPOINT 名字; -- 设置保存点
ROLLBACK TO SAVEPOINT 名字; -- 回滚到保存点
RELEASE SAVEPOINT 名字; -- 删除保存点
```

> 一般通过关闭自动提交、手动开启事务来实现多个语句的统一控制。

**例：手动提交事务**

```sql
START TRANSACTION;
UPDATE account SET balance = balance - 500 WHERE name = '张三';
UPDATE account SET balance = balance + 500 WHERE name = '李四';
COMMIT;
-- 若操作异常可使用 ROLLBACK;
```

**例：保存点**

```sql
START TRANSACTION;
UPDATE account SET balance = balance - 10 WHERE name = '张三';
SAVEPOINT sp1; -- 设置保存点 sp1
UPDATE account SET balance = balance - 20 WHERE name = '李四';
SAVEPOINT sp2; -- 设置保存点 sp2
ROLLBACK TO SAVEPOINT sp1; -- 回滚到保存点 sp1（撤销李四的修改）
COMMIT; -- 提交事务，此时仅张三的修改生效
```

---

## 3. 事务的四大特性（ACID）

| 特性                      | 说明                                                 |
| ------------------------- | ---------------------------------------------------- |
| **原子性**（Atomicity）   | 事务中的所有操作要么都执行，要么都不执行             |
| **一致性**（Consistency） | 执行前后数据应保持一致（如账户余额总额不变）         |
| **隔离性**（Isolation）   | 多个事务之间互不干扰                                 |
| **持久性**（Durability）  | 一旦事务提交，数据将永久保存，即使系统崩溃也不会丢失 |

---

## 4. 并发事务问题

| 问题类型       | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| **脏读**       | 读取到未提交事务中的修改数据                                 |
| **不可重复读** | 同一事务中两次读取**结果**不一致（数据被其他事务修改）       |
| **幻读**       | 同一事务中读取的数据**数量**不一致（数据被其他事务新增或删除） |

> 脏读关乎是否读取到**未提交数据**，不可重复读关乎**数据内容在事务中被改动**，幻读关乎**数据条目数量发生变化**。

---

## 5. 事务隔离级别

| 隔离级别             | 脏读 | 不可重复读 | 幻读 | 说明                             |
| -------------------- | ---- | ----------- | ----- | -------------------------------- |
| **`READ UNCOMMITTED`** | ✅ | ✅         | ✅ | 最宽松，允许读取未提交数据       |
| **`READ COMMITTED`** | ❌  | ✅         | ✅   | Oracle/SQL Server 默认           |
| **`REPEATABLE READ`** | ❌  | ❌         | ✅   | MySQL 默认                       |
| **`SERIALIZABLE`**   | ❌  | ❌         | ❌   | 最严格，性能最差                 |

> 隔离等级越严格，事务的并发安全性越高，但数据库的性能和并发能力通常会随之下降。

### 5.1 设置隔离级别

```sql
SELECT @@transaction_isolation; -- 查看当前隔离级别
SET SESSION TRANSACTION ISOLATION LEVEL [隔离级别]; -- 会话 隔离级别
SET GLOBAL TRANSACTION ISOLATION LEVEL [隔离级别]; -- 全局 隔离级别
```

> 会话级别的隔离设置仅对当前连接有效；`GLOBAL` 级别则只影响**新建立的连接**，对当前连接和已有连接无效。

---

**例：不可重复读模拟**

```sql
-- A连接
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
SELECT balance FROM account WHERE name = '张三';
-- B连接在此修改数据并提交
SELECT balance FROM account WHERE name = '张三'; -- 第二次读取得到不同结果（即使当前事务没有提交）

-- B连接
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
UPDATE account SET balance = 0 WHERE name = '张三';
COMMIT;
```

---

## ✅ 总结

- 事务用于保障操作的完整性，确保数据一致。
- 使用 `START TRANSACTION`/`COMMIT`/`ROLLBACK` 控制事务行为。
- **事务四大特性 ACID**：原子性、一致性、隔离性、持久性。
- 并发访问可能引发**脏读**、**不可重复读**、**幻读**等问题。
- 合理选择隔离级别，可平衡安全性与性能。
