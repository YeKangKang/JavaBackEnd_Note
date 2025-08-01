# Day25 · MySQL（概述、安装、数据类型、DDL、DML）

## 1. 数据库

- **数据库（Database）**：存储数据的仓库，本质为结构化的数据文件集合。
- **数据库管理系统（DBMS）**：操作数据库的软件系统，提供建库、查询、维护等功能。

> 常见 DBMS：`MySQL`、`MongoDB`、`SQL Server` 等

---

## 2. 数据库表

数据库中包含多个表，每张表由**记录（行）**和**字段（列）**构成：

- **字段（列）**：定义每一列的数据名称与类型，表示数据的某种属性，如 `用户名`、`年龄`
- **记录（行）**：具体的数据实例，每行存储一组字段对应的实际值

---

## 3. MySQL 安装 & 启动

推荐使用 `Homebrew` 进行安装（自动配置环境变量）。也可以通过 MySQL 官网下载安装，但需手动设置 PATH。

#### 控制台启动 MySQL

MySQL 通常会在系统启动时自动运行，也可以手动执行以下命令启动服务：

```bash
sudo /usr/local/mysql/support-files/mysql.server start	# 若通过官网安装
brew services start mysql	# 若通过 Homebrew 安装（推荐）
```

#### 登录 MySQL

启动服务后，可通过以下命令连接数据库：

- **本地连接**（连接本机 MySQL 实例）

  ```bash
  mysql -u用户名 -p密码
  ```

- **远程连接**（连接指定主机 IP）

  ```bash
  mysql -u用户名 -p密码 -h主机IP
  ```

---

## 4. MySQL 常见数据类型

| 类型         | 描述                                       |
|--------------|--------------------------------------------|
| `INT`        | 整数类型                                   |
| `DOUBLE`     | 浮点数                                     |
| `DECIMAL(m,d)` | 定点小数，适合财务类高精度数据               |
| `VARCHAR(n)` | 变长字符串，占用实际长度                     |
| `DATE`       | 日期，格式 `YYYY-MM-DD`                     |
| `DATETIME`   | 日期时间，格式 `YYYY-MM-DD HH:MM:SS`        |
| `TIMESTAMP`  | 时间戳，自动基于 UTC 转换显示                |

> 表中为常见数据类型，仅列出常用项，完整类型定义请参考 MySQL 官方文档或 API 说明

---

## 5. SQL 基础语法

SQL 指令按功能大致可分为四类：  

- **`DDL`（结构定义）**：用于定义数据库、表的结构，如 `CREATE`、`ALTER`、`DROP`

- **`DML`（数据操作）**：用于插入、更新、删除表中数据，如 `INSERT`、`UPDATE`、`DELETE`
- **`DQL`（数据查询）**：用于查询数据，如 `SELECT`、`FROM`、`WHERE`
- **`DCL`（权限控制）**：用于控制访问权限，如 `GRANT`、`REVOKE`

### 5.2 数据定义（DDL）

#### 数据库操作

- **创建数据库**

  ```mysql
  CREATE DATABASE 数据库名;	-- 默认字符集
  CREATE DATABASE 数据库名 CHARACTER SET utf8; -- 指定字符集
  ```

- **查看数据库**

  ```mysql
  SHOW DATABASES; -- 查看所有数据库
  SHOW CREATE DATABASE 数据库名; -- 查看建库语句
  SELECT DATABASE(); -- 查看当前使用的数据库
  ```

- **删除数据库**

  ```mysql
  DROP DATABASE 数据库名;
  ```

- **切换数据库**

  ```mysql
  USE 数据库名;
  ```

#### 表操作

- **创建表**

  ```mysql
  CREATE TABLE 表名 (
    列名1 数据类型 [约束],
    列名2 数据类型 [约束]
  );
  ```
  
  > `约束` 用于限制字段的取值，比如 `PRIMARY KEY`、`NOT NULL` 等。添加约束后，该字段只能接受符合条件的操作
  
- **查看表**

  ```mysql
  SHOW TABLES; -- 查看当前数据库中的所有表
  ```
  
- **删除表**

  ```mysql
  DROP TABLE 表名;
  ```
  
- **修改表结构**

  ```mysql
  ALTER TABLE 表名 ADD 字段名 数据类型 [约束]; -- 添加字段
  ALTER TABLE 表名 CHANGE 旧名 新名 数据类型 [约束]; -- 修改字段名+类型+约束
  ALTER TABLE 表名 MODIFY 字段名 数据类型 [约束]; -- 修改字段类型或约束
  ALTER TABLE 表名 DROP 字段名; -- 删除字段
  RENAME TABLE 原表名 TO 新表名; -- 修改表名
  ALTER TABLE 表名 CHARACTER SET 字符集名; -- 修改表字符集
  ```
  
  > `CHANGE` 和 `MODIFY` 都要求**完整声明字段名和数据类型**，即使未修改的部分也必须写出，不能仅写变更项

---

### 5.2 数据操作（DML）

- **插入记录**

  ```mysql
  INSERT INTO 表名 (字段1, 字段2, ...) VALUES (值1, 值2, ...); -- 插入指定字段，未写出的字段为默认值或 NULL
  INSERT INTO 表名 VALUES (值1, 值2, ...); -- 插入所有字段（必须按字段顺序）
  ```
  
  > 除数值外，其它类型需用英文单引号 `'...'` 包裹，空值可写作 `NULL` 或省略
  
- **修改记录**

  ```mysql
  UPDATE 表名 SET 字段1=值, 字段2=值; -- 修改所有行的指定字段
  UPDATE 表名 SET 字段1=值, 字段2=值 WHERE 条件; -- 修改满足条件行的字段
  ```
  
  > 赋值必须与字段类型兼容，字符串需用英文单引号 `'...'` 包裹

- **删除记录**

  ```mysql
  DELETE FROM 表名 WHERE 条件; -- 删除满足条件的整行
  ```
  
> `DELETE` 删除整行，若省略 `WHERE` 条件，将清空整张表中的所有数据

---

## ✅ 总结

- 数据库由多张表组成，表由字段（列）与记录（行）构成
- **DDL 负责结构创建与修改**，常用语句：`CREATE` / `ALTER` / `DROP`
- **DML 负责数据插入、更新、删除**，常用语句：`INSERT` / `UPDATE` / `DELETE`
- 字段修改需完整声明名称与类型，`MODIFY` 与 `CHANGE` 使用场景不同
- `DELETE` 永远整行删除，`UPDATE` 建议配合 `WHERE` 限定范围
- SQL 字段赋值遵循类型规则，字符/日期等需用 `'...'` 包裹