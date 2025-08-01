# Day22 · JavaEE（Tomcat服务器、Tomcat启动、Web项目结构、IDEA配置Tomcat）

---

## 1. Tomcat 服务器

Tomcat 是一个开源的轻量级 Web 服务器，支持 JavaEE 中部分规范，如 `Servlet`、`JSP`、`EL 表达式` 等。适用于中小型系统、并发访问不高的场景，是开发调试 Java Web 项目的常用服务器。

> 每新建一个 Web 项目时，可复制一份 Tomcat 压缩包并解压，将解压目录配置到 IDEA 或其他 IDE 中使用。

#### Tomcat 目录

| 目录名     | 作用说明 |
|------------|----------|
| `bin/`     | 启动/关闭脚本等（如 `startup.sh`、`shutdown.bat`） |
| `conf/`    | 全局配置（如端口、编码、权限）含 Tomcat 自带的：`server.xml`、`web.xml`、`tomcat-users.xml` 等 |
| `lib/`     | 运行依赖的 jar 包 |
| `webapps/` | Web 应用部署目录（默认扫描） |
| `logs/`    | 运行日志输出 |
| `work/`    | 缓存目录（如 JSP 编译生成的 Servlet 文件） |
| `temp/`    | 临时文件目录 |
| 其他文件   | 如 `LICENSE`、`README` 为附属说明文档，可忽略 |


## 2. Tomcat 启动

- Tomcat 启动方式：运行 `startup.sh`（或 `.bat`）脚本，关闭用 `shutdown.sh`

- 启动后会监听以下端口（默认配置）：
  - `8005`：关闭端口（接收 shutdown 命令）
  
  - `8080`：主 HTTP 服务端口
  
  - `8009`：AJP 连接器端口（与 Apache 协作用）
  
> 若多个 Tomcat 实例绑定同一端口，将出现端口冲突错误


## 3. Web 项目

Web 项目可打包为 `.war` 文件部署到 Tomcat 的 `webapps/` 下，Tomcat 启动时会自动识别并运行该项目。

开发阶段通过 IDE（如 IDEA）配置本地 Tomcat 并绑定运行配置后，可无需手动打包 `.war`，IDEA 会构建临时部署结构并挂载至虚拟目录运行。

#### Web 项目标准资源结构

```
MyWebProject/
├── src/                  → Java 源码目录（如 Servlet、Listener 等）
│   └── com/example/...
├── web/                  → Web 根目录（部署后的公开访问起点）
│   ├── index.jsp         → 首页页面
│   ├── static/           → 静态资源目录（JS/CSS/图片）
│   └── WEB-INF/          → 私有资源目录（不可被客户端直接访问）
│       ├── web.xml       → 部署描述符
│       ├── lib/          → 第三方依赖（非 Maven 项目）
│       └── classes/      → 编译输出目录（自动生成）
└── pom.xml / build.xml   → 构建工具配置文件
```

- **`src/`：**源码目录，仅用于开发维护，不参与直接部署
- **`web/`：**Web 根目录，部署入口
  - **静态资源：**静态资源应放在 `web/` 根目录或其子目录中（如 HTML、CSS、JS、图片）
  - **`WEB-INF/`：**私有资源目录，<u>**不可被客户端直接请求访问**</u>，包含：
    - `web.xml`：配置 Servlet、Filter、Listener 等
    - `classes/`：编译生成的 `.class` 文件
    - `lib/`：第三方依赖（若未使用 Maven）


> 现代 Maven Web 项目采用标准目录结构，**`web/` 被规范为 `src/main/webapp/`**，其中包含 `WEB-INF/web.xml` 等配置文件。**项目源码和资源也被明确划分为 `main` 和 `test` 两个部分**，确保构建过程清晰、规范。


## 4. IDEA 中配置 Tomcat

配置本地 Tomcat 后，点击 Run 时 IDEA 会：

1. **将项目打包为标准的 Web 应用结构**（含 `WEB-INF` 等），并放入Maven 管理的 `target/` 目录（或自定义输出目录）
2. 将此结构映射为 Tomcat 的虚拟部署目录（Application Context）

> IDEA 中配置 Tomcat 后，**每一次运行项目会自动将代码构建为临时部署结构**，映射为虚拟路径挂载至 Tomcat 中，方便快速调试而无需手动打包。

## ✅ 总结

- Tomcat 是轻量级 Java Web 服务器，支持部分 JavaEE 规范（如 Servlet、JSP、EL 表达式）
- 启动 Tomcat 后会占用默认端口：`8005`（关闭）、`8080`（HTTP）、`8009`（AJP）
- Web 项目可打包为 `.war` 部署到 `webapps/`，也可通过 IDEA 配置本地 Tomcat 实现临时调试运行
- 标准 Web 项目结构包括 `src`（源码）与 `web/`（部署根目录），内部包含静态资源与 `WEB-INF` 私有目录
- `WEB-INF` 下包含 `web.xml` 配置文件、`classes` 编译目录与 `lib` 第三方依赖（非 Maven 时使用）
- 现代 Maven 项目结构中，`web/` 被替换为 `src/main/webapp/`，含义一致，结构不同
- IDEA 启动调试项目时会将项目构建为部署结构，放入 `target/`，无需手动打包 `.war` 即可运行
