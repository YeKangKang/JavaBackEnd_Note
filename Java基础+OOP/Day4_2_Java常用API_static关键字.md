# Day4 · Java 常用 API（String、Static、Arrays、Math）

## 1. String 类详解（java.lang.String）

### 1.1 字符串的不可变特性

- `String` 是不可变类（immutable），**一旦创建，其值不可更改**。
- 任何对字符串的修改（如拼接、替换）都会生成新的 `String` 对象。
- 所有字符串字面值（如 `"abc"`）都存储在 **字符串常量池** 中。
- 示例：
  ```java
  String s = "abc";
  s = s + "def"; // 实际是创建了一个新字符串对象 "abcdef"
  ```

### 1.2 字符串共享示例

```java
String str1 = "abc";
String str2 = "abc";
System.out.println(str1 == str2); // true，指向同一常量池对象
```

### 1.3 字符串创建方式

| 方式         | 示例                                            | 特点                               |
|--------------|-------------------------------------------------|------------------------------------|
| 字面量       | `String s = "abc";`                             | 推荐方式，使用常量池              |
| `char[]` 数组 | `new String(new char[]{'a','b','c'})`          | 创建新对象，不使用常量池          |
| `byte[]` 数组 | `new String(new byte[]{97,98,99})`             | 与字符编码相关，需注意默认编码     |

---

## 2. String 常用方法

### 2.1 字符串判断与比较

| 方法                    | 示例                              | 说明               |
|-------------------------|-----------------------------------|--------------------|
| `equals()`              | `s1.equals(s2)`                   | 区分大小写比较     |
| `equalsIgnoreCase()`    | `s1.equalsIgnoreCase(s2)`         | 忽略大小写比较     |

### 2.2 字符串内容操作

| 方法                    | 示例                              | 说明                        |
|-------------------------|-----------------------------------|-----------------------------|
| `length()`              | `s.length()`                      | 字符个数                    |
| `concat(str)`           | `s.concat("abc")`                 | 拼接字符串                  |
| `charAt(index)`         | `s.charAt(0)`                     | 指定索引字符                |
| `indexOf(ch/str)`       | `s.indexOf("a")`                  | 首次出现位置（-1 不存在）   |
| `substring(begin, end)` | `s.substring(1, 4)`               | 截取子串，左闭右开           |
| `replace(old, new)`     | `s.replace("ab", "xy")`           | 替换内容，生成新字符串      |

### 2.3 字符串转化

| 方法            | 示例                    | 说明                |
|-----------------|-------------------------|---------------------|
| `toCharArray()` | `s.toCharArray()`       | 转为 `char[]`       |
| `getBytes()`    | `s.getBytes()`          | 转为 `byte[]`       |
| `split()`       | `s.split(" ")`          | 返回 `String[]` 数组 |

---

## 3. static 关键字进阶理解

### 3.1 核心概念

| 特性       | 说明                                                                 |
|------------|----------------------------------------------------------------------|
| 类加载时创建 | `static` 成员随着类加载一起创建，只存在一份                          |
| 属于类本身   | 可通过类名访问，无需创建对象                                          |
| 生命周期长   | 生命周期与类相同，不随对象销毁而消失                                 |
| 不可访问非静态 | 因为未绑定具体对象（无 `this`）                                      |

### 3.2 static 变量（类变量）

```java
public class Student {
    static int count = 0;     // 所有 Student 对象共享
    String name;
    public Student(String name) {
        this.name = name;
        count++;
    }
}
```

### 3.3 static 方法（类方法）

```java
public class Tool {
    public static void greet() {
        System.out.println("Hello!");
    }
}
Tool.greet(); // 通过类名调用
```

### 3.4 static 代码块（静态初始化块）

```java
static {
    System.out.println("静态代码块运行");
}
```

- 随类加载执行一次，常用于初始化静态资源。

---

## 4. Arrays 工具类（java.util.Arrays）

### 4.1 常用静态方法

| 方法                           | 示例                               | 说明                     |
|--------------------------------|------------------------------------|--------------------------|
| `Arrays.toString(array)`       | `Arrays.toString(arr)`             | 打印数组内容             |
| `Arrays.sort(array)`           | `Arrays.sort(arr)`                 | 原地升序排序             |
| `Arrays.binarySearch(array, k)`| `Arrays.binarySearch(arr, 5)`      | 二分查找（需先排序）     |

---

## 5. Math 工具类（java.lang.Math）

### 5.1 常用静态方法

| 方法            | 示例              | 说明                     |
| --------------- | ----------------- | ------------------------ |
| `Math.abs(x)`   | `Math.abs(-5)`    | 绝对值                   |
| `Math.ceil(x)`  | `Math.ceil(2.3)`  | 3，向上取整              |
| `Math.floor(x)` | `Math.floor(2.9)` | 2，向下取整              |
| `Math.round(x)` | `Math.round(2.5)` | 四舍五入返回 `long` 类型 |

---

## ✅ 总结

- `String` 是不可变对象，设计成共享常量，利于性能与线程安全。
- `static` 让成员绑定到类本身，适用于工具方法、常量等。
- `Arrays` 与 `Math` 是典型的静态工具类，用于简化开发工作。

