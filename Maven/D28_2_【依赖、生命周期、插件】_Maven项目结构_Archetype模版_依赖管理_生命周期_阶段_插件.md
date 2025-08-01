# Day28 · Maven（Maven项目结构、Archetype模版、依赖管理、生命周期、阶段、插件、目标、mvn指令）

## 1. Maven 项目结构

### 1.1 工程目录结构（标准 Java 项目）

```bash
my-app/
├── pom.xml                # 项目构建描述文件（核心）
├── src/                   # 源代码根目录
│   ├── main/              # 主程序代码
│   │   ├── java/          # Java 源码
│   │   ├── resources/     # 配置文件、资源文件（不编译）
│   └── test/              # 测试代码
│       ├── java/          # 测试源码
│       └── resources/     # 测试资源文件
└── target/                # 编译输出目录（Maven 自动生成）
```

### 1.2 构建产物结构（target）

```bash
target/
├── classes/                 # 编译后的主程序 class 文件
├── test-classes/            # 编译后的测试 class 文件
├── my-app-1.0-SNAPSHOT.jar  # 打包产物（取决于 packaging 类型）
├── generated-sources/       # 插件生成的源码（如 Lombok）
└── surefire-reports/        # 测试报告（配合 Surefire 插件）
```

---

## 2. Archetype 模版

Archetype 是 Maven 提供的项目模板机制，用于快速生成标准化项目骨架，常见原型：

- `maven-archetype-quickstart`：普通 Java 项目骨架
- `maven-archetype-webapp`：Web 项目最小骨架（仅含 `webapp/`）

```bash
# 使用 Maven 命令行生成 Web 项目骨架（等价于 IDEA 中选择 Web 模板）

mvn archetype:generate \
  -DgroupId=com.yuheng \                           	# 设置 groupId
  -DartifactId=my-app \                          		# 设置 artifactId
  -Dversion=1.0 \                                  	# 设置 version（默认 1.0-SNAPSHOT）
  -DarchetypeArtifactId=maven-archetype-webapp \  	# 使用的 Archetype 模板名
  -DinteractiveMode=false                          	# 关闭交互模式，使用默认值快速生成
```

> `maven-archetype-webapp` 模板提供的是 Web 项目的最小骨架，仅包含 `webapp` 目录。缺少 `java`、`resources` 、`test` 等标准结构，需手动补全。

---

## 3. 依赖管理

### 3.1 概述

依赖管理即对 `<dependencies>` 中引入的资源（如 Jar 包）的配置控制，主要包括：

- **依赖配置**：通过 `pom.xml` 中的 `<dependencies>` 标签声明当前项目所需的外部资源（Jar 包）。
- **依赖传递**：引入依赖会自动引入其间接依赖。**冲突时优先路径就近，其次声明顺序**，可通过 `<optional>` 禁用传递，或 `<exclusions>` 显式排除。
- **依赖范围**：通过 `<scope>` 标签设置依赖在编译、运行、测试、打包等阶段的可见性和参与度。

> 依赖配置是“**把书买回来并放到书架上**”【取操作】，`import` 是“**到书架找书看**”【编译器找操作】

### 3.2 依赖范围表（scope）

| Scope               | 编译 | 运行 | 测试 | 传递 | 打包 | 用途说明                            |
| ------------------- | ---- | ---- | ---- | ---- | ---- | ----------------------------------- |
| **compile（默认）** | ✅    | ✅    | ✅    | ✅    | ✅    | **主业务模块**（Spring）            |
| **provided**        | ✅    | ❌    | ✅    | ❌    | ❌    | **容器提供**（servlet-api，Tomcat） |
| **runtime**         | ❌    | ✅    | ✅    | ✅    | ✅    | **编译时用不到**（JDBC 驱动）       |
| **test**            | ❌    | ❌    | ✅    | ❌    | ❌    | **测试框架**（JUnit）               |
| **import**          | ❌    | ❌    | ❌    | ❌    | ❌    | **仅用于 BOM 版本导入**             |

> 接口或功能由容器提供实现的依赖应设置为 `provided`，**因为添加依赖的目的只是为了通过编译，其具体实现由容器在运行时提供**。

---

**依赖管理示例**

```xml
<dependencies>
  <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.0.33</version>
    <scope>runtime</scope>       <!-- 依赖范围 -->
    <optional>true</optional>     <!-- 声明为可选依赖，不传递给下游项目 -->
    <exclusions>                  <!-- 排除上游传递依赖，避免冲突 -->
      <exclusion>
        <groupId>org.slf4j</groupId>	<!-- 不需要version，排除所有版本 -->
        <artifactId>slf4j-api</artifactId>
      </exclusion>
    </exclusions>
  </dependency>
</dependencies>
```

---

## 4. Maven 构建执行机制

### 4.1 生命周期（Lifecycle）

生命周期本质是 Maven 构建流程的抽象组织。**生命周期不可执行，执行的是其内部阶段**。生命周期由三部分组成：

- **clean**：清理产物
- **default**：构建流程（编译、打包等）
- **site**：生成站点文档

### 4.2 阶段（Phase）

每个生命周期由多个阶段组成，阶段按顺序执行。

| 常见阶段     | 所属生命周期 | 说明                      |
| ------------ | ------------ | ------------------------- |
| **clean**    | clean        | 删除构建产物（如 target） |
| **validate** | default      | 校验项目结构和参数        |
| **compile**  | default      | 编译主源码                |
| **test**     | default      | 执行测试                  |
| **package**  | default      | 打包为 jar/war            |
| **install**  | default      | 安装至本地仓库            |
| **deploy**   | default      | 部署至远程仓库            |

> `mvn 阶段` 指令会从该阶段所属生命周期的**起始阶段开始顺序执行**，直到目标阶段，并执行这些阶段中绑定的所有插件目标（goal）。

### 4.3 插件 & 目标（Plugin & Goal）

插件可通过 `<executions>` 标签将目标 `<goals>` 绑定到指定阶段 `<phase>`，在构建过程中自动执行。

- **插件（Plugin）**：构建功能模块，一个插件可包含多个目标（Goal）
- **目标（Goal）**：插件中的具体执行任务，可与生命周期阶段绑定执行

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.example</groupId>
      <artifactId>demo-plugin</artifactId>
      <version>1.0</version>
      <executions>
        <execution>									<!-- 每个 execution 只能绑定一个 phase-->
          <phase>compile</phase>           <!-- 绑定执行阶段 -->
          <goals>
            <goal>do-something</goal>      <!-- 执行目标，可添加多个 <goal> 标签 -->
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

---

## ✅ 总结

- Maven 项目的目录结构是标准化的，包含源码区、测试区、资源目录与输出目录。
- `archetype` 是项目骨架模板，不同 archetype 会影响项目初始结构。
- 依赖配置控制外部资源的引入、范围、传递性等。
- 构建机制由 生命周期 → 阶段 → 插件 → 目标 四层组成，Maven 指令实际是调用某一阶段。
- 实战中掌握 compile/provided/runtime 三种常见 `scope` 足够覆盖绝大多数场景。