# Day9 · Java 提高（线程通信、等待唤醒、静态工厂、线程池、Lambda表达式）

本节内容覆盖 Java 多线程中的线程通信机制、`wait` / `notify` / `notifyAll` 协作模型、线程池原理与使用方式、静态工厂方法概述，以及 Lambda 表达式的语法规则与函数式接口要求。

---

## 1. 线程通信机制

多个线程可能访问同一共享资源，但执行职责不同。为协调顺序、避免冲突，Java 提供了**等待-唤醒机制**，用于在线程间建立协作关系。

该机制依赖 `Object` 类的对象锁方法：`wait()` / `notify()` / `notifyAll()`，用于控制线程状态切换与资源争用。

---

## 2. 等待-唤醒机制详解（基于对象锁）

### 2.1 基本说明

- 所有操作必须在同步代码块（`synchronized`）中调用
- 生效前提：多个线程使用**同一对象锁**
- 常见状态转换：`RUNNABLE` ⇄ `WAITING` / `TIMED_WAITING` / `BLOCKED`

### `Object.wait()`

- 释放当前对象锁
- 线程进入 `WAITING` 或 `TIMED_WAITING` 状态
- 支持传入等待时间（毫秒）

### `Object.notify()`

- 唤醒**一个**调用了同一对象锁 `wait()` 的线程
- 被唤醒线程转为 `RUNNABLE`，等待竞争锁

### `Object.notifyAll()`

- 唤醒所有调用了该对象 `wait()` 的线程
- 所有线程进入 `RUNNABLE` 状态，参与锁竞争

> ⚠️ 若未持有对象锁调用上述方法，将抛出 `IllegalMonitorStateException`

---

## 3. 线程池机制

### 3.1 为什么需要线程池

- 避免频繁创建/销毁线程带来的系统资源浪费
- 提升线程复用率，提高并发任务执行效率
- 限制最大并发线程数，防止过载

### 3.2 原理概述

- 线程池是一个容器，包含若干“工作线程”
- 任务以 `Runnable` 或 `Callable` 对象提交至任务队列
- 空闲线程自动从队列中领取任务并执行，完成后回归空闲状态

---

## 4. 线程池的使用方式

### 4.1 使用流程

1. 创建线程池对象（推荐使用 `Executors` 工厂）
2. 创建任务对象（`Runnable` 或 `Callable`）
3. 提交任务至线程池（`submit()` 方法）
4. 可选关闭线程池（`shutdown()`）

```java
ExecutorService pool = Executors.newFixedThreadPool(5);	// 创建固定线程数量的 ThreadPool 对象
pool.submit(new MyTask());
pool.shutdown();
```

### 4.2 常见线程池工厂方法

| 方法 | 说明 |
|------|------|
| `newFixedThreadPool(int n)` | 固定线程数量 |
| `newCachedThreadPool()` | 按需创建线程，空闲线程可复用 |
| `newSingleThreadExecutor()` | 单线程顺序执行 |
| `newScheduledThreadPool(n)` | 支持定时与周期任务 |

---

## 5. 静态工厂方法（Static Factory）

静态工厂是一种使用静态方法创建对象的方式，可替代传统 `new` 构造方式：

```java
public static Product create(String name) {
    return new Product(name);
}
```

优点：
- 隐藏构造逻辑，封装性更强
- 返回子类或缓存对象更灵活
- 命名清晰，适配不同创建需求

---

## 6. Lambda 表达式

### 6.1 只有函数式接口类型才可使用 Lambda 表达式

**Lambda 表达式是一种仅用于函数式接口类型的语法糖**，其本质是创建一个**函数式接口的匿名实现对象**（引用类型）。该匿名对象可作为参数传递，或赋值给变量，从而支持函数式编程场景。

函数式接口需满足：

- 接口中**只能包含一个抽象方法**，包括自身定义及从父接口继承的方法在内
- 可含多个 `default` / `static` 方法

```java
@FunctionalInterface
interface MyFunc {
    void run();	// 有且只有一个抽象方法，包括自身定义及父类接口继承的方法
    static void log(String s) {}
    default void help() {}
}
```

> **Lambda 表达式只能用于函数式接口**。函数式接口是指只包含一个抽象方法的接口，如常见的 `Runnable`（无参无返回）、`Function<T, R>`（一参一返回）等。

---

### 6.2 Lambda 语法结构 & 简化规则

#### ✅ 标准语法结构

```java
(参数列表) -> {
    // 方法体
    return 返回值;
}
```

#### ✅ 简化规则

| 条件 | 可省略 |
|------|----------|
| 参数类型可推断 | 参数类型 |
| 仅一个参数 | 小括号 |
| 仅一行代码 | `{}`、`;`、`return` |

- 上下文**参数类型明确可推断**：可省略【参数类型】

  - 上下文可推断：`Function<Integer, Integer> f = x -> 2 * x;`
  - 上下文不可推断：`Object o = (Integer x) -> 2 * x;`

- **仅有一个参数**：可省略【`()`】
  - 只有一个参数：`Function<Integer, Integer> f = x -> 2 * x;`
  - 没有 / 有多个参数：`BiFunction<Integer, Integer, Integer> f = (a, b) -> a + b;`

- **仅有一行代码**：可省略【`{}`、`;`、`return`】
  - 仅有一行代码（即使有返回值）：`Function<Integer, Integer> f = x -> 2 * x;`
  - 有多行代码：见下面的示例

---

### 6.3 使用示例

```java
// 没有参数和返回值
Runnable r = () -> System.out.println("Running");

// 有参数有返回值，仅一行代码可省略 {}
Function<Integer, Integer> f = x -> x * x;

// 多行代码：不能省略 ;、{}、return，但可省略参数括号（仅1个参数）
Function<Integer, Integer> g = x -> {
    int y = x + 1;
    return y * y;
};

// 函数式接口作为参数
public static void calc(Function<Integer, Integer> func) {
    int result = func.apply(5);
    System.out.println("输出：" + result);
}

// ✅ 方式 1：直接传入 Lambda 表达式（等价于匿名对象）
calc(x -> x + 10); // 输出：15

// ✅ 方式 2：传入已声明的函数式接口变量（Lambda 表达式的返回对象）
calc(f);           // 输出：25
```

> Lambda 表达式等价于创建函数式接口的**匿名实现对象**。

---

## ✅ 总结

- 多线程中的协作关系通过 `wait/notify` 实现，区别于加锁竞争
- 线程池通过线程复用提升并发效率，是高性能系统常见组件
- Lambda 表达式本质是“行为抽象”的简化语法，仅适用于函数式接口
- 所有机制本质服务于“高效、可靠、可控”的并发任务模型构建