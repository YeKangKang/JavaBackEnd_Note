# Day30 · MyBatis（框架、软件分层、MyBatis概述、开发流程）

## 1. 什么是框架？

b框架是一套被预先设计好的、可重用的代码结构。它帮助开发者**封装底层细节**；**实现代码的松耦合**；**解决重复“造轮子”的问题**。

---

## 2. 软件分层

软件分层是一种工程化设计思想。通过将系统按**功能职责进行拆分**，形成结构清晰、职责分明的层级体系，提升系统的可维护性、可扩展性与复用性。

- **高内聚：**每一层**只负责自己该做的事情**，职责单一、逻辑集中
- **低耦合：**各层通过**接口进行交互**，相互独立，便于解耦与替换

| 层级       | 类型                 | 责任                         |
| ---------- | -------------------- | ---------------------------- |
| **表现层** | 接收请求，调用业务，响应数据 | `Controller` / `Web` |
| **业务层** | 校验、逻辑处理、事务管理     | `Service`            |
| **持久层** | 执行 SQL，与数据库交互       | `Dao` / `Mapper`     |

---

## 3. MyBatis 概述

MyBatis 是一个**持久层框架**，对 JDBC 进行了封装。帮助开发者解决**代码重复、硬编码、管理结果集等繁琐操作。**

- 支持 XML 或注解写 SQL
- 可自定义参数映射
- 自动将结果集映射为 Java 实体

---

## 4. MyBatis 开发流程（基于 Mapper 代理的 DAO 模式）

```
引入依赖
 ↓
配置 mybatis-config.xml（注册数据源，配置类型别名包，注册 Mapper 接口 / XML映射文件）
 ↓
编写实体类
 ↓
编写 Mapper 接口
 ↓
编写 Mapper.xml 映射文件
 ↓
实现业务逻辑（读取配置，获取 SqlSession ，调用 Mapper ，返回结果）
```

## 5. 具体流程

### 5.1 引入依赖

- **Jar 包**：将 MyBatis jar 包放入 classpath
- **Maven项目**：在 `pom.xml` 中定义依赖

---

### 5.2 核心配置文件（`mybatis-config.xml`）

MyBatis 的主配置文件，提供了框架运行所需的核心信息。主要包含：

- **类型别名配置 `<typeAliases>`**：用于<u>简化 Java 类型的全限定名</u>，在映射文件中可以直接使用类名或别名。

```xml
<typeAliases>
  <!-- 指定别名 -->
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
  <!-- 扫描整个包（默认使用类名为别名，大小写不敏感） -->
  <package name="domain.blog"/>
</typeAliases>
```

- **环境配置 `<environments>`**：用于<u>定义多个数据库连接环境（如 dev、prod）</u>，通过 `default` 属性切换当前激活环境。

```xml
<environments default="dev">
  <!-- 环境 1 -->
  <environment id="dev">
    <transactionManager type="JDBC"/>
    <dataSource type="POOLED">
      <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
      <property name="url" value="jdbc:mysql://localhost:3306/dev_db"/>
      <property name="username" value="root"/>
      <property name="password" value="123456"/>
    </dataSource>
  </environment>
  <!-- 环境 2 -->
  <environment id="prod">...</environment>
</environments>
```

- **映射器配置 `<mappers>`**：用于<u>注册 Mapper 接口或 XML 映射文件</u>。MyBatis 启动时会加载这些信息以完成 SQL 映射绑定。

```xml
<mappers>
  <!-- 方法1：仅注册 XML 文件（不使用接口，直接通过 namespace.id 执行 SQL） -->
  <mapper resource="org/example/mapper/UserMapper.xml"/>

  <!-- 方法2：注册 Mapper 接口（推荐，自动绑定同名 XML，实现 DAO 模式） -->
  <mapper class="org.example.mapper.UserMapper"/>

  <!-- 方法3：包扫描注册接口（批量注册，要求接口与 XML 文件同名同路径） -->
  <package name="org.example.mapper"/>
</mappers>
```

> **配置项声明顺序敏感**，建议按照官方推荐顺序书写各配置项。

---

### 5.3 实体类（Entity）

**实体类用于表示数据库表中的一行数据**，类中的每个字段对应表中的一列，且对于每一个字段都带有 `getter/setter`。MyBatis 会自动根据查询结果，将其**映射为一个或多个该实体类对象**。

> **POJO（Plain Old Java Object）：** 普通 Java 对象，不依赖框架、不继承特殊接口，常用于封装数据。

---

### 5.4 接口代理（Mapper 接口类）

接口代理用于定义与数据库操作对应的方法，MyBatis 会通过代理机制自动生成其实现类，开发者无需手动编写。

- **接口命名规范**
  - **接口名** = **映射文件名**
  
  - **接口方法名** = 映射文件中对应 SQL 的 **`id`**
- **绑定规范**
  - 接口与映射文件在同一路径下
  - 映射文件的 `namespace` = 接口类的全限定类名
  - `mybatis-config.xml` 中注册所有映射关系**（通过 `class` 手动注册 / `package` 包扫描）**

> ⚠️ 以上所有规范缺一不可，否则接口代理将无法成功注册。

---

### 5.5 Mapper 映射文件（`mapper.xml`）

映射文件内包含自定义的 SQL 语句。SQL 语句可通过 **`SqlSession`** 或 **Mapper 接口代理**调用，调用结果将自动封装为指定实体类对象（由 `resultType` 指定）。

- **SqlSession 调用：** `namespace.id` 仅作为逻辑路径标识，无需接口参与。
- **Mapper 代理调用：** `namespace` 表示归属接口类（全限定类名），`id` 对应接口方法名，完成动态绑定。

---

### 5.6 启动与业务调用

将前述配置与组件整合后，即可开始进行数据库操作。调用流程包括：

1. 加载配置文件（`mybatis-config.xml`）
2. 构建 `SqlSessionFactory`
3. 获取 `SqlSession` 实例（⚠️ 每个线程或每次操作都应使用独立实例，**`SqlSession` 本身线程不安全**）
4. 执行 SQL（推荐使用 Mapper 接口代理）
5. 关闭资源

```java
// 加载 MyBatis 配置文件
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);

// 构建 SqlSessionFactory
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

// 获取 SqlSession
SqlSession sqlSession = sqlSessionFactory.openSession();

// SqlSession 调用（不推荐）
List<User> users = sqlSession.selectList("UserMapper.selectAll");
// Mapper 接口代理调用（推荐）
UserMapper mapper = sqlSession.getMapper(UserMapper.class);
List<User> usersFromProxy = mapper.selectAll();

sqlSession.close();
```

> 推荐使用 Mapper 接口代理方式进行开发，结构更清晰、类型安全性更高

---

## ✅ 总结

- **MyBatis 是典型的持久层框架**，通过封装 JDBC、支持 XML 与注解配置，简化了数据库访问开发。
- **开发流程清晰有序**：依赖引入 → 配置文件 → 实体类 → 接口 + 映射 XML → 业务调用。
- MyBatis **核心配置：**
  - `<environments>`：配置数据源与事务管理器
  - `<mappers>`：注册接口与 SQL 映射关系
  - `<typeAliases>`：简化类型引用
- **实体类用于封装表中一行数据**，必须具备字段、getter/setter、无参构造。
- **Mapper 接口 + XML 绑定必须满足三要素**：路径一致、namespace 匹配、配置注册。
- **推荐使用接口代理方式**操作数据库，更简洁、安全、结构清晰。