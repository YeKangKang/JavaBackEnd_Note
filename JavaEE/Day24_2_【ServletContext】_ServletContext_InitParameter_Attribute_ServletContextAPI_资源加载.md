# Day24 · JavaEE（ServletContext、InitParameter、Attribute、ServletContextAPI、资源加载）

## 1. ServletContext 概述

`ServletContext` 对象面向整个服务器，服务于整个 Web 应用。它提供了服务器层面的上下文内容，作为全局共享对象存在，可以在不同的 Servlet 或其他 Web 组件中访问。**只要服务器启动，它就会存在。**

> **主要作用**：存取全局共享的数据，例如配置信息、路径设置及一些需要在 Servlet 中共享的数据。

---

## 2. InitParameter（初始化参数）

初始化参数存储在 `web.xml` 文件中，使用 `<context-param>` 标签声明，这些参数在 Web 容器启动时被初始化到 `ServletContext` 对象中。

- **共享参数：** `InitParameter` 对容器中所有成员可见，但主要用于 Web 容器的配置和参数设置。
- **不可修改（只读）：** 在初始化后，任何成员都无法修改 `InitParameter`。

> `InitParameter` 常用于存储配置信息，如数据库连接、路径设置等。

**示例：**

```xml
<context-param>
    <param-name>paramName</param-name>	<!-- InitParameter 名称（键） -->
    <param-value>paramValue</param-value>	<!-- InitParameter 值 -->
</context-param>
```

---

## 3. Attribute（全局属性）

只要 Web 容器还在运行，`ServletContext` 对象的 Attribute 就对容器中所有的成员可见。

- **数据共享：** `Attribute` 主要用于 Web 容器内部各功能之间的数据共享。
- **可修改：** 容器运行过程中，任何成员都可以对 `Attribute` 进行修改。

---

## 4. 常用 API

#### ✅ 添加/获取全局属性

| 方法                                            | 描述                    |
| ----------------------------------------------- | ----------------------- |
| **`void`** `setAttribute(String, Object)`       | 添加/修改全局属性键值对 |
| **`Object`** `getAttribute(String)`             | 获取全局属性值          |
| **`void`** `removeAttribute(String)`            | 移除指定的全局属性      |
| **`Enumeration<String>`** `getAttributeNames()` | 获取所有键的枚举        |

> 全局属性在 Web 容器中全局可见，Web 容器内所有成员都可以**访问和修改**。

#### ✅ 获取初始化参数

| 方法                                                | 描述                       |
| --------------------------------------------------- | -------------------------- |
| **`String`** `getInitParameter(String)`             | 获取初始化参数的值         |
| **`Enumeration<String>`** `getInitParameterNames()` | 获取所有初始化参数键的枚举 |

> 初始化参数在 `web.xml` 中声明，Web 容器启动后初始化并加载到 `ServletContext` 对象中，**全局可见但无法再次修改**。

#### ✅ 资源加载 & 路径

| 方法                                            | 描述                                 |
| ----------------------------------------------- | ------------------------------------ |
| **`String`** `getRealPath(String)`              | 获取文件或目录的**绝对路径**         |
| **`URL`** `getResource(String)`                 | 获取指定路径资源的 **URL 对象**      |
| **`InputStream`** `getResourceAsStream(String)` | 获取指定路径资源的**输入流**         |
| **`Set<String>`** `getResourcePaths(String)`    | 获取指定目录下所有资源的**路径集合** |

---

## 5. 资源加载

Web 容器支持三种常用的资源加载方式：

#### 通过 `getRealPath` 获取资源绝对路径

```java
String realPath = getServletContext().getRealPath("/WEB-INF/classes/config.properties");
InputStream is = new BufferedInputStream(new FileInputStream(realPath));	// 通过绝对路径创建资源流
```

#### 通过 `getResourceAsStream` 获取资源流

```java
InputStream is = getServletContext().getResourceAsStream("/WEB-INF/classes/config.properties");	// 直接获取资源流
```

#### 通过类加载器获取资源流（✅ 推荐）

```java
InputStream is = getClass().getClassLoader().getResourceAsStream("config.properties");
```

> 通过类加载器获取资源时，**无需关注绝对路径或相对路径，只要资源与类路径平齐即可**。这种方法更加灵活，适用于**不依赖特定路径且位于类路径中的资源**。

---

## ✅ 总结

- **`ServletContext`** 提供 Web 应用的全局上下文，主要用于共享全局数据，如初始化参数和全局属性。
- **初始化参数（`InitParameter`）** 存储在 `web.xml` 中，并在 Web 容器启动时初始化到 `ServletContext` 对象中。它们对所有成员可见，但无法修改，通常用于存储配置数据。
- **全局属性（`Attribute`）** 在容器运行期间可被所有成员访问和修改，主要用于容器内部各功能之间的数据共享。
- 常用的 **API** 包括添加/获取全局属性、获取初始化参数、加载资源等操作。
- **资源加载** 方式：
  - **`getRealPath`**：通过绝对路径获取资源，适用于需要文件路径的场景。
  - **`getResourceAsStream`**：通过相对路径获取资源流，通常用于读取配置文件等。
  - **类加载器方式**：不依赖路径，适用于类路径下的资源，推荐使用。
