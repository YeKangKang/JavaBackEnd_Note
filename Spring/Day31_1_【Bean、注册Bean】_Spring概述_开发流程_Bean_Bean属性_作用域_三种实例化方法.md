# Day31 · Spring（Spring概述、开发流程、Bean、Bean属性、作用域、三种实例化方法）

## 1. Spring 概述

Spring 的本质是 **Bean 管理容器**，其解耦了各模块之间的依赖关系，将应用中的各类功能模块（如业务类、工具类、适配器等）抽象为 `Bean`，交由容器统一管理。

框架核心机制包括：

- **`IoC`（控制反转）**：容器负责对象注册、创建与依赖注入
- **`AOP`（面向切面编程）**：抽离事务、日志等横切逻辑

---

## 2. Spring 开发流程

```text
引入依赖（如 spring-context）
				↓
编写接口与实现类（业务逻辑模块）
				↓
配置 Spring 核心文件（如 applicationContext.xml）
				↓
注册 Bean（XML / 注解 / JavaConfig）
				↓
通过 Spring 容器获取 Bean
```

---

## 3. Bean 概述

`Bean` 是**由 Spring 容器创建、管理和销毁的对象**，通常代表一个具体功能模块。声明为 Bean 后，其生命周期和依赖关系将由容器统一接管。

> `Bean` 可通过 `<bean>` 标签或注解（如 `@Component`）声明，容器自动完成依赖注入与管理。

---

## 4. `<bean>` 标签 & 属性

```xml
<bean id="userService" class="com.example.UserService"/>  <!-- 默认无参构造方式创建 Bean -->
```

| 属性名           | 说明 |
|------------------|------|
| **`id`**（必填）   | Bean 在容器中的唯一标识 |
| **`class`**（必填） | Bean 的全限定类名 |
| **`scope`**      | Bean 的作用域，生命周期 |
| **`init-method`** | 初始化方法名，在构造与依赖注入完成后调用 |
| **`destroy-method`** | 销毁方法名，容器销毁 Bean 前调用（不适用于 prototype） |
| **`factory-method`** | 创建 Bean 的方法名（静态方法 / 实例方法） |
| **`factory-bean`** | 工厂 Bean 的 ID，配合 `factory-method` 调用实例方法 |

> `init-method` 适用于所有作用域；`destroy-method` 仅对容器负责销毁的 Bean 生效，`prototype` 类型需手动销毁。

---

## 5. Bean 生命周期 & 作用域（`scope`）

Spring 通过 `scope` 属性控制 Bean 的生命周期与作用域：

| 作用域                   | 生命周期                                                     |
| ------------------------ | ------------------------------------------------------------ |
| **`singleton`**（默认）  | 容器创建时实例化，容器销毁时销毁；整个容器共享一个实例       |
| **`prototype`**          | 每次调用 `getBean()` 时新建实例，Spring 不管理销毁；GC 负责销毁 |
| **`request`**（Web）     | 每个 HTTP 请求对应一个 Bean 实例，请求结束时销毁             |
| **`session`**（Web）     | 每个用户 Session 拥有一个 Bean 实例，Session 失效时销毁      |
| **`application`**（Web） | 整个 ServletContext 中一个实例，Web 应用关闭时销毁           |

---

## 6. Bean 的三种实例化方式（XML）

通过 XML 注册并实例化 Bean 主要有三种方式。常见做法是直接使用无参构造方法，或使用 `constructor-arg` 传入构造参数；仅当类无法直接实例化时，才采用静态或实例工厂方法。

| 实例化方式        | 配置示例                             | 场景说明 |
|-------------------|--------------------------------------|----------|
| 无参构造          | `<bean class="Xxx"/>`                | 默认方式，推荐使用 |
| 静态工厂方法      | `<bean class="Xxx" factory-method="create"/>` | 类不可直接构造时使用 |
| 实例工厂方法      | `factory-bean + factory-method`      | 调用已注册工厂 Bean 的实例方法 |

#### 1. 默认构造方法

```xml
<bean id="userService" class="com.example.UserService"/>
```

#### 2. 静态工厂方法

```xml
<bean id="userService"
      class="com.example.StaticFactory"
      factory-method="getInstance"/> <!-- 通过静态方法实例化 Bean -->
```

#### 3. 实例工厂方法

```xml
<bean id="userFactory" class="com.example.ServiceFactory"/> <!-- 注册工厂 Bean -->
<bean id="userService"
      factory-bean="userFactory"
      factory-method="createUser"/> <!-- 通过工厂 Bean 的实例方法创建 -->
```

> 实现类注册为 Bean 时，建议**业务方通过接口类型接收 Bean，以避免类型绑定**，实现真正的解耦与面向接口编程。

---

## ✅ 总结

- Spring 是基于 IoC 和 AOP 的轻量级容器，核心作用是管理 Bean 的创建、依赖注入与生命周期
- `<bean>` 标签用于注册 Bean，可通过属性控制构造方式、作用域和初始化/销毁方法
- `scope` 决定 Bean 的生命周期与共享范围，常见有 `singleton`、`prototype`、`request` 等
- Bean 实例化方式包括：
  - 默认构造（推荐，配合 `constructor-arg` 可传参）
  - 静态工厂方法（用于构造器受限场景）
  - 实例工厂方法（通过注册工厂 Bean 调用实例方法）
- 注册实现类为 Bean 时，建议通过接口类型引用以实现解耦与面向接口编程
