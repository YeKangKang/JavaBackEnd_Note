# Day32 · Spring（第三方数据源、注解概述、组件扫描、原始注解、新注解、Spring整合JUnit）

## 1. 数据源（DataSource）

数据源（连接池）是对数据库连接的统一抽象，通常在初始化阶段预创建若干连接对象供业务使用，使用结束后再归还，以提升系统性能。

- 通过连接池管理，程序**无需为每次请求反复创建和销毁连接，而是复用已有连接资源。**
- 常见实现包括：`DBCP`、`C3P0`、`HikariCP`、`Druid` 等。

> **第三方实现的数据源**可以通过 XML 或注解注册到 Spring 容器中，作为可注入的 Bean 提供给业务层使用。

#### 例：C3P0 数据源初始化

```java
ComboPooledDataSource dataSource = new ComboPooledDataSource();
dataSource.setDriverClass("com.mysql.cj.jdbc.Driver");
dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test_JDBC_database");
dataSource.setUser("root");
dataSource.setPassword("597474789pP");
dataSource.setInitialPoolSize(2);
dataSource.setMaxPoolSize(10);

Connection connection = dataSource.getConnection();
System.out.println(connection);
connection.close();
```

---

### 1.1 第三方数据源注册（XML）

Spring 本身不提供连接池实现，但**支持将第三方数据源（如 C3P0、Druid）注册为 Bean**，由容器管理其生命周期，并通过直接赋值或依赖注入方式配置连接参数。

#### 方式一：硬编码注入

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql://127.0.0.1:3306/test"/>
    <property name="user" value="root"/>
    <property name="password" value="597474789pP"/>
</bean>
```

> 将数据源注册为 Bean，并通过 `setter` 注入连接参数。此方式仍存在**值硬编码问题**，配置变更需修改 XML。

#### 方式二：外部配置注入

为实现数据库连接参数的配置解耦，可将配置信息写入 `.properties` 文件，并通过 Spring 容器统一注入。该方式**依赖 `context` 命名空间**，需在 XML 文件头部引入对应声明。

```xml
<beans xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/context 
                           http://www.springframework.org/schema/context/spring-context.xsd"
       ...>

    <!-- 加载外部 jdbc.properties 文件 -->
    <context:property-placeholder location="classpath:jdbc.properties"/>

    <!-- 注册第三方数据源并注入配置参数 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driverClass}"/>
        <property name="jdbcUrl" value="${jdbc.jdbcUrl}"/>
        <property name="user" value="${jdbc.user}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

- **`<context:property-placeholder>`：**加载指定路径的 `.properties` 配置文件
- **`${key}`：**占位符，用于注入配置文件中对应 `key=value` 的值

---

## 2. 注解开发

Spring 注解开发用于代替繁琐的 XML 配置，提升开发效率。按功能可分为两类：

- **原始注解**：替代 `<bean>` 标签及其属性，主要负责自定义类的注册和依赖注入
- **新注解**：负责容器结构搭建、资源加载、第三方对象注册

---

## 3. 组件扫描

组件扫描（Component Scanning）是指 Spring 容器在启动时，**自动扫描指定包及其子包中的注解类，并将其注册为 Bean** 的过程。要启用注解功能，必须通过 **XML 配置或注解方式开启组件扫描**，否则 Spring 容器无法识别相关注解。

##### XML 配置方式：

```xml
<context:component-scan base-package="com.example"/>
```

##### 注解配置方式（基于配置类）：

```java
@ComponentScan("com.example")
```

---

## 4. 原始注解

原始注解用于替代 `<bean>` 标签，控制类的注册、作用域和依赖注入。

| 注解          | 说明 |
|---------------|------|
| **`@Component`** | 注册 Bean（默认 ID 为类名首字母小写） |
| **`@Controller` `@Service` `@Repository`** | 含义化版本的 `@Component`，用于分层 |
| **`@Scope`**  | 指定 Bean 作用域 |
| **`@Autowired`** | 按类型自动注入**（字段 / setter / 构造方法 / 普通方法）** |
| **`@Qualifier`** | 搭配 `@Autowired`，按名称精确注入 |
| **`@Resource`** | 按名称注入，找不到时按类型匹配 |
| **`@Value`**  | 注入值（常配合 `${}` 使用） |
| **`@PostConstruct`** | 初始化方法（Bean 创建后调用） |
| **`@PreDestroy`** | 销毁方法（Bean 销毁前调用） |

```java
@Service // 注册为业务层 Bean
@Scope("singleton") // 设置作用域（单例）
public class UserService {

    @Autowired // 按类型注入
    @Qualifier("userDao") // 按名称精确匹配
    private UserDao userDao;

    @Resource(name = "userDao") // 按名称注入（找不到时按类型）
    private UserDao userDao1;

    @Value("${jdbc.driverClass}") // 注入配置文件中的值
    private String driverClass;

    @PostConstruct // 初始化方法
    public void init() {
        System.out.println("初始化：" + driverClass);
    }

    @PreDestroy // 销毁方法
    public void destroy() {
        System.out.println("销毁 UserService");
    }
}
```

> Spring 支持将依赖注入到<u>**字段**</u>、<u>**构造方法参数**</u>、<u>**setter 参数**</u>或<u>**任意方法参数**</u>中。所有方式都可**搭配 `@Autowired` 和 `@Qualifier` 实现对每一个参数或字段的精确注入**，使用上非常灵活。

> 字段注入通过反射实现，不依赖 setter 方法

---

## 5. 新注解（配置类相关）

配置类相关注解用于**完全替代 XML 配置文件**，以实现 Spring 容器的结构化搭建、Bean 注册与资源加载。

对于**无法在源码中添加注解的第三方类（如数据源）**，可通过配置类中的 `@Bean` 方法进行手动构造与初始化，并注册到 Spring 容器中。

| 注解                  | 功能说明                                                  |
| --------------------- | --------------------------------------------------------- |
| **`@Configuration`**  | 声明当前类为 Spring 配置类                                |
| **`@ComponentScan`**  | 配置组件扫描路径，自动注册指定包下的注解类为 Bean         |
| **`@PropertySource`** | 加载 `.properties` 配置文件，供 `@Value` 使用             |
| **`@Import`**         | 导入其他配置类，实现多模块组合初始化                      |
| **`@Bean`**           | 注册方法返回值为 Bean，默认以方法名首字母小写为 Bean 名称 |

```java
@Configuration // 声明配置类
@ComponentScan({"org.example"}) // 配置组件扫描路径（一个或多个）
@PropertySource("classpath:app.properties") // 加载配置文件
@Import({DataSourceConfig.class, ServiceConfig.class}) // 导入配置类
public class AppConfig {
    @Value("${app.name}") // 注入属性值
    private String appName;

    @Bean("customBanner") // 注册返回值为 Bean
    public String banner() {
        return "启动应用：" + appName;
    }
}
```

> 配置类相关注解通过**将初始化模块拆分为独立配置类，并由 `@Import` 汇总至主配置类**，结合组件扫描与依赖注入，完成对 Spring 容器的结构化、初始化操作。

---

## 6. Spring 整合 JUnit

传统测试中需手动创建 `ApplicationContext` 并显式获取 Bean，过程重复繁琐、耦合度高。

通过整合 JUnit，可**由 Spring 接管测试执行与依赖注入，自动加载容器并注入待测 Bean**，使测试更简洁、模块更独立。

| 注解                           | 功能说明                                                     |
| ------------------------------ | ------------------------------------------------------------ |
| **`@RunWith`**                 | 指定使用 Spring 提供的测试运行器（如 `SpringJUnit4ClassRunner`） |
| **`@ContextConfiguration`**    | 指定容器初始化所需的配置文件或配置类                         |
| **`@Autowired`**               | 自动注入需要测试的 Bean                                      |
| **`@Qualifier` / `@Resource`** | 精确指定注入 Bean 的名称，避免类型冲突                       |

```java
@RunWith(SpringJUnit4ClassRunner.class) // 启用 Spring 测试运行器
@ContextConfiguration(classes = SpringConfiguration.class) // 加载配置类
public class TestFile {
    @Autowired
    @Qualifier("dataSource") // 精确注入目标 Bean
    private DataSource dataSource;

    @Test	// 执行测试
    public void testFunction() throws SQLException {
        System.out.println(dataSource.getConnection());
    }
}
```

## ✅ 总结

- **数据源** 是数据库连接池的抽象，常通过第三方库（如 C3P0、Druid）实现，可通过 XML 或注解注册为 Spring Bean。
- **注解开发** 替代繁琐 XML，分为原始注解（负责类注册与依赖注入）和配置类相关注解（负责容器结构搭建与资源加载）。
- **组件扫描** 是注解生效前提，需显式开启，支持 XML 或 `@ComponentScan`。
- **原始注解** 控制 Bean 的注册、作用域与注入方式，推荐使用构造函数或 setter 方式增强可维护性。
- **配置类体系** 实现对容器结构的模块化构建，支持加载外部资源、注册第三方对象、导入其他配置类，完全替代 XML。
- **整合 JUnit** 可让 Spring 自动加载容器并注入依赖，提升测试代码的简洁性与可维护性，推荐使用配置类方式构建测试环境。
