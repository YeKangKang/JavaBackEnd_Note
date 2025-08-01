# Day33 · Spring MVC（HTTP请求、@RequestMapping、请求参数、中文参数乱码处理、传统 / JSON / 日期参数绑定）

## 1. 常见 HTTP 请求

HTTP 协议支持多种请求方式，但浏览器原生表单仅支持 **GET** 与 **POST**，其余如 **PUT**、**DELETE** 等通常通过 AJAX 或 RESTful 接口使用。

- **GET**：**从服务器获取资源。**通过 URL 查询参数传递，数据可见，适用于小体积、非敏感的读取操作
- **POST**：**向服务器提交资源。**通过请求体提交数据，适合表单提交与敏感信息传输
- **PUT**：**更新资源。**通常携带完整数据，具备幂等性（多次执行结果相同）
- **DELETE**：**删除资源。**具备幂等性
- **HEAD**：**仅获取响应头，不返回响应体。**用于检测资源存在性或元信息

---

## 2. 请求路径映射（`@RequestMapping`）

`@RequestMapping` 用于将客户端请求 URL 映射到指定的 Controller 类或方法。该注解本身**不区分请求方式**，只要路径匹配，所有类型的请求均可处理。

- **作用于类上**：定义**统一路径前缀**，作为该类中所有方法的映射基础，避免路径冲突
- **作用于方法上**：定义**具体处理路径**，若类上也标注了路径，则组成“前缀 + 方法路径”的完整映射

```java
@Controller
@RequestMapping("/user") // 统一路径前缀
public class UserController {
    @RequestMapping("/delete")
    public String delete() { ... } // 映射路径为 /user/delete
}
```

> `@RequestMapping` 默认不区分请求类型，适用于通用请求映射；在 REST 风格中可通过 **`method` 属性**或语义化注解（如 `@GetMapping`、`@PostMapping`）对请求方式进行明确区分。

---

## 3. 请求参数绑定（`@RequestParam` / `-parameters`）

在 Java 中，`.class` 文件默认不保留方法参数名，Spring 无法通过反射识别实际参数名，导致请求参数无法自动绑定到方法参数。可通过 `@RequestParam` 或 `-parameters` 解决。

- **`@RequestParam`：**用于绑定 URL 查询参数或表单字段。**适用于传统 `key=value` 形式的请求**，不适用于 JSON 请求体。

  ```java
  @RequestMapping("/param")
  public String func(@RequestParam("name") String name, @RequestParam("age") int age) {...}
  ```

- **`-parameters`：**在 `pom.xml` 中通过插件启用编译参数，让 Spring 自动识别方法参数名（适用于 JDK8+，无需使用注解）

  ```xml
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
      <compilerArgs>
        <arg>-parameters</arg>
      </compilerArgs>
    </configuration>
  </plugin>
  ```

---

## 4. 中文参数乱码处理（`CharacterEncodingFilter`）

Spring 提供 `CharacterEncodingFilter` 解决传统 `key=value` 请求中的中文乱码问题，适用于 **URL 查询参数** 与 **表单字段**。

通常在 DispatcherServlet 的**初始化类中通过重写 `getServletFilters()` 方法注册**，用于统一设置请求与响应的编码格式。

```java
// 省略 3 个核心方法：Spring 容器、MVC 容器、DispatcherServlet 拦截路径
public class DispatcherServletConfig extends AbstractDispatcherServletInitializer {
    @Override
    protected Filter[] getServletFilters() {
        CharacterEncodingFilter encodingFilter = new CharacterEncodingFilter();
        encodingFilter.setEncoding("UTF-8");         // 设置字符编码为 UTF-8
        encodingFilter.setForceEncoding(true);       // 强制应用编码到请求和响应
        return new Filter[]{encodingFilter};         // 可注册多个过滤器
    }
}
```

> 若使用 XML 配置，也可通过 `<filter>` 在 `web.xml` 中注册，作用一致。

---

## 5. `key=value` 结构参数绑定（传统方式）

| 类型          | 描述                                       | 绑定示例（方法参数）                                 |
| ------------- | ------------------------------------------ | ---------------------------------------------------- |
| **基本类型**  | 使用 `@RequestParam` 显式绑定              | `(@RequestParam("name") String name)`               |
| **POJO**      | 自动按字段注入，依赖 `setter()` 方法       | `(User user)`                                       |
| **嵌套 POJO** | 使用 `对象名.属性名` 格式绑定内部对象属性  | `(UserAndAddress userAndAddress)`                   |
| **数组**      | 同名字段组装为数组，需加 `@RequestParam`   | `(@RequestParam("hobby") String[] arr)`             |
| **集合**      | 同数组，使用集合接收，需加 `@RequestParam` | `(@RequestParam("hobby") List<String> tags)`        |

---

## 6. JSON 参数绑定

现代 Web 中，最常见的参数传递方式是将 **JSON 数据写入请求体**，由服务端反序列化为 Java 对象绑定到控制器方法参数中。该方式不依赖传统的 `key=value` 查询参数格式。

> SpringMVC 支持自动反序列化 JSON，前提是：

- **引入依赖**：`jackson-databind`（或 Spring Boot 默认集成）

- **启用注解驱动**：`@EnableWebMvc` / `<mvc:annotation-driven />`

- 控制器方法参数使用 `@RequestBody` 注解

### 6.1 触发反序列化机制（`@RequestBody`）

使用 `@RequestBody` 注解时，SpringMVC 会通过 **HttpMessageConverter** 将请求体中的 JSON 数据自动反序列化为 Java 对象。

**每个处理方法只能声明一个 `@RequestBody` 参数**，即一次请求的所有数据必须统一封装为一个 JSON 数据体进行传输。

| JSON 数据结构            | 方法参数接收方式                         | JSON 示例                           |
| ------------------------ | ---------------------------------------- | ----------------------------------- |
| **基本类型**             | `@RequestBody int`                       | `18`                                |
| **POJO 对象**            | `@RequestBody User user`                 | `{ "name": "Tom", "age": 18 }`      |
| **数组 / 集合**          | `@RequestBody String[]` / `List<String>` | `["A", "B", "C"]`                   |
| **POJO 对象数组 / 集合** | `@RequestBody User[]` / `List<User>`     | `[{"name":"Tom"},{"name":"Jerry"}]` |
| **键值对结构（Map）**    | `@RequestBody Map<String, Object>`       | `{ "id": 1, "flag": true }`         |

> `@RequestBody` 用于 **接收请求体中的 JSON 数据并自动绑定为对象**，对应于 `@ResponseBody` 的响应序列化机制（将 Java 对象转换为 JSON）。

---

## 7. 日期参数绑定

SpringMVC 可自动将请求参数中的日期字符串转换为 Java 中的时间类型（如 `Date`、`LocalDate`、`LocalDateTime`），需要满足：

- **默认格式匹配：** 日期字符串符合类型默认格式可自动转换（如 `Date` 为 `yyyy-MM-dd`）

- **启用注解驱动**：`@EnableWebMvc` / `<mvc:annotation-driven />`

### 7.1 非默认格式转换（`@DateTimeFormat`）

若请求参数不符合接收的Java日期对象的默认日期格式，**可使用 `@DateTimeFormat(pattern = "...")` 指定解析格式**。

```java
@RequestMapping("/wrongFormatDate")
public String query(@RequestParam("date") @DateTimeFormat(pattern = "yyyy-MM-dd") Date date) {...} // 将 "yyyy-MM-dd" 格式的字符串参数绑定为 Date 类型
```

> 该注解仅适用于 `key=value` 结构的请求（如 URL 参数、表单字段），**不支持 JSON 请求体中的日期字符串**

---

## ✅ 总结

- 请求映射通过 `@RequestMapping` 定义，默认不区分请求类型，可配合 `method` 属性或语义注解精确控制
- 请求参数支持多种绑定方式：
  - `@RequestParam`：绑定 URL 查询参数 / 表单字段，适用于 key=value 请求
  - POJO / 嵌套 POJO / 数组 / 集合：按名称自动绑定，支持结构化数据
  - `@RequestBody`：绑定 JSON 请求体，仅支持一个参数，用于结构化数据传输
- 编译器参数 `-parameters` 可提升无注解绑定兼容性（需 JDK8+）
- 中文乱码处理推荐使用 `CharacterEncodingFilter`，统一设置请求/响应编码
- 日期字符串支持默认自动转换，非标准格式使用 `@DateTimeFormat` 显式指定（仅支持 URL / 表单参数）
