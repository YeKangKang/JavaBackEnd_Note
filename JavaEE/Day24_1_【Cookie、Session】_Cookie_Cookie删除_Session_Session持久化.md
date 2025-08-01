
# Day24 · JavaEE（Cookie、Cookie删除、Session、Session持久化）

## 1. 会话概念

**会话**指浏览器与服务器之间的一段连续请求和响应过程。Web 应用会通过 `Cookie` 和 `Session` 进行会话数据的临时储存和交换：

- **Cookie**：存储在浏览器端，字符形式保存，不安全。
- **Session**：存储在服务器端，对象形式保存，安全。

> `Session` 基于 `Cookie` 技术，通过浏览器端所携带的名为 `JSESSIONID` 的 `Cookie` 来区分不同的浏览器。

---

## 2. Cookie

### 2.1 概念
`Cookie` 是一种 **客户端会话技术**，本质上是由服务器发送并保存在浏览器中的一小段文本数据。每次浏览器访问符合 `Cookie` 作用域的服务器时，都会携带这些数据，用于识别用户身份和记录用户状态。

- **生命周期：**生命周期默认为会话，可通过 `setMaxAge` 设置过期时间。
- **唯一性：**同 `(name, path, domain)` 的 `Cookie` 只能有一个，后者覆盖前者。
- **作用域：**`Cookie` 的作用域由 `Path` 和 `Domain` 决定，路径匹配时才会携带。

> 浏览器和服务器的每次请求可以携带多个 `Cookie`，每个 `Cookie` 保存一对键值对。

### 2.2 常用 Cookie API

| 方法                                  | 描述                                   |
| ------------------------------------- | -------------------------------------- |
| **`Cookie[]`** `getCookies()`         | `HttpServletRequest` 获取所有 `Cookie` |
| **`void`** `addCookie(Cookie)`        | `HttpServletResponse` 添加 `Cookie`    |
| **`Cookie`** `Cookie(String, String)` | 创建 `Cookie` 对象                     |
| **`String`** `getName()`              | 获取 `Cookie` 名称                     |
| **`String`** `getValue()`             | 获取 `Cookie` 值                       |
| **`void`** `setValue(String)`         | 设置 `Cookie` 值                       |
| **`void`** `setPath(String)`          | 设置 `Cookie` 作用路径                 |
| **`String`** `getPath()`              | 获取 `Cookie` 路径                     |
| **`void`** `setMaxAge(int)`           | 设置 `Cookie` 生命周期（秒）           |
| **`int`** `getMaxAge()`               | 获取 `Cookie` 生命周期                 |

### 2.3 删除Cookie
通过创建一个过期的 Cookie，使其 `name`、`path`、`domain` 与目标 Cookie 相同，并发送给客户端替换目标 Cookie。

```java
Cookie c = new Cookie("cName", "");	// 用于替换目标 Cookie
c.setMaxAge(0);  // 设置过期
response.addCookie(c);  // 发送给客户端，客户端会自动删除过期 Cookie
```

---

## 3. Session

### 3.1 概念
**`Session`** 存储在服务器端，用于在多个请求之间保持会话状态。通过唯一的 `Session ID` 区分不同会话，确保会话数据的安全。

- **生命周期：** 默认生命周期为 30 分钟（最大非活动时间），可通过 `setMaxInactiveInterval` 设置过期时间。
- **唯一对应：** 每个会话通过浏览器端的唯一 `Session ID` 区分。

> Session 通常会在第一次会话时发送一个名为 `JSESSIONID` 的 Cookie 给浏览器来区分不同的会话。

### 3.2 常用 Session API

| 方法                                      | 描述                                                        |
| ----------------------------------------- | ----------------------------------------------------------- |
| **`HttpSession`** `getSession()`          | `HttpServletRequest` 获取 `Session` 对象，不存在则创建      |
| **`HttpSession`** `getSession(false)`     | `HttpServletRequest` 获取 `Session` 对象，不存在返回 `null` |
| **`void`** `setAttribute(String, Object)` | 存储/更新数据到 `Session`                                   |
| **`Object`** `getAttribute(String)`       | 从 `Session` 获取数据                                       |
| **`void`** `removeAttribute(String)`      | 从 `Session` 删除数据                                       |
| **`String`** `getId()`                    | 获取当前 `Session` ID                                       |
| **`void`** `setMaxInactiveInterval(int)`  | 设置最大非活动时间                                          |
| **`int`** `getMaxInactiveInterval()`      | 获取最大非活动时间                                          |
| **`void`** `invalidate()`                 | 使当前 `Session` 失效                                       |

### 3.3 Session 持久化
为了确保会话数据能够在浏览器关闭后仍然有效，可以通过手动设置长生命周期的 `JSESSIONID` Cookie 来实现持久化。

```java
HttpSession session = req.getSession();
Cookie c = new Cookie("JSESSIONID", session.getId());	// 制作一个用于替换 SessionID 的 Cookie
c.setMaxAge(60 * 60 * 24 * 2);  // 设置生命周期为两天
response.addCookie(c);
```

---

## ✅ 总结
- **Cookie** 存储在客户端，生命周期可设置，作用范围由 `Path` 和 `Domain` 决定。
- **Session** 存储在服务器端，通过 `Session ID` 唯一标识会话，生命周期为 30 分钟。
- 可以通过设置 `MaxAge(0)` 删除 Cookie，Session 数据通过 `invalidate()` 使其失效。
