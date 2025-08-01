# Day10 · Java 提高（I/O、字节流、字符流、流异常处理、Properties集合）

本节系统系统化梳理 Java IO 流的输入输出机制、字节流与字符流的使用方式、异常处理手段以及 `Properties` 配置集的核心用法，适用于文本与非文本文件的读写操作基础与进阶。

---

## 1. Java IO 流模型与分类

### 1.1 IO 的核心思想

Java 中的 IO 操作可以理解为**数据在内存与外部设备之间的流动**：

- 输入流（Input）：从设备 → 内存（如读文件、接收网络数据）
- 输出流（Output）：从内存 → 设备（如写文件、发送数据）

---

### 1.2 基本IO类型

| 类别     | 输入流                     | 输出流                      |
|----------|----------------------------|-----------------------------|
| 字节流   | `InputStream`              | `OutputStream`              |
| 字符流   | `Reader`                   | `Writer`                    |

> **字节流适用于所有类型数据**（文本、图片、视频等）

> **字符流仅用于处理“文本文件”**（支持字符集解析）

---

## 2. 字节流

### 2.1 OutputStream / FileOutputStream

#### 构造方式

```java
new FileOutputStream(File file[, boolean append])	// 不写 append 默认为 false
new FileOutputStream(String name[, boolean append])
```

- 若文件不存在：自动创建文件
- 若路径不存在 / 是目录：抛出异常
- `append = true`：追加写入；默认 false 会清空原内容

#### 常用方法

| 方法 | 说明 |
|------|------|
| `write(int b)` | 写入单字节（只写入低 8 位） |
| `write(byte[] b)` | 写入整个数组 |
| `write(byte[] b, int off, int len)` | 写入子数组，从索引 `off` 开始，连续取 `len` 个字节 |
| `flush()` | 刷新输出流（某些缓冲流有效） |
| `close()` | 关闭流，释放资源 |

#### 示例

```java
fos.write(97); // 'a'
fos.write("abc".getBytes()); // 写入 abc
fos.write("abcdefg".getBytes(), 3, 3); // 写入 def
```

> 若需写入换行，Unix/macOS 用 `\n`，Windows 用 `\r\n`；其中 `\r` 表示回车。
> ✅ 建议使用 `System.lineSeparator()` 获取平台兼容的换行符。

---

### 2.2 InputStream / FileInputStream

#### 构造方式

```java
new FileInputStream(File file)
new FileInputStream(String name)
```

- 文件必须真实存在
- 若是目录或路径不存在，抛出 `FileNotFoundException`

#### 常用方法

| 方法 | 说明 |
|------|------|
| `read()` | 读取一个字节（返回 `int`），读到文件末尾 `EOF` 返回 `-1` |
| `read(byte[] b)` | 读取多个字节到数组中，读到文件末尾 `EOF` 返回 `-1` |
| `close()` | 关闭流 |

#### 示例

```java
int data = fis.read(); // 读取单字节
System.out.println((char) data);
```

```java
byte[] buffer = new byte[1024];
int len;
while ((len = fis.read(buffer)) != -1) {	// 读取一组字节
    System.out.println(new String(buffer, 0, len));
}
```

> 若直接输出 new String(buffer)，最后一轮未完全覆盖的旧数据可能残留。需配合 read() 返回值 len 控制打印长度。

---

### 2.3 字节流的写入与写出（复制文件案例）

```java
InputStream in = new FileInputStream("src.png");
OutputStream out = new FileOutputStream("dest.png");

byte[] buffer = new byte[1024];
int len;
while ((len = in.read(buffer)) != -1) {
    out.write(buffer, 0, len);
}
in.close();
out.close();
```

---

## 3. 字符流

### 3.1 Reader / Writer 抽象类与 FileReader / FileWriter

| 类别 | 构造器 | 使用说明 |
|------|--------|----------|
| `FileReader` | `new FileReader(File|String)` | 读取字符文本文件 |
| `FileWriter` | `new FileWriter(File|String[, append])` | 写入文本文件，支持追加 |

- 目标必须是文本文件；对于 Reader 类，文件必须存在
- 若是目录或路径不存在，抛出 `FileNotFoundException`
- `append = true`：追加写入；默认 false 会清空原内容

---

### 3.2 Reader / FileReader

#### 常用方法

| 方法 | 说明 |
|------|----------|
| `read()` | 读取单字符，返回 `int`，读到文件末尾 `EOF` 返回 `-1` |
| `read(char[] cbuf)` | 读取多个字符到数组中，返回读取长度，`EOF` 返回 `-1` |
| `close()` | 关闭流，释放资源 |

#### 示例

```java
FileReader fr = new FileReader("a.txt");
char[] buf = new char[1024];
int len;
while ((len = fr.read(buf)) != -1) {
    System.out.println(new String(buf, 0, len));
}
fr.close();
```

---

### 3.3 Writer / FileWriter

#### 常用方法

| 方法 | 说明 |
|------|----------|
| `write(int c)` | 写入单字符（`int` 会转为 `Unicode` 字符） |
| `write(char[] cbuf)` | 写入字符数组 |
| `write(char[] cbuf, int off, int len)` | 写入数组片段，从索引 `off` 开始，写入 `len` 个字符 |
| `write(String str)` | 写入字符串 |
| `write(String str, int off, int len)` | 写入字符串片段 |
| `flush()` | 刷新字符缓冲区 |
| `close()` | 关闭流，释放资源 |

#### 示例

```java
FileWriter fw = new FileWriter("a.txt", true);

// 写入单字符 'a'（int → Unicode 字符）
fw.write(97);

// 写入字符串片段："程序"
fw.write("程序员", 0, 2);

// 写入数组片段，从索引 2 开始写 4 个字符（"Java"）
char[] data = {'学', '习', 'J', 'a', 'v', 'a'};
fw.write(data, 2, 4);

fw.flush();
fw.close();
```

---

## 4. 异常处理与自动资源释放

### 4.1 传统 try-catch-finally 写法

```java
FileReader fr = null;
try {
    fr = new FileReader("a.txt");
} catch (IOException e) {
    e.printStackTrace();
} finally {
    try { if (fr != null) fr.close(); } catch (IOException e) {}
}
```

---

### 4.2 自动资源释放（JDK 7+）

```java
try (FileReader fr = new FileReader("a.txt")) {	// 资源对象必须实现 AutoCloseable 接口
    ...
} catch (IOException e) {
    ...
}
```

- `try()` 中声明的资源对象必须实现 `AutoCloseable` 接口（如 `FileReader`、`BufferedWriter` 等）
- 可声明多个资源，使用 `;` 分隔
- JDK 9+ 后支持将已在 `try` 外部声明的 `AutoCloseable` 资源引用放入 `try()` 中，仍可实现自动关闭：

```java
FileReader fr = new FileReader("a.txt");
try (fr) {
    ...
}
```

---

## 5. Properties 配置集

### 5.1 概念

- `Properties` 是 `Hashtable<String, String>` 子类
- 常用于读取 `.properties` 配置文件
- 支持从 `InputStream` 或 `Reader` 加载

### 5.2 构造方法

```java
new Properties();	// 构造一个空的 Properties 对象
```

### 5.3 常用方法

| 方法 | 说明 |
|------|------|
| `setProperty(String k, String v)` | 添加属性 |
| `getProperty(String k)` | 获取属性值 |
| `stringPropertyNames()` | 获取全部 key |
| `load(InputStream / Reader)` | 从输入流中加载 `.properties` 配置文件 |

---

### 5.4 示例：从文件中加载

```java
Properties props = new Properties();
try (Reader r = new FileReader("config.properties")) {
    props.load(r);	// 加载外部 .properties 文件内容到内存
    for (String key : props.stringPropertyNames()) {
        System.out.println(key + "=" + props.getProperty(key));
    }
} catch (IOException e) {
    ...
}
```

---

## ✅ 总结

- 字节流适用于任何数据（尤其是非文本）
- 字符流适合处理编码明确的文本文件
- 所有流都应手动关闭或使用 try-with-resources
- Properties 是读取外部配置数据的常用结构，搭配流一起使用最常见