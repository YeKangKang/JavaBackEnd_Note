# Day10 · Java 提高（File类、路径、递归、过滤器）

本节内容讲解 Java 中 `File` 类的使用方式、绝对路径与相对路径的区别、递归的三步法模型、文件递归遍历与搜索结构，以及 `FileFilter` 接口与 Lambda 表达式在文件过滤中的应用。

---

## 1. File 类构造与路径

### 1.1 File 构造方式

`File` 表示文件或目录路径的抽象。创建对象时**不检查路径是否存在，也不会创建实际文件**。

| 构造方法 | 示例 | 说明 |
|----------|------|------|
| `File(String pathname)` | `new File("src/Hello.java")` | 通过完整路径字符串构造 |
| `File(String parent, String child)` | `new File("src", "Hello.java")` | 父路径 + 子路径字符串 |
| `File(File parent, String child)` | `new File(parentDir, "Hello.java")` | 父 File 对象 + 子路径 |

---

### 1.2 路径类型说明

- **绝对路径**：从磁盘根目录开始，如 `C:/Users/xxx`
- **相对路径**：相对于项目根目录，如 `src/data.txt`
- 默认 IDE 中相对路径以项目文件夹为根目录

---

## 2. File 常用方法

### 2.1 路径与文件信息获取

| 方法 | 说明 |
|------|------|
| `getAbsolutePath()` | 返回绝对路径 |
| `getPath()` | 返回构造时路径（原始） |
| `getName()` | 返回文件/目录名 |
| `length()` | 返回文件大小（字节）|

### 2.2 判断功能

| 方法 | 说明 |
|------|------|
| `exists()` | 文件/目录是否真实存在 |
| `isFile()` | 是否为文件 |
| `isDirectory()` | 是否为目录 |

### 2.3 创建与删除

| 方法 | 说明 |
|------|------|
| `createNewFile()` | 创建文件（若已存在则失败） |
| `delete()` | 删除文件或空目录 |
| `mkdir()` | 创建单层目录 |
| `mkdirs()` | 创建多级目录（含父级） |

> 删除目录前需保证内部为空，否则删除失败。

### 2.4 目录遍历

| 方法 | 返回类型 | 说明 |
|------|-----------|------|
| `list()` | `String[]` | 返回文件名字符串数组 |
| `listFiles()` | `File[]` | 返回子文件或子目录的 File 对象 |

> 若路径不存在或不是目录，`list()` / `listFiles()` 返回 `null`，建议使用前先判断 `file.exists()`。

---

## 3. 递归

### 3.1 递归概念

递归指方法调用自身，可分为：

- **直接递归**：自身调用自身
- **间接递归**：方法 A → 方法 B → 方法 A

常用于树结构、分治结构、搜索问题中。

---

### 3.2 编写递归的三步法

1. **明确目标**：该方法要解决什么问题，返回什么
2. **设置终止条件**：什么时候结束递归，边界返回什么
3. **本层逻辑 + 子问题递归调用**：如何处理当前 + 交给递归处理其余

---

### 3.3 常见递归示例

#### ✅ 递归求和

```java
public static int sum(int n) {
    if (n <= 1) return 1;
    return sum(n - 1) + n;
}
```

#### ✅ 递归求阶乘

```java
public static int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
```

---

## 4. 文件与目录递归操作

#### ✅ 打印所有目录和文件（递归遍历）

```java
public static void printAll(File dir) {
    for (File item : dir.listFiles()) {
        System.out.println(item.getAbsolutePath());
        if (item.isDirectory()) {
            printAll(item);
        }
    }
}
```

- **目标**：打印指定路径下的所有文件和目录（含子目录）
- **终止条件**：`listFiles()` 返回为空时自动终止
- **子问题**：对子目录递归调用自身，继续遍历其内容

---

#### ✅ 搜索目标文件（递归匹配）

```java
public static boolean find(File dir, String targetName) {
    for (File item : dir.listFiles()) {
        if (item.isFile() && item.getName().equals(targetName)) return true;
        if (item.isDirectory() && find(item, targetName)) return true;
    }
    return false;
}
```

- **目标**：查找指定路径下是否存在目标文件
- **终止条件**：所有子项都未命中（`for` 扫完，递归层层返回 false）
- **子问题**：若当前项是目录，递归进入其内部继续查找

---

## 5. FileFilter 文件过滤器

### 5.1 接口介绍

`FileFilter` 是函数式接口，仅包含一个方法：

```java
boolean accept(File pathname);
```

**用于配合 `listFiles(FileFilter)` 方法筛选文件**，如只保留 `.java` 文件和目录。

```java
// 使用匿名内部类方式定义过滤器
FileFilter filter = new FileFilter() {
    @Override
    public boolean accept(File pathname) {
        return pathname.isDirectory() || pathname.getName().endsWith(".java");
    }
};
```

> `FileFilter` 的 `accept()` 方法返回 `true` 表示保留该文件，返回 `false` 表示过滤掉该文件。

---

### 5.2 Lambda 表达式简化用法

```java
FileFilter filter = f -> f.getName().endsWith(".java") || f.isDirectory();
File[] list = dir.listFiles(filter);
```

- 筛选 `.java` 文件 + 所有目录
- 可配合递归使用
- 因为 `FileFilter` 是函数式接口，Lambda 简写语法完全合法

---

## ✅ 总结

- `File` 类用于文件路径抽象与操作，支持构造、判断、创建与删除
- 遍历目录内容需使用 `list()` / `listFiles()`，多级结构建议使用递归
- 所有递归问题均可使用“目标 → 终止 → 子问题”三步法设计
- `FileFilter` 可配合递归按条件筛选文件，使用 Lambda 表达式可进一步简化逻辑