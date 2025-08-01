# Day2 · Java 流程控制语句

本日内容覆盖 Java 中三类基本控制结构：**顺序执行、条件判断、循环结构**，并补充 `break` / `continue`、三元运算符、嵌套结构建议等。

---

## 1. 顺序结构

程序默认按从上到下的顺序执行。

```java
System.out.println(1);
System.out.println(2);
System.out.println(3);
```

---

## 2. 条件判断结构

### 2.1 if / if-else / else if

```java
if (condition) {
    // [如果]
} else if (otherCondition) {
    // [另一个如果]
} else {
    // [最后]
}
```

> `if` / `else if` / `else` 属于一组结构，**只会执行其中一个分支**。

### 2.2 三元运算符（简化 if-else）

```java
类型 变量 = (条件) ? 表达式A : 表达式B;

int score = 80;
String result = (score >= 60) ? "及格" : "不及格";	// result = “及格”
```

- 条件为 `true`：返回 `表达式A` 的值
- 条件为 `false`：返回 `表达式B` 的值

> 适用于有返回值的二选一逻辑，常见于简化 `if...else` 赋值结构

---

## 3. switch 多分支选择结构

用于根据变量值匹配执行某一分支。

```java
int day = 3;
switch (day) {
    case 1: System.out.println("星期一"); break;
    case 2: System.out.println("星期二"); break;
    default: System.out.println("非法输入"); break;
}
```

### ✅ 特性说明

- `switch` 支持类型：`int`、`byte`、`short`、`char`、`enum`、`String`
- `case` 后必须跟 `break` 否则会**继续执行后续 case**（穿透）
- `default` 不是必须，但推荐写作兜底处理

---

## 4. 循环结构

### 4.1 for 循环（次数明确）

```java
for (int i = 0; i < 5; i++) {
    System.out.println(i);
}
```

- 通常用于计数、结构遍历等情况
- 循环变量只在循环体中有效

---

### 4.2 while 循环（次数不定）

```java
int i = 0;
while (i < 5) {
    System.out.println(i);
    i++;
}
```

- 先判断循环条件，条件为 `true` 再执行循环体代码**（先判断，再执行）**
- 通过在循环内对条件表达式/循环变量进行维护，来控制是否继续循环

---

### 4.3 do-while 循环（至少执行一次）

```java
int i = 0;
do {
    System.out.println(i);
    i++;
} while (i < 5);
```

- 先执行循环体代码，再判断循环条件是否为 `true` 来决定是否继续循环**（先执行，再判断）**

---

## 5. `break`, `continue` 循环控制

### 5.1 break

立即终止当前循环，跳出循环体。

```java
for (...) {
    if (x == 5) break;
}
```

### 5.2 continue

跳过本次循环剩下的所有语句，立即进入下一次循环条件判断或迭代更新。

```java
for (...) {
    if (x == 5) continue;
}
```

---

## 6. 循环嵌套与建议

### ✅ 常见嵌套结构（打印 5x5 星号方块）

```java
for (int i = 0; i < 5; i++) {
    for (int j = 0; j < 5; j++) {
        System.out.print("*");
    }
    System.out.println();
}
```

### 🔎 使用建议

- `for`：用于处理结构明确的数据（如数组、计数）
- `while`：用于控制流程或等待某条件达成
- `do-while`：用于至少执行一次逻辑的场景（如菜单输入）
- **避免嵌套层级超过 2 层**，建议先用流程图/伪代码设计逻辑
- `for` 与 `while` 混用需谨慎，常用于算法或状态机设计

---

## ✅ 总结

本日内容建立 Java 中最基础的控制流程工具，是构建逻辑结构与实现程序功能的核心组成部分。