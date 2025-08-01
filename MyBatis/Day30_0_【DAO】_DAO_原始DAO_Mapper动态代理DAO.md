# Day30 · MyBatis（DAO、原始DAO、Mapper动态代理DAO）

## 1. 什么是 DAO？

DAO（Data Access Object）是一种数据库访问层设计模式，用于封装所有数据库访问逻辑与组件，其主要目标是**解耦业务逻辑与持久化操作**，提供更清晰、更可维护的结构。DAO 层的优势包括：

- **接口隔离**：仅向上层暴露必要的数据库访问接口
- **可维护性高**：SQL 改动不会影响业务逻辑代码
- **便于测试与重构**：数据库操作抽象为独立模块，便于替换实现

> DAO 通常操作 JavaBean（POJO 实体类），通过接口 + SQL 映射文件配合完成数据库交互。

---

## 2. MyBatis 中的 DAO 模式

MyBatis 提供两种主要的 DAO 实现方式，二者本质均围绕 **实体类 + Mapper 接口 + 映射文件** 三者协同构建。

| 模式               | 实现方式                      | 特点                        |
| ------------------ | ----------------------------- | --------------------------- |
| **原始 DAO**       | 手写接口实现类 + 映射文件     | 灵活，可扩展，冗余较多      |
| **Mapper 动态代理 DAO** | 编写接口，MyBatis 自动代理实现 | 推荐，简洁高效，开发主流方式 |

> 实际开发中几乎统一采用 **Mapper 动态代理方式**，仅在特殊场景下才保留原始 DAO。

---

## 3. 原始 DAO 实现方式

原始 DAO 方式通过手动实现接口、调用 `SqlSession` 映射文件来完成数据库操作。

```java
public class UserDaoImpl implements UserDao {
    private SqlSessionFactory sqlSessionFactory;

    public UserDaoImpl(SqlSessionFactory factory) {
        this.sqlSessionFactory = factory;
    }

    @Override
    public List<User> findAll() {
        try (SqlSession session = sqlSessionFactory.openSession()) {
            // 调用映射文件中的 select 语句
            return session.selectList("UserMapper.findAll");
        }
    }
}
```

- `SqlSessionFactory` 用于创建会话对象
- 每次方法调用需打开会话、执行操作、关闭资源
- 推荐使用 try-with-resources 保证资源关闭

> ⚠️ `SqlSession` 是线程不安全的，每次调用必须重新创建并及时关闭。

---

## 4. Mapper 动态代理 DAO（推荐）

MyBatis 支持根据接口和 SQL 映射文件**自动生成接口实现类（即 DAO 对象）**，开发者只需编写 Mapper 接口与映射文件，无需关心实现类创建。

```java
// UserMapper.java
public interface UserMapper {
    List<User> findAll();
}
```

```xml
<!-- UserMapper.xml -->
<mapper namespace="org.example.mapper.UserMapper">  <!-- 关联的 Mapper 接口(全限定名) -->
  <select id="findAll" resultType="User">
    SELECT * FROM user
  </select>
</mapper>
```

```java
// 获取 Mapper 代理对象
SqlSession session = sqlSessionFactory.openSession();
UserMapper mapper = session.getMapper(UserMapper.class);
List<User> users = mapper.findAll();
```

- MyBatis 自动为接口生成代理实现类
- 方法与 SQL 映射文件中的语句一一对应
- 更符合现代分层架构思想，推荐使用

> 相比原始 DAO，Mapper 动态代理 **无需手写实现类**，大幅提升开发效率与代码可读性。

---

## ✅ 总结

- DAO 是持久化访问的核心模式，MyBatis 提供原始 DAO 与动态代理两种实现
- 实际开发推荐统一采用 Mapper 动态代理方式，结构更简洁、扩展性更强
- 所有数据库交互都应通过 DAO 层进行封装，避免业务逻辑直接操作数据库
