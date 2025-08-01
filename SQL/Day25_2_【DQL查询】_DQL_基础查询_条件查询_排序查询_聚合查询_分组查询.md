# Day25 · MySQL（DQL、基础查询、条件查询、排序查询、聚合查询、分组查询）

---

## 1. DQL 概述

- DQL（数据查询语言）用于从表中**按条件筛选并返回满足要求的数据字段**  
- 查询语句以 `SELECT` 开头，默认扫描所有行，仅返回被选中的字段

---

## 2. 简单查询（字段选择、别名、去重、表达式）

- **基础查询**
  ```mysql
  SELECT * FROM product; -- 查询所有字段
  SELECT pname, price FROM product; -- 查询指定字段
  ```
  > 查询只返回被选中的字段，未选出的字段将无法参与后续操作

- **别名**
  
  ```mysql
  SELECT * FROM product AS p; -- 表别名
  SELECT pname AS pn FROM product; -- 字段别名
  ```
  > **表别名**常用于多表查询，避免字段重名时的歧义；**字段别名**用于统一输出字段名或改善展示效果
  
- **去重**
  
  ```mysql
  SELECT DISTINCT price FROM product; -- 去除重复的 price 值
  ```
  > `DISTINCT` 会对选中字段组合整体去重，仅当**所有选中字段值完全相同时**才算重复
  
- **查询中的表达式**
  
  ```mysql
  SELECT pname, price + 10 FROM product; -- 查询字段 + 表达式
  ```

---

## 3. 条件查询（`WHERE` + 运算符）

### 3.1 比较运算符

| 运算符       | 说明                      |
|--------------|---------------------------|
| **`=` `!=`** | 等于 / 不等于              |
| **`<` `<=` `>` `>=`** | 大小比较           |
| **`IN (...)`** | 是否包含在集合内           |
| **`BETWEEN ... AND ...`** | 是否在区间内        |
| **`LIKE`**   | 模糊匹配（`%` 多字符，`_` 单字符） |
| **`IS NULL`** | 是否为空                   |

```mysql
SELECT * FROM product WHERE pname = '可乐';
SELECT * FROM product WHERE price IN (200, 350, 1000);
SELECT * FROM product WHERE pname LIKE '%巧克力'; -- 以巧克力结尾
SELECT * FROM product WHERE category_id IS NULL;
SELECT * FROM product WHERE price BETWEEN 200 AND 1000;
```

---

### 3.2 逻辑运算符

| 运算符     | 说明         |
|------------|--------------|
| **`AND` `&&`** | 与（同时满足） |
| **`OR` `||`** | 或（满足其一） |
| **`NOT` `!`** | 非（取反）     |

```mysql
SELECT * FROM product WHERE price >= 200 AND price <= 1000;
SELECT * FROM product WHERE price = 200 OR price = 800;
SELECT * FROM product WHERE NOT(price = 800); -- 等价于 price != 800
```

---

## 4. 排序查询（ORDER BY）

`ORDER BY` 可指定查询结果的排序方式：**`ASC` 表示升序（默认）**，**`DESC` 表示降序**。排序可同时使用多个字段，按优先级依次生效。

- **基本排序**

  ```mysql
  SELECT * FROM product ORDER BY price DESC; -- 按价格降序
  SELECT DISTINCT price FROM product ORDER BY price DESC; -- 去重后排序
  ```

- **多级排序**

  ```mysql
  SELECT * FROM product ORDER BY price DESC, category_id ASC; -- 先按照价格降序，再对价格相同的值按照分类升序排序
  ```
  > 多字段排序时，只有**首字段值相同**的记录，才会继续参考下一个排序字段。

---

## 5. 聚合查询（聚合函数）

聚合函数用于对 **当前组中某列** 的多行数据进行统计或汇总，返回一个单一的聚合结果。

| 函数           | 说明                           |
|----------------|--------------------------------|
| `COUNT(col)`   | <u>当前组中</u>**该列**非 NULL 值的个数 |
| `MAX(col)`     | 当前组中该列的最大值            |
| `MIN(col)`     | 当前组中该列的最小值            |
| `SUM(col)`     | 当前组中该列值的总和            |
| `AVG(col)`     | 当前组中该列值的平均值          |

```mysql
SELECT COUNT(*) FROM product;
SELECT COUNT(price) FROM product WHERE price > 200;
SELECT SUM(price) AS total_price FROM product;
SELECT SUM(price) FROM product WHERE category_id = 'c001';
SELECT MAX(price), MIN(price) FROM product;
```

---

## 6. 分组聚合（GROUP BY + HAVING）

分组查询用于将表中的数据**按一个或多个字段进行分组**，仅当分组字段的值全部相同时才归为同一组；随后可在**每个分组**内部**应用聚合函数进行统计分析**。

- 分组后的聚合函数作用域限定在“每组”内
- 所有非聚合字段必须出现在 `GROUP BY` 中
- 分组后的过滤使用 `HAVING`（不能用 `WHERE`）

```mysql
SELECT price, COUNT(price) FROM product GROUP BY price;
SELECT pid, price, COUNT(price) FROM product GROUP BY pid, price;
SELECT price, COUNT(price) FROM product GROUP BY price HAVING price = 5000;
```

> 所有聚合函数的作用域都是“当前组”——若查询未分组，则整张表视为一组；若使用 `GROUP BY`，则会**对每组分别执行一次聚合函数**，最终将所有结果拼接成一个“多行一列”（或多列）的结果表。

> 在 `GROUP BY` 分组语法中，如果所选中列没有 `NULL` 值，则 `COUNT(col)` 的结果与 `COUNT(*)` 等价，**不管你写的是 `id`、`price`、还是其他字段，得到的行数都一样**。只有当所选列中存在 `NULL`，才会导致 `COUNT(col)` 的统计结果少于 `COUNT(*)`。

---

## ✅ 总结

- `SELECT` 默认查所有行，返回被选中的字段
- 可通过别名、去重、表达式增强结果控制
- `WHERE` 用于加条件，支持多种比较与逻辑运算符
- 排序用 `ORDER BY`，支持多级排序、ASC/DESC
- 聚合函数用于列的统计分析，如 `SUM`、`COUNT`
- `GROUP BY` 结合聚合函数对数据分组，`HAVING` 用于分组后过滤