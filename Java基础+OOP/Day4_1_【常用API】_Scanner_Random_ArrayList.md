# Day4 · Java 常用 API（Scanner / Random / ArrayList）

---

## 1. 什么是 API？

Java 提供了大量预构建的类和方法，供开发者复用这些“轮子”来简化编码工作。这些类组成了 Java 的 **标准类库（API）**，通常以包（package）组织形式分发，如 `java.util`、`java.io` 等。

- API 全称：**Application Programming Interface**
- 可通过 Oracle 官方文档快速查阅类说明、方法说明、参数说明、抛出异常等内容。

🔗 [Oracle Java 官方 API 手册](https://docs.oracle.com/en/java/javase/17/docs/api/index.html)

---

## 2. 包与导入（Package & Import）

Java 使用包来组织类。一个包就像一个“功能文件夹”，同一类功能的类通常被放入同一个包中。

```java
import java.util.*;             // 导入 java.util 包下的所有类
import java.util.Scanner;       // 只导入 Scanner 类
import java.util.ArrayList;     // 只导入 ArrayList 类
```

---

## 3. Scanner 类（用户输入）

Scanner 是 Java 用于 **接收控制台输入** 的类，位于 `java.util` 包中。

### ✅ 使用步骤

```java
import java.util.Scanner;

Scanner scanner = new Scanner(System.in); // 接收系统输入
System.out.println("请输入姓名：");
String name = scanner.nextLine();         // 接收整行输入（字符串）
int id = scanner.nextInt();               // 接收整数
double salary = scanner.nextDouble();     // 接收浮点数
```

---

## 4. ArrayList 类（动态数组）

### ✅ 问题背景

- 原生数组大小固定，无法动态扩容
- `ArrayList` 是 Java 提供的**动态数组容器**

### ✅ 定义方式

```java
import java.util.ArrayList;

// ✅ 推荐使用泛型限制数据类型（类型安全）
ArrayList<String> list = new ArrayList<>();      // 仅可存入 String 类型
ArrayList<Integer> nums = new ArrayList<>();     // 仅可存入 Integer 类型

// ⚠️ 原始类型（不使用泛型），不推荐
ArrayList mixedList = new ArrayList();           // 可存入任意引用类型，存在类型混杂风险
mixedList.add("Alice");
mixedList.add(123);  // 编译不报错，但运行时可能因类型转换错误崩溃
```

> ⚠️ ArrayList 不允许使用基本类型，如 `int`，请用对应包装类如 `Integer`

---

### ✅ 常见操作汇总

| 操作           | 示例                                | 说明                     |
|----------------|-------------------------------------|--------------------------|
| 添加元素       | `list.add("Alice");`                | 末尾添加元素             |
| 获取元素       | `list.get(0);`                      | 通过索引获取元素         |
| 替换元素       | `list.set(0, "Bob");`               | 替换指定位置元素         |
| 删除元素       | `list.remove(0);`                   | 删除指定索引的元素       |
| 获取大小       | `list.size();`                      | 当前元素个数             |
| 遍历           | `for (String s : list)`             | 增强 for 遍历元素        |

---

## 5. Random 类（随机数生成）

位于 `java.util` 包下，可生成整数、浮点数等。

```java
import java.util.Random;

Random rand = new Random();
int r1 = rand.nextInt(4);     // [0, 4) 的整数
double r2 = rand.nextDouble(); // [0.0, 1.0) 的小数
```

---

## 6. 匿名对象（了解）

### ✅ 定义

- **匿名对象**：没有赋值给变量名的临时对象。
- 用于**方法参数传入**或**一次性使用场景**。

```java
// 通常写法
Scanner sc = new Scanner(System.in);
return sc;

// 使用匿名对象
return new Scanner(System.in);
```

> 适合用于只用一次、无需命名或复用的场景，提高代码简洁度。

---

## ✅ 小结

- Java 提供了丰富的标准类（API），通过 `import` 导入包使用
- 常用工具类：`Scanner`, `ArrayList`, `Random`
- 推荐：始终使用泛型约束 `ArrayList` 类型，避免运行期错误
- 匿名对象提升代码精简性，但不适合多次复用

---