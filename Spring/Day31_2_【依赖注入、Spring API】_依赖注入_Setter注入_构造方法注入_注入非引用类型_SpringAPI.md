# Day31 · Spring（依赖注入、Setter注入、构造方法注入、注入非引用类型、Spring API）

## 1. 依赖注入（DI）

依赖注入（Dependency Injection）是 Spring 实现 IoC（控制反转）的核心机制，由容器统一负责管理和注入 Bean 所需的依赖，其支持注入的依赖类型包括：

- **引用类型**：由容器管理的其他 Bean
- **值类型**：如 `int`、`String` 等基本类型
- **集合类型**：如 `List`、`Map` 等结构化依赖

> Spring 只能注入容器管理的对象，业务代码应通过容器提供的 `getBean()` 方法获取 Bean。若使用 `new` 创建实例，将无法触发依赖注入，易导致空指针异常。

---

## 2. Setter 注入

Spring 在实例化 Bean 后，会自动调用其公开的 `setXxx()` 方法注入依赖对象。每个 setter 方法必须符合 JavaBean 规范：

- **方法名以 `set` 开头，后跟属性名（驼峰命名）**
- **方法必须是 `public`，且仅接收一个参数**
- **每个依赖项需提供一个独立的 setter 方法**

```java
public class UserServiceImpl implements UserService {
    private UserDao userDao;  // Bean 依赖
    public void setUserDao(UserDao userDao) {  // 供容器自动调用的 setter 方法
        this.userDao = userDao;
    }
}
```

---

#### `<property>` 标签（推荐）

`property` 标签用于在 Bean 内部配置依赖，必须写在 `<bean>` 标签内；**每个依赖对应一个 `<property>` 标签，多个依赖需声明多个标签。**

```xml
<bean id="userService" class="org.example.UserServiceImpl">
    <property name="userDao" ref="userDao"/>	<!-- 注入依赖 Bean -->
</bean>
```

- **`name`：**对应 setter 方法名（去掉 `set` 并首字母小写）
- **`ref`：**要注入的 Bean 的 ID

---

#### `p` 命名空间（了解即可）

`p` 命名空间用于简化依赖注入语法，**需在 `<beans>` 标签中引入命名空间**。

它将原本的 `<property>` 标签转换为 `<bean>` 标签上的属性形式，本质仍是调用 setter 方法。适用于注入项较少的场景。

```xml
<beans ...
       xmlns:p="http://www.springframework.org/schema/p">	<!-- 引入 p 命名空间 -->
    <bean id="userService" class="com.example.UserServiceImpl" p:xxx-ref="beanId"/>
</beans>
```

- **`xxx`：**对应setter方法名（去掉 `set` 并首字母小写）
- **`beanId`：**要注入的 Bean 的 ID

---

## 3. 构造方法注入

Spring 支持通过构造方法注入依赖。容器会**根据 `<constructor-arg>` 标签提供的参数信息选择匹配的构造方法进行实例化**。

```java
public class UserServiceImpl implements UserService {
    private UserDao userDao;  // 注入的依赖 Bean
    private User user;        // 注入的依赖 Bean
    public UserServiceImpl(UserDao userDao, User user) {  // 容器调用的有参构造方法
        this.userDao = userDao;
        this.user = user;
    }
}
```

---

#### `<constructor-arg>` 标签

```xml
<bean id="userServiceConstructor" class="org.example.service.impl.UserServiceImpl">
    <constructor-arg name="userDao" ref="userDao"/>  <!-- 按参数名注入 userDao Bean -->
    <constructor-arg index="1" ref="user"/>          <!-- 按参数位置注入 user Bean -->
</bean>
```

- **`index`**：指定构造方法参数位置，从 0 开始（推荐）
- **`name`**：指定参数名，需配合 `-parameters` 编译参数使用（不推荐）
- **`ref`**：注入的 Bean ID（引用类型）
- **`value`**：注入的基本类型值，Spring 会自动类型转换

> 建议优先使用 `index` 标注参数顺序，避免因参数名缺失或推断失败导致注入错误。

---

## 4. 基本类型 & 集合类型注入

除了注入 Bean（引用类型），Spring 还支持通过 `value` 注入基本类型，通过 `<list>`、`<map>` 等标签注入集合类型。这些类型与引用类型一样，均可通过 setter 方法或构造方法完成注入。

### 4.1 基本类型（值）注入（`value`）

可通过 `value` 属性注入基本类型值。Spring 容器会根据参数名称或位置，调用对应的 setter 方法或构造方法完成注入。

```xml
<!-- setter 方法注入基本类型值 -->
<property name="userName" value="小王"/>

<!-- 构造方法注入基本类型值 -->
<constructor-arg name="serverId" value="123456"/>
```

> Spring 会根据目标字段或构造方法参数的类型，自动完成 `value` 的类型转换。

### 4.2 集合类型注入（`<List>` / `<Map>` / `<Set>` / `<Props>`）

可通过 `<list>`、`<set>`、`<map>`、`<props>` 等**集合标签及其子标签**注入集合类型。Spring 容器根据参数名称或位置，调用对应的 setter 方法或构造方法完成注入。

下面示例展示了将 `Map<String, User>` 注入 Bean 的两种方式：

```xml
<bean id="userPojo" class="org.example.pojo.User">
    <property name="userAge" value="55"/>
    <property name="userName" value="张三"/>
</bean>

<!-- setter 方法注入 Map<String, User> -->
<bean id="userServicePropertyMap" class="org.example.UserServiceImpl">
    <property name="userMap">
        <map>
            <entry key="u1" value-ref="userPojo"/>
            <entry key="u2" value-ref="userPojo"/>
        </map>
    </property>
</bean>

<!-- 构造方法注入 Map<String, User> -->
<bean id="userServiceConstructorMap" class="org.example.UserServiceImpl">
    <constructor-arg index="0">
        <map>
            <entry key="u1" value-ref="userPojo"/>
            <entry key="u2" value-ref="userPojo"/>
        </map>
    </constructor-arg>
</bean>
```

> 集合中的引用类型元素必须在配置中先注册为 Bean。

---

## 5. 配置文件引入（`<import>`）

Spring 支持通过 `<import>` 标签在主配置文件中引入其他配置文件，实现模块化配置管理。常用于大型项目中将 DAO、Service、Controller 等按层拆分配置，便于维护与复用。

```xml
<beans>
    <import resource="classpath:applicationContext-dao.xml"/>
    <import resource="classpath:applicationContext-service.xml"/>
</beans>
```

> Spring 只需加载主配置文件，所有被引入子配置将自动加载。

---

## 6. Spring API

### 6.1 容器接口：`ApplicationContext`

`ApplicationContext` 是 Spring 的核心容器接口，负责加载配置、创建 Bean、注入依赖、管理生命周期等。

| 实现类                                   | 说明                         |
| ---------------------------------------- | ---------------------------- |
| **`ClassPathXmlApplicationContext`**     | 从类路径加载 XML 配置        |
| **`FileSystemXmlApplicationContext`**    | 从磁盘路径加载 XML 配置      |
| **`AnnotationConfigApplicationContext`** | 加载 Java 配置类（基于注解） |

---

### 6.2 获取 Bean：`getBean()`

`getBean()` 方法用于从容器中获取已注册的 Bean 实例。

| 方法签名                                                | 说明                                                         |
| ------------------------------------------------------- | ------------------------------------------------------------ |
| **`Object getBean(String name)`**                       | 按 Bean ID 获取实例（需强转），适用注册了多个同类 Bean 的情况 |
| **`<T> T getBean(Class<T> requiredType)`**              | 按类型获取唯一 Bean                                          |
| **`<T> T getBean(String name, Class<T> requiredType)`** | 按 ID + 类型联合获取 Bean，适用于精确控制                    |

> 建议使用接口类型接收 Bean，避免实现类耦合，保持良好的可替换性与扩展性。

---

## ✅ 总结

- Spring 通过依赖注入（DI）实现对象解耦，由容器统一负责依赖管理
- 推荐使用 `<property>` 和 `<constructor-arg>` 配置依赖关系
- 注入方式包括 **Setter 注入、构造注入**。**支持 Bean、值类型、集合类型**
- 支持通过 `<import>` 标签引入多个配置文件，实现模块化组织
- `ApplicationContext` 是核心容器，使用 `getBean()` 获取对象，建议接口接收

