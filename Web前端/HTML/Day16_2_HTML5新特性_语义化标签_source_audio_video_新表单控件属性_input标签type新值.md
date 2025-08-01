# Day16 · HTML5 新特性（语义化标签、source、audio、video、新表单控件属性、input标签type新值）

HTML5 是 HTML 的第五次重大修订，定义了新的标签、属性和表单特性，合并了 HTML + CSS3 + JavaScript 成为强大的前端技术集合。

---

## 1. 语义化符号标签

HTML5 新增了大量语义化结构标签，用于更清晰地描述页面模块。目的是提升 SEO ，同时让页面在网页样式失效时仍能完整显示结构。

### 常见标签列表

| 标签     | 用途                    |
|------------|-----------------------------|
| `<header>` | 页面或模块头部       |
| `<nav>`    | 导航区域                |
| `<main>`   | 唯一主体内容区      |
| `<section>`| 分区内容（带主题）    |
| `<article>`| 独立完整内容段      |
| `<aside>`  | 侧边附助内容         |
| `<footer>` | 页面或区块尾部       |
| `<figure>` | 图片/多媒体容器     |
| `<figcaption>` | 图片描述          |
| `<time>`   | 时间或日期标识       |
| `<mark>`   | 高亮内容（如搜索关键词） |

---

## 2. `<source>` 标签

- 用于 `<audio>` 或 `<video>` 提供多格式备选资源
- 自闭合标签，不允许子内容
- 只有首个被支持的资源会被播放，其余被忽略
- 常用属性：
  - `src` : 资源路径
  - `type`: 媒体格式类型（如 `audio/mp3`、`video/mp4`、`audio/ogg`）

```html
<source src="video.mp4" type="video/mp4">
```

---

## 3. `<audio>` 音频标签

- HTML5 新增，`inline-block` 元素，但**不支持设置宽高**
- 内部只允许 `<source>` 和文本 fallback 
- 使用 `<source>` 时优先选择支持的资源播放

### 常用属性

| 属性      | 类型   | 作用                          |
|-----------|--------|-----------------------------------|
| `src`     | URL    | 音频文件地址                   |
| `autoplay`| 布尔  | 加载后自动播放                |
| `controls`| 布尔  | 显示播放控件                  |
| `loop`    | 布尔  | 播放完后循环                  |

### 示例

```html
<!-- 直接使用 <audio> 标签 -->
<audio src="music.mp3" controls>
  您的浏览器不支持播放音频
</audio>

<!-- 或者内部嵌套 <source> 标签组 -->
<audio controls>
  <source src="music.mp3" type="audio/mpeg">
  <source src="music.ogg" type="audio/ogg">
  您的浏览器不支持播放音频
</audio>
```

---

## 4. `<video>` 视频标签

- HTML5 新增，`inline-block`元素，**支持设置宽高**
- 支持 `<source>` 提供多格式备选和文本 fallback 
- 使用 `<source>` 时优先选择支持的资源播放

### 常用属性

| 属性        | 类型   | 作用                                     |
|---------------|--------|--------------------------------------------|
| `src`         | URL    | 视频地址                                 |
| `width`（建议使用CSS） | 数字    | 设置宽度，等价于 CSS 样式属性             |
| `height`（建议使用CSS） | 数字    | 设置高度                              |
| `autoplay`    | 布尔  | 自动播放                                |
| `controls`    | 布尔  | 显示播放控件                            |
| `loop`        | 布尔  | 循环播放                                |
| `muted`       | 布尔  | 静音播放（可配合 autoplay 使用）     |
| `preload`     | 文本   | 资源预加载策略：`auto`，`none`，`metadata` |
| `poster`      | URL    | 等待加载时显示的图片                      |

### 示例

```html
<!-- 直接使用 <video> 标签 -->
<video src="video.mp4" controls autoplay muted loop poster="cover.jpg">
  您的浏览器不支持播放视频
</video>

<!-- 或者内部嵌套 <source> 标签组 -->
<video controls autoplay muted loop poster="cover.jpg">
  <source src="video.mp4" type="video/mp4">
  <source src="video.ogg" type="video/ogg">
  您的浏览器不支持播放视频
</video>
```

---

## 5. HTML5 表单新特性

### 5.1 表单控件属性

以下属性用于表单控件（如 `input`、`select`、`textarea`）：

| 属性         | 类型   | 作用                                                         |
|------------------|--------|------------------------------------------------------------------|
| `required`       | 布尔  | 必填，控件未填值时无法提交                            |
| `placeholder`    | 文本   | 灰色提示文本，展示在控件中，不会被提交     |
| `autocomplete`   | on/off| 是否开启历史输入智能表单填充                    |
| `autofocus`      | 布尔  | 页面加载后自动聚焦                          |
| `readonly`       | 布尔  | 只读，用户不可编辑                            |
| `disabled`       | 布尔  | 禁用，无法选择，不被表单提交              |
| `multiple`       | 布尔  | 允许多选（选择框 / 文件上传）               |

---

### 5.2 `<input>` type 新值

HTML5 对 `<input>` 的 `type` 属性增加了多种类型，用于进一步维护表单输入的合法性和描述义：

| 新类型       | 描述                       |
|---------------|------------------------------|
| `email`       | 邮箱地址校验                |
| `url`         | URL 地址校验               |
| `tel`         | 手机号（验证松散）     |
| `number`      | 数值选择控件              |
| `range`       | 滑动条输入数值           |
| `search`      | 搜索类输入，可清除     |
| `color`       | 颜色选择器                |
| `date`        | 日期选择器（年月日）  |
| `time`        | 时间选择器（时分秒）  |
| `month`       | 月份选择（年+月）     |
| `week`        | 周选择（年+第几周）   |

---

## ✅ 总结

- HTML5 新增大量语义化符号标签，提升结构清晰度和 SEO 友好度
- `<audio>` 和 `<video>` 是原生多媒体播放器，支持 fallback 和优先级资源
- `<source>` 用于提供多格式备选，只播放首个被支持的资源
- HTML5 表单增强了控件类型和表现性，加强强校验、简化操作
