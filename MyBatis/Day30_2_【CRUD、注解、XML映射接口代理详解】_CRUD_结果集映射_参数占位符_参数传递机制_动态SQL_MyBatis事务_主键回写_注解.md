# Day30 · MyBatis（CRUD、结果集映射、参数占位符、参数传递机制、动态SQL、MyBatis事务、主键回写、注解）

## 1. CRUD 操作

基于 DAO 模式的 MyBatis 开发，本质上围绕 **“实体类 + 接口方法 + 映射文件”** 三个核心构件展开，实现业务逻辑与 SQL 执行的解耦。

- 编写实体类（如 User.java）
- 定义 Mapper 接口方法（无需实现）
- 编写/修改 XML 映射配置文件（SQL 映射）

> **本节以 DAO（Mapper 代理）方式为基础展开**，因其结构正规、安全、解耦性强，是实际开发的主流方式。

---

## 2. SQL 映射文件中的 CRUD 标签

| 标签名         | 用途 | 常见属性                                                     |
| -------------- | ---- | ------------------------------------------------------------ |
| **`<select>`** | 查询 | `id`、`resultType`/`resultMap`、`parameterType`（附带知识点：[结果集映射](#3. 结果集映射（`resultMap` / `resultType`）)） |
| **`<insert>`** | 插入 | `id`、`useGeneratedKeys`、`keyProperty`（附带知识点：[主键回写](#8. 主键回写（用于 `INSERT`）)） |
| **`<update>`** | 更新 | `id`、`parameterType`（附带知识点：[`<set>`标签](#`<set>`：拼接 UPDATE 中的 SET 子句)） |
| **`<delete>`** | 删除 | `id`、`parameterType`                                        |

---

## 3. 结果集映射（`resultMap` / `resultType`）

**MyBatis 会将查询结果封装为 `resultType` 指定的类型**。若为 JavaBean 类型，则按列名与属性名的对应关系完成字段映射。

#### `resultMap` 手动映射

当数据库字段名与 Java 对象属性名不一致（如 `user_id` 与 `userId`）时，MyBatis 默认映射将失败，字段值将为 `null`，需显式指定映射关系。

```xml
<resultMap id="userMap" type="User">				<!-- 指定映射结果的目标类 -->
  <id column="user_id" property="userId"/>   				<!-- 主键字段映射（<id> 标签） -->
  <result column="username" property="userName"/>		<!-- 普通字段映射（ <result> 标签） -->
</resultMap>

<!-- 使用 resultMap 进行结果集映射（替代 resultType） -->
<select id="findAll" resultMap="userMap">...</select>
```

#### 启用驼峰自动映射（在核心配置文件中）

```xml
<settings>
  <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

---

## 4. 参数占位符（`#{}` / `${}`）

|           | 名称             | 安全性                    | 行为说明                             |
| --------- | ---------------- | ------------------------- | ------------------------------------ |
| **`#{}`** | **预编译占位符** | 安全，防止 SQL 注入       | 在 SQL 中预编译为 `?`，**自动添加引号** |
| **`${}`** | **文本替换标记** | 不安全，可能发生 SQL 注入 | 将值直接拼接入 SQL，无任何处理      |

> 实际开发中，除非用于**拼接表名、列名等内部逻辑才使用 `${}`**，一切参数传入场景均**推荐统一使用 `#{}`**。

---

## 5. 参数传递机制

MyBatis 会将 **接口方法参数统一封装为 `Map<String, Object>`**，在 SQL 映射中通过不同的 key 名规则进行访问。

| 参数类型          | 映射文件访问方式        | 说明                          |
| ----------------- | ----------------------- | ----------------------------- |
| **不添加注解**    | `#{param1}` / `#{arg0}` | 默认顺序命名                  |
| **`@Param` 注解** | `#{注解名}`             | 语义清晰**（推荐）**          |
| **JavaBean 对象** | `#{字段名}`             | 类型安全，IDE友好**（推荐）** |
| **`Map` 类型**    | `#{key}`                | 直接以键访问                  |

> 推荐优先使用 **JavaBean** 或 **`@Param`** 作为传入参数，提高代码可维护性与类型安全。

---

### 5.1 补充说明

#### 嵌套对象访问

若传入的参数对象中某字段本身是一个对象（即“**值为对象**”），MyBatis 支持通过 **`#{字段.值对象属性}`** 或 **`${字段.值对象属性}`** 的方式访问其内部属性。

```xml
<select id="findAllCityByOrder" resultType="City">
  SELECT * FROM user
  WHERE city = #{address.city}           <!-- 绑定嵌套属性：值使用 #{ } -->
  ORDER BY ${sort.field} ${sort.order}   <!-- 拼接结构性语句：使用 ${ } -->
</select>
```

> 对于传入的 JavaBean 参数对象，MyBatis 支持通过连续访问嵌套对象的 `getter` 方法获取属性值，即使内部字段为 `private`，只要存在对应的 `getXxx()` 方法即可正常访问。

---

#### `Collection` / `List` / `Set` / `array` 作为参数

对于 `Collection` / `List` / `Set` / 数组这类参数，**除了可通过 `paramX` / `argX` 访问外**，MyBatis 底层还会自动为它们在参数 `Map` 中添加对应的额外键名：

- **集合类**（`Collection` / `List` / `Set`）
  - 添加通用 key：`collection`
  - 按类型额外添加 key：`list` 或 `set`
- **数组**（`T[]`）
  - 添加 key：`array`

>  这些 key 可用于 `<foreach>` 等标签访问，但**仍推荐使用 `@Param()` 明确命名**，避免因默认键名混淆导致的取值错误。

---

## 6. 动态 SQL 标签

MyBatis 提供一组动态 SQL 标签，用于根据不同参数构建出可变的 SQL 语句。典型场景包括条件查询、字段更新、批量操作等。

#### `<if>`：条件判断

用于按需拼接 SQL 片段，`test` 属性中写入逻辑表达式。

```xml
<if test="userName != null and userName != ''">
  user_name = #{userName}  <!-- 若 userName 有值，则拼接该条件 -->
</if>
```

> 表达式与 Java 基本一致，但 `&&` / `||` 需写为 `and` / `or`。

------

#### `<where>`：自动拼接 WHERE 子句

自动补全 `WHERE` 关键字，并去除首个条件的 `AND` / `OR`。

```xml
<where>
  <if test="userName != null">AND user_name = #{userName}</if>
  <if test="age != null">AND age = #{age}</if>
</where>
```

> 类似于 `WHERE 1=1` 的结构，但更智能，避免语法错误。

------

#### `<choose>`：条件分支（多选一）

模拟 `switch-case`，只选择第一个满足的分支。无满足条件时执行 `otherwise`。

```xml
<choose>
  <when test="userName != null">user_name = #{userName}</when>
  <when test="age != null">age = #{age}</when>
  <otherwise>1=1</otherwise>  <!-- 可配合 <where> 使用 -->
</choose>
```

------

#### `<set>`：拼接 UPDATE 中的 SET 子句

自动拼接字段赋值，避免逗号问题（多余逗号或字段顺序问题）。

```xml
<set>
  <if test="userName != null">user_name = #{userName},</if>
  <if test="age != null">age = #{age},</if>
</set>
```

------

#### `<foreach>`：批量操作

可用于遍历 **数组、集合、`Map` 等可迭代参数**，生成多个 SQL 片段，常用于构造 `IN (...)`、批量插入、批量删除等场景。

```xml
<foreach collection="array" item="item" open="(" separator="," close=")">
  #{item}  <!-- 每一项作为一个参数 -->
</foreach>
```

- `collection`：被遍历的参数名（建议使用 `@Param` 注解明确）
- `item`：当前迭代元素的变量名
- `open` / `close`：拼接前后的符号（如括号）
- `separator`：分隔符号（如 `,`  `OR`  `AND` 等）

---

## 7. MyBatis 事务

MyBatis 默认通过 `SqlSession` 执行数据库操作，其事务控制行为如下：

- **查询操作（`select`）**：默认 **自动提交**，不参与事务控制
- **修改操作（`insert` / `update` / `delete`）**：默认 **不自动提交**，需显式调用 `commit()` 手动提交事务

```java
SqlSession session = sqlSessionFactory.openSession(); // 手动提交模式（默认）
session.insert(...);
session.commit(); // 显式提交事务
```

**启用自动提交事务：**

```java
SqlSession session = sqlSessionFactory.openSession(true); // 自动提交模式
```

> `SqlSession` 的 **自动提交模式在创建时确定，实例创建后不可切换提交行为。**

---

## 8. 主键回写（用于 `INSERT`）

MyBatis 支持将数据库生成的主键值**自动回写**到传入对象中字段，常用于 `INSERT` 操作（例如后续作为外键使用等场景）。

#### `useGeneratedKeys`：适用于支持 JDBC 自增主键的数据库（推荐）

```xml
<insert id="addUser" useGeneratedKeys="true" keyProperty="id">
  INSERT INTO user (username, password) VALUES (#{username}, #{password})
</insert>
```

- **`useGeneratedKeys="true"`**：启用 JDBC 的主键自动回写机制
- **`keyProperty="id"`**：指定 JavaBean 中接收主键的字段名

> 要求数据库主键字段为 **自增类型**，且传入参数为 **JavaBean 对象**

#### `<selectKey>`：支持所有数据库的通用主键回写机制

```xml
<insert id="addUser">
  INSERT INTO user (username, password)
  VALUES (#{username}, #{password})
  <selectKey keyProperty="id" resultType="int" order="AFTER">
    SELECT LAST_INSERT_ID()
  </selectKey>
</insert>
```

- **`keyProperty="id"`**：指定 JavaBean 中接收主键的字段名
- **`resultType="int"`**：指定查询结果的类型

- **`order="AFTER"`**：指定 `selectKey` 在 `INSERT` 执行之后运行（常用于获取数据库生成的主键）；若为 `BEFORE`，则在插入前执行（适用于主键需预生成的场景）
- `LAST_INSERT_ID()`：MySQL 的函数，用于获取最近插入记录的自增主键

> `selectKey` 提供更明确的执行控制，可用于非自增主键或需要先生成主键再插入的场景。

---

## 9. 注解式开发（替代 XML 配置）

MyBatis 支持使用注解在 Mapper 接口中直接声明 SQL 语句，实现快速开发与配置精简。**注解绑定接口方法，MyBatis 根据方法返回值自动推断结果集映射**。

- 适用于**简单 SQL 语句**
- 不推荐处理复杂逻辑、动态 SQL（建议改用 XML）

| 注解名     | 说明                                         |
| ---------- | -------------------------------------------- |
| `@Select`  | 声明查询语句（对应 `<select>`）              |
| `@Insert`  | 声明插入语句（对应 `<insert>`）              |
| `@Update`  | 声明更新语句（对应 `<update>`）              |
| `@Delete`  | 声明删除语句（对应 `<delete>`）              |
| `@Param`   | 显式绑定参数名                               |
| `@Results` | 映射字段与属性关系（对应 `<resultMap>`）     |
| `@Result`  | 配置单个字段映射（对应 `<id>` / `<result>`） |

#### 例：使用注解进行查询与结果映射

```java
@Select("SELECT user_id, user_name, age FROM user WHERE age > #{minAge}")	// SELECT 注解
@Results({																													// 类似与 <resultMap>
    @Result(id = true, column = "user_id", property = "userId"),   	// 主键映射（id=true）
    @Result(column = "user_name", property = "userName")          	// 字段映射
    // 未声明字段（如 age）将尝试按默认规则映射
})
List<User> selectByMinAge(@Param("minAge") int minAge);		// Mapper 接口
```

## ✅ 总结

- [**CRUD 结构**](#1. CRUD 操作)：基于实体类、Mapper 接口、XML 映射文件三层解耦实现。
- [**结果映射**](#3. 结果集映射（`resultMap` / `resultType`）)：字段不一致用 `resultMap`；可开启驼峰自动映射。
- [**参数传递**](#5. 参数传递机制)：推荐使用 `@Param` 或 JavaBean；集合/数组支持 `<foreach>`。
- [**占位符区别**](#4. 参数占位符（`#{}` / `${}`）)：`#{}` 安全，`${}` 会拼接原文，慎用。
- [**动态 SQL**](#6. 动态 SQL 标签)：用 `<if>` / `<where>` / `<foreach>` 等标签拼接灵活 SQL。
- [**事务机制**](#7. MyBatis 事务)：查询默认自动提交，增删改需手动 `commit()`。
- [**主键回写**](#8. 主键回写（用于 `INSERT`）)：推荐 `useGeneratedKeys`，也可用 `<selectKey>`。
- [**注解开发**](#9. 注解式开发（替代 XML 配置）)：适用于简单 SQL，复杂逻辑建议用 XML。