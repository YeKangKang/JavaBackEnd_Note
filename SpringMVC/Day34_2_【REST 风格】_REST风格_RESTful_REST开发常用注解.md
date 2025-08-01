# Day34 · Spring MVC（REST风格、RESTful、REST开发常用注解）

## 1. REST 风格概述

REST（Representational State Transfer）是一种接口设计风格，强调通过统一、语义明确的方式使用 HTTP 协议操作资源。广泛应用于前后端分离开发中，后端负责返回 JSON 数据，不涉及视图渲染。

REST 风格以 **URI 表示资源**，以 **请求方法（如 GET、POST、PUT、DELETE）表达操作行为**，避免在 URL 中使用动词暴露操作意图。

- **URI（统一资源标识符）**：用于标识资源集合，通常对应一个控制器模块

- **请求方法**：表示客户端意图的 HTTP 动作（如 `GET`/`POST`/`PUT`/`DELETE` 等）
- **命名建议**：URI 使用**资源名复数形式**，避免动词式命名（如 `/users`）

| URI 路径   | 请求方法 | 操作说明     |
| ---------- | -------- | ------------ |
| `/users`   | GET      | 查询所有用户 |
| `/users/1` | GET      | 查询指定用户 |
| `/users`   | POST     | 新增用户     |
| `/users/1` | PUT      | 更新用户     |
| `/users/1` | DELETE   | 删除用户     |

---

## 2. RESTful 接口实现（`@RequestMapping` + `@PathVariable`）

RESTful 用于描述符合 REST 风格的接口实现，通常在**类上用 `@RequestMapping` 定义统一 URI 前缀**，**方法上通过请求类型注解映射具体操作**。

```java
@Controller("userController")
public class UserController {
  
    @RequestMapping(value = "/users/{id}", method = RequestMethod.PUT) // 处理 PUT 请求
    @ResponseBody
    public String update(@PathVariable("id") int id, @RequestBody User userUpdate) { ... }
}
```

- **`@RequestMapping(value = ..., method = ...)`**
  - **`value`**：**映射路径**，可使用 `{}` 占位符表示路径变量
  - **`method`**：**指定请求类型**，用于明确请求语义
- **`@PathVariable("id")`**
  - 将 URI 中 `{id}` 的值绑定到方法参数
  - 启用 `-parameters` 编译参数可省略名称

---

## 3. REST 开发常用注解参考

| 注解                                  | 说明                                                      |
| ------------------------------------- | --------------------------------------------------------- |
| **`@RestController`**                 | 等价于 `@Controller` + `@ResponseBody`，返回 JSON 数据    |
| **`@RequestMapping`**                 | 可用于类上统一定义 URI 前缀，或通过 `method` 指定请求类型 |
| **`@GetMapping` / `@PostMapping` 等** | 请求方法语义注解，简化传统 `@RequestMapping` 写法         |
| **`@PathVariable`**                   | 绑定 URI 中的路径变量到方法参数                           |
| **`@RequestBody`**                    | 接收请求体中的 JSON 数据                                  |

```java
@RestController("userController") // @Controller + @ResponseBody
@RequestMapping("/users")
public class UserController {

    @PutMapping("/{id}") // 语义化请求映射
    public String update(@PathVariable("id") int id, @RequestBody User userUpdate) { ... }
}
```

---

## ✅ 总结

- REST 风格接口通过 URI + 请求方法组合表达操作语义，避免 URL 中使用动词
- 推荐使用资源名复数形式作为 URI 路径，提升规范性与一致性
- 常用注解：
  - `@RestController` 简化 JSON 接口开发
  - `@GetMapping`、`@PostMapping` 等语义注解替代传统 `@RequestMapping`
  - `@PathVariable` 和 `@RequestBody` 用于路径参数绑定与请求体解析
- 类级注解配合语法糖可提升开发效率并统一接口结构

