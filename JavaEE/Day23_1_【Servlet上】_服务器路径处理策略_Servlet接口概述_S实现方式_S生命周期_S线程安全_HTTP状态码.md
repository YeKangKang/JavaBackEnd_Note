# Day23 · JavaEE（服务器路径处理策略、Servlet接口概述、S实现方式、S生命周期、S线程安全、HTTP状态码）

---

## 1. URL 路径处理策略

大多数主流服务器处理 URL 路径时遵循 “**先匹配动态资源，后查静态资源，最后才返回 404**” 的策略：

1. 请求 URL 到达（如 `http://www.myWeb.com/login`），解析出路径 `/login`
2. **优先查找是否有注册 Servlet（动态资源）**
3. 若无，再查找是否存在对应的静态文件 `/login`
4. 若仍无匹配，返回 **404 Not Found**

---

## 2. Servlet 接口概述

`Servlet`（Server Applet）是 JavaEE 中的服务端组件，用于处理浏览器或客户端请求，是**服务器的最小服务接口**。它用于接收、处理客户端发起的 URL 请求，并将结果响应回客户端。

#### 使用建议 & 入口设计

实际开发中，应**避免碎片化注册多个 Servlet 类**，而是：

- 使用统一的功能入口路径（如 `/login`），避免为每个子功能都注册一个新的 Servlet
- 通过请求参数（如 `/login?action=check`）在 Servlet 内部分流处理逻辑，而非将 `/login/check` 作为一个新路径注册

> 一个 Servlet 可作为某个模块的统一入口，再通过参数值判断具体子功能，提升维护性与扩展性。

---

## 3. Servlet 的实现方式

### 3.1 Servlet 3.0 之前（web.xml 配置）

**Servlet 类**

```java
public class HelloServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        System.out.println("POST method called…");
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        System.out.println("GET method called…");
        resp.getWriter().write("Hello from HelloServlet!");
    }
}
```

**web.xml 配置**

```xml
<servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>com.example.demo1.HelloServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>hello</servlet-name>
  	<!-- 配置多个 URL 路径映射到同一个 Servlet -->
    <url-pattern>/hello</url-pattern>
    <url-pattern>/hi</url-pattern>
    <url-pattern>/greeting</url-pattern>
</servlet-mapping>
```

- **`<servlet-name>`**：中间名，用作 URL 路径与 Servlet 类的桥梁（早期规范所需）
- **`<servlet-class>`**：映射的 Servlet 类全限定名
- **`<url-pattern>`**：URL 路径，**可配置多个路径对应同一个 Servlet**

---

### 3.2 Servlet 3.0 之后（注解配置）

```java
/* 注解方式映射多个 URL 到当前 Servlet 类 */
@WebServlet(urlPatterns = {"/hello", "/hi", "/greeting"})
public class HelloServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        System.out.println("POST method called…");
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        System.out.println("GET method called…");
        resp.getWriter().write("Hello from HelloServlet!");
    }
}
```

**`@WebServlet()` 注解常用参数：**

| 参数名          | 作用                           | 示例                                                       |
| --------------- | ------------------------------ | ---------------------------------------------------------- |
| **`name`**      | Servlet 的名称（中间名）         | `@WebServlet(name = "hello")`                              |
| **`urlPatterns`** | 显式配置路径数组                 | `@WebServlet(urlPatterns = {"/a", "/b"})`                  |
| **`value`**     | `urlPatterns` 的简写形式         | `@WebServlet("/hello")` 或 `@WebServlet({"/a", "/b"})`     |
| **`loadOnStartup`** | 控制 Servlet 是否懒加载（默认 `-1` 懒加载，`≥0` 表示预加载） | `@WebServlet(value = "/hi", loadOnStartup = 1)`            |


> **`@WebServlet("/hello")`** 或 **`@WebServlet({"/hello", "/hi"})`** 是实际使用中更推荐的写法，它们是 `value` 参数的简写形式。

---

## 4. Servlet 生命周期

Servlet 生命周期由接口中三个核心方法控制：

- `init()`：初始化方法，**仅执行一次**
- `service()`：用于处理请求和响应，每次请求都会调用
- `destroy()`：销毁方法，服务器**关闭时调用一次**

> **默认情况下，Servlet 会在首次访问时创建**（懒加载）。也可以通过在 `web.xml` 使用 `<load-on-startup>`，或在 `@WebServlet` 注解中配置，使其在服务器启动时初始化。

---

## 5. Servlet vs 线程安全

每一个对应特定 URL 路径的 Servlet 类都是单例模式，在服务器运行期间最多只有一个实例。如果多个请求并发访问这个共享对象，可能会引发线程安全问题：

- **避免使用非只读的成员变量**
- 尽量让 Servlet 类保持无状态，仅作为请求处理工具
- 同步锁（如 `synchronized`）仅在极端情况下使用，一般应避免，以免影响吞吐量和效率

> 避免在 Servlet 中使用使用**非只读的成员变量**，以避免出现线程安全问题。

---

## 6. 常见 HTTP 状态码

| 状态码 | 状态名称            | 解释                         |
| ------ | ------------------- | ---------------------------- |
| **200** | `OK`                | 请求成功                     |
| **302** | `Found`             | 临时重定向                   |
| **403** | `Forbidden`         | 权限不足，禁止访问           |
| **404** | `Not Found`         | 路径未找到                   |
| **405** | `Method Not Allowed`| 请求方法不被支持             |
| **500** | `Internal Server Error` | 服务器内部错误            |
| **503** | `Service Unavailable`   | 服务不可用（如宕机、重启中） |

---

## ✅ 总结

- Servlet 是 JavaEE 标准的服务端组件，负责处理 URL 请求
- Servlet 可通过实现接口或继承 `HttpServlet` 实现，常采用注解方式注册路径
- Servlet 对象为单例，需注意线程安全问题
- 生命周期包括 `init()`、`service()`、`destroy()` 三个阶段
- HTTP 状态码是 Web 编程的重要参考指标
