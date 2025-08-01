# Day12 · Web基础（文档声明头、Head内元素、HTML语法特性）

本节介绍 HTML 文档声明规范、`<head>` 常用配置（字符集、SEO、标题），以及空白折叠与标签闭合等基础语法特性，重点理解 XHTML 的规范性要求。

## 1. 文档声明头 `<!DOCTYPE>`

任何标准 HTML 页面首行都应以 `<!DOCTYPE ...>` 开头，称为 **文档声明头（DocType Declaration）**。它用于告诉浏览器当前页面采用哪种 HTML 规范（如 HTML、XHTML、HTML5）。

### 1.1 HTML4 与 XHTML 文档类型

HTML4 可分为两类标准：HTML 和 XHTML，均下设三个子规范：

| 类别    | 子规范        | 特性说明 |
|---------|----------------|----------|
| HTML    | Strict         | 不允许使用表现型标签（如 `<u>`） |
|         | Transitional   | 允许使用部分表现型标签 |
|         | Frameset       | 允许使用框架等老旧标签 |
| XHTML   | Strict         | 所有标签必须小写、闭合、属性需加引号 |
|         | Transitional   | XHTML 语法 + 允许部分老标签 |
|         | Frameset       | 最宽松，允许表现型标签和框架结构 |

> 本课程采用 **XHTML 的 Frameset 规范**，语法严格但容错相对宽松，利于控制页面行为。

---

## 2. `<head>` 标签常见配置元素

`<head>` 标签用于声明网页配置信息，浏览器在渲染网页前会优先解析此部分内容。

### 2.1 字符集声明 `<meta charset>`

```html
<meta http-equiv="Content-Type" content="text/html;charset=UTF-8" />
```

- 告诉浏览器：接收到网页字节数据后，应以何种字符集进行解码。
- 推荐使用 `UTF-8`，为国际标准，能兼容多语言。
- 如使用 `GBK`/`GB2312`，需确保目标用户群体为中文环境，否则可能出现乱码。
- 多数编码集向下兼容 ASCII，但与 UTF-8 在中文处理上不兼容。

### 2.2 SEO 配置

网页中的 SEO（Search Engine Optimization，搜索引擎优化）配置有助于提升网站在搜索结果中的可见性

#### 关键词与页面描述

```html
<meta name="description" content="本网页提供食谱的查找功能" />
<meta name="keywords" content="食谱,教程,做饭,美食" />
```

- `description`：提供网页简要描述，可能作为搜索摘要展示。
- `keywords`：早期用于提升搜索排名，现已被主流搜索引擎忽略。
- 两者均不影响页面功能，但有助于 SEO 管理。

#### 页面标题 `<title>`

```html
<title>京东(JD.COM) - 正品低价、品质保障、配送及时、轻松购物！</title>
```

- 显示在浏览器标签页上。
- 是搜索引擎优先抓取的元素之一。

---

## 3. HTML 基本语法特性

### 3.1 空白折叠（Whitespace Collapsing）

HTML 中多个空格、换行、缩进（如 Tab）将被折叠为一个空格。

示例：
```html
<p>test1        多空格
	换行符		tab	test2</p>
```

显示结果：`test1 test2`

> HTML 更关注标签嵌套结构，而非格式排版。

### 3.2 标签必须闭合

XHTML 规范要求所有标签均必须闭合：

- 普通标签用成对闭合：
  ```html
  <p>文本内容</p>
  ```
- 空标签需用 `/` 自闭合：
  ```html
  <img src="xx.jpg" />
  <meta ... />
  ```

> 即使 HTML 对未闭合标签容忍度高，XHTML 依然要求严格闭合。

---

### ✅ 总结

- HTML 文档必须包含文档声明头 `<!DOCTYPE>`，用于指定所采用的标准规范（如 HTML4、XHTML）。
- XHTML 语法更为严谨，要求标签小写、闭合、属性加引号等。
- `<head>` 区用于配置网页信息，包括字符集、标题、关键字、页面描述等，是浏览器渲染前的必要信息。
- 推荐统一使用 `UTF-8` 字符集，避免跨区域用户出现乱码。
- HTML 中空白符（如换行、缩进）默认会折叠为一个空格。
- 在 XHTML 或严谨开发场景下，所有标签都应闭合，空标签需使用 `/` 结尾。