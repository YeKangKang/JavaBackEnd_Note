# Day23 · JavaEE（Servlet实现类、HttpServletRequest、HttpServletResponse）

---

## 1. Servlet 实现类

在实际开发中，自定义 `Servlet` 实现类通常继承 `HttpServlet`，仅重写需要的方法即可，避免实现 `Servlet` 接口的全部方法。

### 常用方法

| 方法         | 说明                   |
|--------------|------------------------|
| `doGet()`    | 处理 GET 请求（查询、页面加载） |
| `doPost()`   | 处理 POST 请求（表单提交）     |

---

## 2. `HttpServletRequest` · 客户端请求对象

`HttpServletRequest` 是服务器在调用 `doGet()` 或 `doPost()` 方法时自动传入的参数对象，**内部封装了客户端请求的全部信息**，如：路径、请求参数、请求头、请求方式、IP 等。

### 2.1 获取请求参数

| 方法                            | 返回类型                   | 用途                     |
|---------------------------------|----------------------------|--------------------------|
| **`getParameter(String)`**     | `String`                  | 获取单个字段值           |
| **`getParameterValues(String)`** | `String[]`                | 获取多值字段（如复选框） |
| **`getParameterMap()`**        | `Map<String, String[]>`   | 获取所有参数             |

**示例：获取请求参数**

```java
// 设置字符集防止乱码
req.setCharacterEncoding("UTF-8");

// 方法 1: 获取单值参数
String username = req.getParameter("username");
// 方法 2: 获取多值参数
String[] hobbies = req.getParameterValues("hobby");
// 方法 3: 获取所有请求参数
Map<String, String[]> paramMap = req.getParameterMap();
```

---

### 2.2 获取请求信息

| 方法                    | 说明                       |
| ----------------------- | -------------------------- |
| **`getRequestURI()`**   | 获取 URI 路径部分 `/xx/yy` |
| **`getRequestURL()`**   | 获取完整 URL               |
| **`getMethod()`**       | 获取请求方式（GET/POST）   |
| **`getHeader(String)`** | 获取指定请求头             |
| **`getRemoteAddr()`**   | 获取客户端 IP 地址         |

---

## 3. `HttpServletResponse` · 服务器响应对象

用于设置响应数据、状态码、跳转等信息，由服务器自动传入。

### 3.1 设置响应头

| 方法                             | 说明                             |
|----------------------------------|----------------------------------|
| **`setContentType(String)`**    | 设置 MIME 类型 + 编码**（推荐）** |
| **`setCharacterEncoding(String)`** | 设置字符编码**（不通知浏览器）** |
| **`setHeader(String, String)`** / **`addHeader(String, String)`** | 设置响应头**（覆盖 / 不覆盖）** |
| **`setIntHeader(String, int)`** / **`addIntHeader(String, int)`** | 设置整数类型响应头**（覆盖 / 不覆盖）** |
| **`addCookie(Cookie)`**         | 将 Cookie 发送给浏览器              |

> 推荐使用 **`setContentType("text/html;charset=utf-8")`** 统一设置响应类型与编码。

---

### 3.2 输出内容（文本 / 字节流）

| 方法                   | 用途                 |
|------------------------|----------------------|
| **`getWriter()`**      | 输出字符数据（文本、HTML） |
| **`getOutputStream()`** | 输出字节数据（图片）  |

> 在同一个请求处理方法中，`getWriter()` 和 `getOutputStream()` 互斥，**只能调用其中一个**。

**示例：设置中文响应防乱码**

```java
// 推荐方式：一行设置 MIME 类型 + 编码
response.setContentType("text/html;charset=utf-8");
response.getWriter().write("中文测试");

// 不推荐方式：分步设置，易遗漏
response.setCharacterEncoding("utf-8");
response.setHeader("content-type", "text/html;charset=utf-8");
response.getWriter().write("中文测试");
```

---

### 3.3 状态码 & 重定向

| 方法                       | 用途                           |
| -------------------------- | ------------------------------ |
| **`setStatus(int)`**       | 设置状态码（如 200、302）      |
| **`sendRedirect(String)`** | 临时重定向，浏览器重新发起请求 |

**示例：重定向请求**

```java
// 浏览器将自动重新请求指定路径
res.sendRedirect("/demo1/getImg");

// 等价于：
res.setStatus(302);
res.setHeader("Location", "/demo1/getImg");
```

---

## 4. 综合示例：下载文件

```java
// 1. 获取请求参数中文件名
String fileName = req.getParameter("file-name");

// 2. 获取文件在服务器的真实路径
ServletContext context = getServletContext();
String realPath = context.getRealPath(fileName);

// 3. 设置 MIME 类型与下载响应头
String mimeType = context.getMimeType(fileName);
res.setHeader("Content-type", mimeType);
res.setHeader("Content-disposition", "attachment;filename=" + fileName);

// 4. 使用字节流完成文件读取与写出
BufferedInputStream bis = new BufferedInputStream(new FileInputStream(realPath));
OutputStream os = res.getOutputStream();
byte[] buffer = new byte[1024];
int len;
while ((len = bis.read(buffer)) != -1) {
    os.write(buffer, 0, len);
}
os.close();
bis.close();
```

---

## ✅ 总结

- `HttpServletRequest` 封装请求所有信息：参数、路径、请求方式、请求头
- `HttpServletResponse` 用于构造响应结果，包括设置响应头、编码、状态码、内容输出
- 实际开发中可根据业务场景灵活使用 `getWriter()` 或 `getOutputStream()`，设置合理的 `Content-Type` 避免乱码
- 下载/重定向等行为依赖正确设置响应头字段，建议遵循标准方式处理
