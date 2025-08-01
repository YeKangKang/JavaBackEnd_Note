# Day8 · Java 提高（异常、Throw、Throws、TryCatch、Finally、Objects判空工具、异常处理注意事项）

本节内容系统讲解 Java 异常体系结构、运行时与编译时异常、五大异常处理方式（try-catch-finally、throw、throws、自定义异常）及异常处理注意事项。

---

## 1. 什么是异常？

- 异常是程序在**运行过程中**出现的不正常情况，可能导致程序中断。
- Java 中异常本质上是一个**类的实例（对象）**，其祖先是 `java.lang.Throwable`。
- 异常 ≠ 语法错误。

---

## 2. 异常体系结构

```
Object
 └── Throwable
      ├── Error              // 严重错误，无法通过代码恢复
      └── Exception          // 可处理的异常
           └── RuntimeException // 运行时异常
```

### ✅ Error
- 系统级错误，JVM 层面的问题（如内存溢出），不可捕获处理。

### ✅ Exception
- 编程层面的异常，可通过代码捕获处理。
- 包括运行时异常和编译时异常。

---

## 3. 异常的分类

### 📌 编译时异常（Checked Exception）

- 编译器**强制要求捕获或声明**，否则编译失败。
- 常见如：`IOException`, `SQLException`, `ClassNotFoundException`

### 📌 运行时异常（Unchecked Exception）

- 编译器不会强制处理，但运行时可能抛出。
- 多由程序逻辑错误导致，如：`NullPointerException`, `ArrayIndexOutOfBoundsException`, `ArithmeticException`

---

## 4. 异常的产生与传播过程

- 程序运行到某行代码时，若触发错误（如数组越界），Java 将自动创建并抛出对应的异常对象。
- 若该方法未处理，则异常继续向上抛出至调用者，直到 JVM。
- 若始终未被捕获，最终导致程序终止并打印异常信息（调用栈）。

---

## 5. Java 处理异常的 5 种方式

### 5.1 throw 抛出异常对象

```java
if (obj == null) {
    throw new NullPointerException("对象不能为空");
}
```

- `throw` **在方法体内**，用于主动抛出异常。
- 一旦执行到 `throw`，方法终止执行。
- 抛出编译时异常时，需搭配 `throws` 使用。

---

### 5.2 throws 声明异常

```java
public void readFile() throws IOException, SQLException { ... }
```

- 写在方法签名后，用于声明该方法**可能抛出异常**。
- 编译时异常必须声明；运行时异常则可选。
- 多个异常用逗号分隔。

---

### 5.3 try-catch 捕获异常

```java
try {
    ...
} catch (IOException e) {	// 子类异常
    ...
} catch (Exception e) {	// 父类异常
    ...
}
```

- 用于捕获代码中可能抛出的异常。
- catch 从上往下匹配，**只执行第一个匹配成功的块**。
- 子类异常必须写在父类异常前，否则编译报错。

---

### 5.4 finally 块

- 不论是否发生异常，`finally` 中代码总会执行，常用于释放资源（文件关闭、网络断开等）。
- 即使 try 或 catch 中有 `return`，`finally` 仍会执行。
- 唯一不会执行：调用了 `System.exit(0)` 等强制退出 JVM 的语句。

---

### 5.5 Objects.requireNonNull() 判空工具

```java
String name = Objects.requireNonNull(input);
```

- 如果参数为 `null`，抛出 `NullPointerException`；否则返回该对象本身（作为链式调用或继续使用）。
- 常用于方法参数非空断言

---

## 6. 异常处理策略总结

### ✅ 多异常处理的三种方式：

1. **分别捕获**：多个 try-catch 分开写
2. **多 catch 分类处理**：一次 try，多个 catch 匹配不同异常（子类写前）
3. **统一处理**：catch 一个大类，如 `Exception`（简单但粗略）

---

### ⚠️ finally 中不建议写 return

- 若写了 return，会覆盖 try/catch 中的返回值，可能引发混乱。
- 尽量只写资源释放代码，不做跳转控制。

---

## 7. 子类方法重写时的异常声明规则（针对编译时异常）

以下规则仅适用于编译时异常（Checked Exception），运行时异常不受限制

- 子类可抛出（向下兼容）：
  - 父类方法声明的异常
  - 或这些异常的**子类**
  - 或**不抛异常**

```java
class Parent {
    public void read() throws IOException, SQLException { }
}

class Child extends Parent {
    @Override
    public void read() throws FileNotFoundException { } // ✅ 合法（IOException 的子类）
}
```

- 若父类未抛出异常，子类也不能抛出编译时异常（但可捕获处理）

> 子类**不能抛出父类未声明，且与其声明异常无继承关系的编译时异常**；运行时异常则**不受此限制**，可自由抛出

---

## 8. 自定义异常类

### ✅ 作用

表达业务层错误，如“非法年龄”“负数余额”等

### ✅ 实现步骤

```java
public class AgeIllegalException extends Exception {
    public AgeIllegalException() {
        super("年龄非法");
    }

    public AgeIllegalException(String message) {
        super(message);
    }
}
```

- 若继承 `Exception` → 编译时异常
- 若继承 `RuntimeException` → 运行时异常

---

## ✅ 总结

- 异常是运行时错误的统一机制，必须合理处理。
- `throw` 抛异常对象，`throws` 声明异常种类。
- `try-catch-finally` 是核心结构，掌握执行顺序至关重要。
- 自定义异常有助于业务清晰化，推荐使用。
- 子类重写方法的编译时异常声明**必须不超出父类范围**。