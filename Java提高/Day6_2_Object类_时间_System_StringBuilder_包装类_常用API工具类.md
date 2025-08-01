# Day6 · Java 提高（Object 、Date、Calendar、DateFormat、System、StringBuilder、包装类、包装类和基本类型转换）

---

## 1. Object 类（java.lang.Object）

Java 所有类的父类，是一个普通的具体类（不是抽象类，也不是接口）。若类未显式继承其他类，则默认继承自 `Object`。常用方法包括：

### 1.1 `toString()`

默认实现：
```java
getClass().getName() + "@" + Integer.toHexString(hashCode())
```

- 表示类名 + 对象在堆内的哈希值（无实际业务意义）
- 直接打印对象时默认调用 `toString()` 方法
- 一般开发中建议重写 `toString()`，返回对象关键属性信息

### 1.2 `equals(Object obj)`

默认实现与 `==` 一致，比较是否为同一个对象（引用地址是否相同）。

可重写实现业务层逻辑判断相等性，通用结构如下：

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;                            // 引用相同，直接相等
    if (o == null || getClass() != o.getClass()) return false; // null 或类型不一致，判定不等
    MyClass other = (MyClass) o;                           // 类型一致后可安全强转
    return this.x == other.x && this.y == other.y;         // 比较关键字段值
}
```

### 1.3 `hashCode()`

返回当前对象的哈希值，默认基于对象的地址计算，主要用于集合类中的哈希分布（如 `HashMap`、`HashSet`）

```java
@Override
public int hashCode() {
    return Objects.hash(x, y); // 用字段生成哈希值，确保与 equals 判定逻辑一致
}
```

> **若重写了 `equals()`，必须一并重写 `hashCode()`**，以保持对象在集合中的一致性（否则会导致查找、去重异常）

> Java 集合类如 `HashSet`、`HashMap` 会先通过 `hashCode()` 定位桶，再通过 `equals()` 判重

---

## 2. Objects 类（java.util.Objects）

工具类，提供**空指针安全**的对象操作方法。

### 2.1 `equals(Object a, Object b)`

- 若 `a == b`，返回 `true`
- 否则若 `a != null`，调用 `a.equals(b)`
- 否则返回 `false`

> 虽然 `Objects.equals(a, b)` 避免了空指针异常，但**相等性仍由 `a.equals(b)` 决定**，必须在其中先判断是否为同一类，再类型转换并进行自定义逻辑判断。

### 2.2 `hash(Object...)`
- 计算多个字段组合的哈希值，常用于 `hashCode()` 实现
- 接收可变参数，自动处理 `null`，避免手写拼接逻辑

```java
@Override
public int hashCode() {
    return Objects.hash(name, age, gender);
}
```
> hashCode() 默认基于对象的内存地址计算，若希望基于字段内容判断“相等性”，则应重写该方法。

> 尤其当对象将被放入如 HashSet、HashMap 等集合中，Java 会优先通过 hashCode() 判断对象是否重复，再根据 equals() 精确判断。
> 因此，hashCode() 的字段应与 equals() 中参与比较的字段保持一致，否则会导致集合行为异常（如无法去重、查找失败）。

---

## 3. Date 类（java.util.Date）

表示时间点，精确到毫秒。底层存储为时间戳（long 类型）。

### 构造方法

```java
new Date();          // 当前系统时间
new Date(long time); // 指定时间戳
```

- 可用 `getTime()` 获取时间戳（**1970 起的毫秒数**）

---

## 4. DateFormat 与 SimpleDateFormat

抽象类 `DateFormat` 用于 Date/String 相互转换，常用子类是 `SimpleDateFormat`。

### 构造 SimpleDateFormat 对象

```java
DateFormat sdf = new SimpleDateFormat(String pattern); // 如 "yyyy-MM-dd HH:mm:ss"
```

### 方法

```java
sdf.format(Date date);      // Date → String
sdf.parse(String source);   // String → Date（格式不匹配会抛异常）
```

> 格式符示例：`y年`、`M月`、`d日`、`H时`、`m分`、`s秒`、`E星期`、`a上午/下午` 等

---

## 5. Calendar 类（java.util.Calendar）

替代部分 `Date` 功能的抽象类，包含对年月日等字段的读写操作。

### 使用方式

```java
Calendar c = Calendar.getInstance(); // 获取 GregorianCalendar 实例
```

### 常用方法

```java
c.get(Calendar.YEAR);
c.set(Calendar.MONTH, 5);
c.add(Calendar.DATE, -3);
```

> 内部通过 `public static final int` 常量标识字段（如 `Calendar.YEAR`、`Calendar.MONTH`），增强可读性

---

## 6. System 类（java.lang.System）

提供系统级别操作的工具类，常用方法如下：

### 6.1 `currentTimeMillis()`

```java
System.currentTimeMillis(); // 当前时间戳（毫秒），与 new Date().getTime() 等价
```

### 6.2 `arraycopy(...)`

```java
System.arraycopy(src, srcPos, dest, destPos, length);
```

将数组内容从 `src` 拷贝到 `dest` 中，适用于性能敏感场景。

---

## 7. StringBuilder 类

可变字符串容器，适用于频繁拼接字符串的场景。

### 构造方法

```java
new StringBuilder();
new StringBuilder(String str);
```

### 拼接方法

```java
sb.append(...); // 多个重载，自动转为字符串
```

支持链式调用：

```java
sb.append("abc").append(123).append(true);
```

### 与 String 互转

```java
new StringBuilder(str); // String → StringBuilder
sb.toString();          // StringBuilder → String
```

---

## 8. 包装类（Wrapper Class）

基本类型 → 引用类型的映射关系：

| 基本类型  | 包装类      |
| --------- | ----------- |
| `int`     | `Integer`   |
| `char`    | `Character` |
| `boolean` | `Boolean`   |
| ...       | ...         |

---

### 装箱与拆箱

- 装箱：基本 → 包装对象
- 拆箱：包装对象 → 基本

```java
Integer i = Integer.valueOf(3); // 手动装箱
int n = i.intValue();           // 手动拆箱
```

### 自动装箱与拆箱（JDK 1.5+）

```java
Integer i = 4;      // 自动装箱
i = i + 5;          // 自动拆箱后计算，再装箱
```

---

## 9. 基本类型与 String 的互转

### 9.1 基本类型 → String

```java
int a = 10;
String s1 = a + "";								// 字符串拼接
String s2 = String.valueOf(a);		// String 类自己的 valueOf 方法
String s3 = Integer.toString(a);	// 包装类自己的 toString 方法
```

### 9.2 String → 基本类型

```java
String s = "123";
int i = Integer.parseInt(s);  // 除 Character 外都有 parseXxx
```

---

## ✅ 小结：Day6 Java 提高（常用类）知识回顾

- `Object` 是所有类的根，重写 `toString()`、`equals()` 实现定制行为
- `Objects` 工具类提供空指针安全的比较与哈希支持
- `Date` 表示时间戳，`SimpleDateFormat` 实现格式转换
- `Calendar` 提供对年/月/日等字段的可读可写访问
- `System` 提供时间戳与数组复制等底层功能
- `StringBuilder` 是高效拼接字符串容器
- 包装类支持装箱/拆箱，解决基本类型与对象转换问题
- 基本类型与 `String` 可通过 `valueOf`、`parseXxx` 实现互转