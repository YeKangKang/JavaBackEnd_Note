# Day33 · Spring MVC（MVC、Spring MVC、开发流程、DispatcherServlet、Bean 加载控制）

## 1. MVC 概述

MVC 是**表现层中的界面架构模式**，强调将数据、视图与控制逻辑解耦，实现结构清晰与职责分离。

- **`Model`（模型）**：封装业务数据与状态。（如 POJO、DTO）
- **`View`（视图）**：负责数据展示与页面渲染。（如 JSP、Thymeleaf）
- **`Controller`（控制器）**：处理前端请求，并协调 Model 与 View 的交互。

---

## 2. Spring MVC 概述

Spring MVC 是 Spring 提供的**基于 MVC 架构的 Web 请求-响应处理框架**，主要用于**三层架构中的表现层**。它在原生 Servlet 基础上进行了封装，屏蔽了分发、绑定、视图渲染等底层细节，使开发者可专注于业务逻辑开发。

- **请求接收**：用户请求由 `DispatcherServlet` 统一接收并分发至对应的控制器（`@Controller`）
- **业务调用**：Controller 调用 Service 执行业务逻辑，Service 内部继续调用 Dao 层完成数据操作
- **结果响应**：将处理结果返回为页面视图或 JSON 数据供前端使用

> 在前后端分离架构中，Spring MVC 主要承担**请求处理与数据响应的职责**（Controller + Model），通常通过 JSON 向前端提供处理结果；页面展示（View）则由前端框架如 Vue、React 负责完成。

---

## 3. Spring MVC 开发流程

#### XML 配置开发
- 通过 `web.xml` **注册 DispatcherServlet**
- 通过 `spring-mvc.xml` 配置组件扫描、视图解析器等 **Spring MVC 组件**

#### Web 3.0 全注解开发（推荐）

- 通过继承抽象类**实现 DispatcherServlet 初始化**
  - 继承 `AbstractDispatcherServletInitializer`：标准方式，需手动指定配置类
  - 继承 `AbstractAnnotationConfigDispatcherServletInitializer`：快捷方式，自动加载配置类

- 使用 **Spring MVC 配置类统一管理组件注册**（如通过 `@ComponentScan` 扫描 Controller、配置视图解析器等）

### 3.1 通用开发流程

```tex
1. 引入依赖（spring-webmvc、servlet-api 等）
2. 配置 DispatcherServlet（web.xml / 配置类）
3. 编写 Controller 与业务方法
4. 设置 URL 映射关系
5. 注册 SpringMVC 组件（扫描路径、视图解析器等）
6. 启动服务，测试请求响应流程
```

---

## 4. 前端控制器（DispatcherServlet）

**DispatcherServlet 是 Spring MVC 的核心组件，继承自 HttpServlet，用于接收、分发请求和生成响应**，贯穿整个 Web 应用生命周期。

其内部维护一个独立的 `WebApplicationContext`（继承自 `ApplicationContext`），专用于管理表现层相关的 Bean 如控制器（Controller）、视图解析器（ViewResolver）、拦截器（Interceptor）等。

> 注册 Controller、视图解析器等组件的操作，本质上是将它们加载到 `DispatcherServlet` 所持有的 MVC 子容器中。

#### 例：`AbstractDispatcherServletInitializer` 注册 DispatcherServlet

```java
public class DispatcherServletConfig extends AbstractDispatcherServletInitializer {

    // 加载 Spring 主容器（业务层、持久层）
    @Override
    protected WebApplicationContext createRootApplicationContext() {
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(SpringConfig.class);
        return ctx;
    }

    // 加载 SpringMVC 子容器（表现层）
    @Override
    protected WebApplicationContext createServletApplicationContext() {
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(SpringMvcConfig.class);
        return ctx;
    }

    // 设置 DispatcherServlet 拦截路径
    @Override
    protected String[] getServletMappings() {
        return new String[]{ "/" };
    }
}
```

- **`createRootApplicationContext()`**：**构建 Spring 主容器。**加载非 Web 层组件（如 Service、DAO）
- **`createServletApplicationContext()`**：**构建 Spring MVC 子容器。**加载 Web 层组件
- **`getServletMappings()`**：**指定 DispatcherServlet 拦截路径。**

> 也可继承 `AbstractAnnotationConfigDispatcherServletInitializer` 使用更简洁方式进行配置。

#### 例：`web.xml` 注册 DispatcherServlet

```xml
<!-- 注册 DispatcherServlet -->
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 指定 SpringMVC 配置文件路径 -->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-mvc.xml</param-value>
    </init-param>
    <!-- 设置启动时加载（0 表示容器启动时立即加载） -->
    <load-on-startup>0</load-on-startup>
</servlet>

<!-- 设置请求拦截路径（/ 表示拦截所有请求） -->
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

---

## 5. Bean 加载控制

在常规 SSM 项目中，通常包含 Spring 主容器（管理 Service、Dao）和 Spring MVC 子容器（管理 Controller）。为避免职责混乱或重复扫描，建议各容器**只加载自身所需的 Bean**，常见方式包括：

- **重复扫描**（默认做法）：不做区分，通常无问题，但可能因重复注册引发冲突

  ```java
  @ComponentScan("org.example") // Spring 配置类
  @ComponentScan("org.example") // MVC 配置类
  ```

- **分层扫描**（推荐）：每个容器只扫描自身职责范围

  ```java
  @ComponentScan({"org.example.dao", "org.example.service"}) // Spring 配置类
  @ComponentScan("org.example.controller") // MVC 配置类
  ```

- **全包扫描 + 注解排除**：主容器扫描全包并排除 `@Controller`，避免加载表现层组件（需避免主容器扫描到 MVC 配置类本身，否则会导致子容器组件被主容器重复加载）

  ```java
  @ComponentScan(
    basePackages = "org.example",
    excludeFilters = @ComponentScan.Filter(
      type = FilterType.ANNOTATION,
      classes = Controller.class)) // Spring 配置类（排除 @Controller 注解）
  
  @ComponentScan("org.example.controller") // MVC 配置类
  ```

---

## ✅ 总结

- Spring MVC 是基于原生 Servlet 封装的 Web 表现层框架，核心在于职责清晰、结构解耦
- DispatcherServlet 是请求总控入口，负责统一分发、处理与响应，内部维护独立的 MVC 子容器
- 开发方式推荐使用注解配置，结合配置类注册组件、管理扫描路径
- SSM 架构下存在主/子容器，建议按职责分层扫描 Bean，避免重复注册和职责混乱
