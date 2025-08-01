# Day11 · Java 提高（缓冲流、转换流、序列化流、打印流）

## 1. 缓冲流（Buffered Stream）

缓冲流是对基础 IO 流的功能增强，提供更高的读写效率。其本质是在读写时**使用内置缓冲区减少磁盘 IO 次数**。

### 1.1 类型分类

- 字节缓冲流：`BufferedInputStream` / `BufferedOutputStream`
- 字符缓冲流：`BufferedReader` / `BufferedWriter`

### 1.2 原理说明

缓冲流在内部维护一个缓冲区数组，数据操作优先在缓冲区进行，当缓冲区满或手动 `flush()` 时再执行实际磁盘 IO 操作，从而**显著减少系统调用次数**。

### 1.3 使用方式

#### ✅ 构造方法

| 类别                   | 构造器                                       | 使用说明                                 |
| ---------------------- | -------------------------------------------- | ---------------------------------------- |
| `BufferedInputStream`  | `new BufferedInputStream(InputStream in)`    | 为字节输入流添加缓冲区                   |
| `BufferedOutputStream` | `new BufferedOutputStream(OutputStream out)` | 为字节输出流添加缓冲区                   |
| `BufferedReader`       | `new BufferedReader(Reader in)`              | 为字符输入流添加缓冲与 `readLine()` 能力 |
| `BufferedWriter`       | `new BufferedWriter(Writer out)`             | 为字符输出流添加缓冲与 `newLine()` 支持  |

##### `BufferedInputStream` 字节缓冲输入流

```java
InputStream fis = new FileInputStream("a.txt");
BufferedInputStream bis = new BufferedInputStream(fis);
```

##### `BufferedOutputStream` 字节缓冲输出流

```java
OutputStream fos = new FileOutputStream("b.txt");
BufferedOutputStream bos = new BufferedOutputStream(fos);
```

##### `BufferedReader` 字符缓冲输入流

```java
BufferedReader br = new BufferedReader(new FileReader("a.txt"));
String line = br.readLine(); // 读取一整行，不包含换行符
```

##### `BufferedReader` 字符缓冲输出流

```java
BufferedWriter bw = new BufferedWriter(new FileWriter("b.txt"));
bw.newLine(); // 写入换行符，系统相关
```

> 即使普通字节流使用了数组读写，其每次 `read()` 和 `write()` 仍可能触发磁盘 IO，而缓冲流通过**一次性读取大块数据进入内存再处理**，显著提升性能。

---

## 2. 转换流（InputStreamReader / OutputStreamWriter）

### 2.1 概念说明

转换流是连接**字节流与字符流之间的桥梁**，用于处理不同编码之间的转换。

- `InputStreamReader`：字节 → 字符（解码）
- `OutputStreamWriter`：字符 → 字节（编码）

这是**单向桥梁**，不可反向复用。

### 2.2 使用场景

当文件编码非 UTF-8 时，使用 `FileReader`/`FileWriter` 可能读取乱码，转换流可通过显式指定编码方式解决该问题。

#### ✅ 构造方法

| 类别                 | 构造器                                                       | 使用说明                |
| -------------------- | ------------------------------------------------------------ | ----------------------- |
| `InputStreamReader`  | `new InputStreamReader(InputStream in[, String charset])`    | 字节 → 字符（指定编码） |
| `OutputStreamWriter` | `new OutputStreamWriter(OutputStream out[, String charset])` | 字符 → 字节（指定编码） |

##### `InputStreamReader`

```java
InputStreamReader isr = new InputStreamReader(new FileInputStream("a.txt"), "GBK");	// 指定读取文件的编码格式为 GBK
BufferedReader br = new BufferedReader(isr);
```

##### `OutputStreamWriter`

```java
OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream("b.txt"), "UTF-8");	// 指定写入文件的编码格式为 UTF-8
BufferedWriter bw = new BufferedWriter(osw);
```

---

## 3. 字符编码与字符集

### 3.1 基本概念

- **编码**：字符 → 二进制
- **解码**：二进制 → 字符
- **字符集**：编码规则的集合

### 3.2 常见字符集

| 字符集 | 特点 |
|--------|------|
| ASCII | 基础英文，7bit 表示 128 个字符 |
| ISO-8859-1 | 扩展 ASCII，8bit，支持西欧字符 |
| GB2312 | 简体中文，双字节编码，支持 7000+ 汉字 |
| GBK | 扩展 GB2312，支持 21000+ 汉字 |
| GB18030 | 国家标准，兼容 GBK，支持 7 万+ 汉字 |
| Unicode | 万国码，支持所有语言 |
| UTF-8 | 可变长编码：ASCII 1 字节，汉字 3 字节，Emoji 最多 4 字节 |

---

## 4. 对象序列化与反序列化

### 4.1 概念

- **序列化**：将对象转换为字节流以便存储或传输（`ObjectOutputStream`）
- **反序列化**：从字节流还原对象（`ObjectInputStream`）

#### ✅ 构造方法

| 类别                 | 构造器                                     | 使用说明                  |
| -------------------- | ------------------------------------------ | ------------------------- |
| `ObjectOutputStream` | `new ObjectOutputStream(OutputStream out)` | 对象 → 字节，序列化写入   |
| `ObjectInputStream`  | `new ObjectInputStream(InputStream in)`    | 字节 → 对象，反序列化恢复 |

##### `ObjectOutputStream` 序列化

```java
// 写入对象到文件
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("person.obj"));
oos.writeObject(new Person("Jack", 20));
oos.close();
```

##### `ObjectInputStream` 反序列化

```java
// 从文件读取对象
ObjectInputStream ois = new ObjectInputStream(new FileInputStream("person.obj"));
Person p = (Person) ois.readObject();
ois.close();
```

### 4.2 实现条件

- 类必须实现 `Serializable` 接口（标记型接口）
- 可使用 `transient` 排除敏感字段
- `static` 字段不会被序列化

### 4.3 注意事项

- 方法不会被序列化
- 反序列化后的 `static` 字段使用当前类值，`transient` 字段变为默认值
- 可通过自定义 `readObject(ObjectInputStream in)` 方法恢复 `transient` 字段

### 4.4 多对象处理方式

- 推荐在写入多个对象前，先使用 `writeInt(count)` 写入对象数量，再逐个调用 `writeObject(obj)` 写入
- 或者使用一次性写入对象数组的方式（即整体快照写入 `writeObject(objArray)`）
- 不建议依赖 `EOFException` + `while(true)` 的组合方式作为对象读取终止条件，因为这种做法依赖异常机制控制流程，可读性差、难维护、不符合最佳实践

---

## 5. serialVersionUID（版本兼容控制）

反序列化时会比对 UID，如果不一致会抛出 `InvalidClassException`。建议手动指定：

```java
private static final long serialVersionUID = 1L;
```

---

## 6. 打印流（PrintStream）

用于简化文本输出，常见于 `System.out`，可通过重定向打印到文件。

#### ✅ 构造方法

| 类别          | 构造器                                      | 使用说明                                                     |
| ------------- | ------------------------------------------- | ------------------------------------------------------------ |
| `PrintStream` | `new PrintStream(File|String|OutputString)` | 创建可打印流对象，可打印各种数据到文件或输出流，支持 `print()` / `println()` |

```java
PrintStream ps = new PrintStream("output.txt");
System.setOut(ps);
System.out.println("Hello"); // 输出到 output.txt
```

---

## ✅ 总结

- 缓冲流通过内存缓冲提升 IO 性能，是对基础流的增强封装。
- 转换流用于编码转换，是字符流与字节流之间的桥梁，必须明确方向。
- 序列化用于对象持久化，要求类实现 `Serializable` 且明确排除不需要序列化的字段（如 `static`、`transient`）。
- 多对象序列化应避免依赖异常控制结构。
- `serialVersionUID` 用于序列化版本校验，推荐手动维护。
- 打印流可用于重定向控制台输出，是常用的简易输出方案。
